# Is Drift on sqlite-wasm production-viable?

Research for [issue 04](../issues/04-drift-sqlite-wasm-viable.md). Investigated 2026-09-11 against
drift 2.35.0 (released 2026-09-09) and current browser compatibility data.

## Verdict

**Viable under conditions.** Drift on sqlite-wasm is a real, maintained, stable-API product that
persists to OPFS in every browser we care about, including iOS Safari. It is not a science project.
But `WasmDatabase.open`'s OPFS path is gated on cross-origin isolation, multi-tab safety was only
repaired six weeks ago, and Safari can delete the database out from under the user. The conditions
below are not "nice to have"; each one, if unmet, produces either a silent downgrade to a slower or
unsafe backend or actual data loss.

### Conditions

1. **Serve the web app cross-origin isolated.** `Cross-Origin-Opener-Policy: same-origin` plus
   `Cross-Origin-Embedder-Policy: require-corp` (or `credentialless`). Without these, Chrome, Safari
   and Chrome-on-Android silently fall back from OPFS to an IndexedDB-backed VFS that loads the whole
   database into memory and flushes asynchronously. Only Firefox gets OPFS without headers.
   ([source](https://raw.githubusercontent.com/simolus3/drift/develop/drift/lib/src/web/wasm_setup.dart),
   lines 132-133; [docs](https://drift.simonbinder.eu/platforms/web/#additional-headers))
2. **Build the Flutter web app with `--no-web-resources-cdn`.** COEP `require-corp` blocks
   cross-origin subresources that do not opt in, and Flutter loads CanvasKit and fonts from
   `gstatic.com` by default. The flag exists and is documented in `flutter build web --help` on the
   installed Flutter 3.44.2. Verify no other cross-origin asset (icons, images, OAuth popups) is
   loaded; drift's own docs call out that COOP breaks Google Auth popups.
   ([drift docs](https://drift.simonbinder.eu/platforms/web/#additional-headers))
3. **Pin drift >= 2.34.3 and never let `sqlite3.wasm` / `drift_worker.js` drift out of sync with the
   package version.** 2.34.3 is the release that added navigator locks around OPFS access; before it,
   two tabs writing concurrently reliably produced `SqliteException(10): disk I/O error` and, per the
   maintainer's own assessment of a sibling issue, a window where locks could be released mid
   transaction. The wasm and worker files are downloaded manually from GitHub releases and are *not*
   version-managed by pub. ([changelog](https://github.com/simolus3/drift/blob/develop/drift/CHANGELOG.md),
   [issue 3840](https://github.com/simolus3/drift/issues/3840))
4. **Decide the header posture before launch and never change it.** COI on and COI off select
   physically different storage (OPFS vs IndexedDB). Drift does not migrate between them
   automatically: `moveExistingIndexedDbToOpfs` defaults to `false`, and 2.30.1 was a bugfix for it
   moving data when it should not have. Flipping headers post-launch orphans every existing database.
5. **Treat the web database as a replica, not the only copy.** Safari deletes all script-writable
   storage, OPFS included, after seven days of browser use without user interaction with the origin.
   Since zarya will have a sync server anyway, this is survivable, but it means the web client must
   be able to rebuild from the server and must not be the sole home of anything.
   ([WebKit](https://webkit.org/blog/14403/updates-to-storage-policy/))
6. **Check `WasmDatabaseResult.chosenImplementation` at runtime and surface it.** If drift lands on
   `unsafeIndexedDb` or `inMemory`, persistence is either unsafe across tabs or absent. Drift's own
   docs tell you to warn the user in that case.
7. **Accept the multi-tab semantics you get.** Under `opfsLocks` (the Chrome/Safari-with-headers
   mode) every statement and every transaction is serialized behind a `navigator.locks` lock named
   `drift-db-<name>`, and stream queries are invalidated across tabs via `BroadcastChannel`. That is
   correct but it means one slow transaction in a background tab blocks the foreground tab.
   ([navigator_locks_interceptor.dart](https://github.com/simolus3/drift/blob/develop/drift/lib/src/web/wasm_setup/navigator_locks_interceptor.dart))

If condition 1 or 2 cannot be met (for example because the hosting arrangement cannot set headers),
the answer is not "not viable" but "OPFS is off the table with stock drift" -- see
[The escape hatch](#the-escape-hatch-sqlite3_web--drift_sqlite_async) below, which removes the header
requirement entirely at the cost of a different connection layer.

## Current state of drift's web support

Drift's web API is `WasmDatabase.open` from `package:drift/wasm.dart`. It is explicitly the stable
one: "`WasmDatabase.open` is drift's stable web API"
([docs](https://drift.simonbinder.eu/platforms/web/#migrating-from-existing-web-databases)). The
older sql.js-based `package:drift/web.dart` was deprecated in drift 2.20.0 and is described in the
docs as still supported but experimental. Do not use it.

Setup is two files copied into `web/` from the matching GitHub release (`sqlite3.wasm`,
`drift_worker.js`), plus `sqlite3.wasm` served as `Content-Type: application/wasm`. With
`drift_flutter` the Dart side is a single call:

```dart
driftDatabase(
  name: 'zarya',
  web: DriftWebOptions(
    sqlite3Wasm: Uri.parse('sqlite3.wasm'),
    driftWorker: Uri.parse('drift_worker.dart.js'),
  ),
);
```

([snippet source](https://github.com/simolus3/drift/blob/develop/docs/lib/src/snippets/platforms/web.dart))

`drift_flutter` has had `DriftWebOptions` since at least 0.2.6; both `nooka` and `habbits` already sit
on `drift_flutter ^0.3.0` and call bare `driftDatabase(name: ...)`, so the delta for zarya is the
`web:` argument plus the two static files. No change to table definitions, DAOs, or generated code.

### Storage backends, in drift's own preference order

Verified against
[wasm_setup.dart](https://github.com/simolus3/drift/blob/develop/drift/lib/src/web/wasm_setup.dart)
and
[wasm_setup/shared.dart](https://github.com/simolus3/drift/blob/develop/drift/lib/src/web/wasm_setup/shared.dart),
not just the prose docs.

| Rank | Implementation | Storage | Requires | Multi-tab |
| --- | --- | --- | --- | --- |
| 1 | `opfsShared` | OPFS, `SimpleOpfsFileSystem` in a dedicated worker spawned by a shared worker | a SharedWorker that can construct a Worker | safe, single owner |
| 2 | `opfsLocks` | OPFS, `WasmVfs` async-OPFS server in a nested dedicated worker driven by `Atomics.wait` | dedicated worker + nested worker + `SharedArrayBuffer` (so: COOP/COEP) | safe since 2.34.3, via `navigator.locks` |
| 3 | `sharedIndexedDb` | IndexedDB chunks, whole DB held in memory in a shared worker, writes flushed async | SharedWorker + IndexedDB | safe in principle, single owner; see caveat below |
| 4 | `unsafeIndexedDb` | same VFS, no worker coordination | IndexedDB | **unsafe**, races between tabs |
| 5 | `inMemory` | nothing | - | n/a, no persistence |

The gate for `opfsLocks` in source is literally
`status.supportsNestedWorkers && status.canAccessOpfs && status.supportsSharedArrayBuffers`. There is
no way to get OPFS out of stock drift on Chrome or Safari without cross-origin isolation.

## Browser support, in practice

Drift's own support matrix on the docs page is **stale**: it is dated by browser versions tested in
2023 (Chrome 114, Firefox 114, Safari 16.2, Safari Technology Preview 172). Treat the table below as
the current picture; it is derived from the drift source gates above combined with MDN's browser
compatibility data, plus one direct measurement.

Primitives, from MDN browser-compat-data (`main` branch, read 2026-09-11):

- `FileSystemSyncAccessHandle` (OPFS sync handles, dedicated workers only): Chrome 102, Chrome Android
  109, Firefox 111, Safari 15.2, Safari iOS 15.2.
  ([BCD](https://github.com/mdn/browser-compat-data/blob/main/api/FileSystemSyncAccessHandle.json))
- `Worker` constructor available inside a worker: Firefox 34 fully; Chrome 69 and Safari 16.4 both
  marked `partial_implementation` with the note "Not available in Shared Workers"
  ([crbug 40695450](https://crbug.com/40695450), [webkit 265263](https://webkit.org/b/265263)).
  ([BCD](https://github.com/mdn/browser-compat-data/blob/main/api/Worker.json))
- `SharedWorker`: Chrome 5, Firefox 29, Safari 16, Safari iOS 16, and **Chrome Android 148**, which
  reached stable on 2026-05-05 per the Chromium milestone schedule. This is new; Chrome on Android had
  no shared workers for the entire history of drift's web support.
  ([BCD](https://github.com/mdn/browser-compat-data/blob/main/api/SharedWorker.json),
  [intent to ship](https://groups.google.com/a/chromium.org/g/blink-dev/c/pS1PDOa69CU),
  [milestone schedule](https://chromiumdash.appspot.com/fetch_milestone_schedule?mstone=148))
- `createSyncAccessHandle({mode})`, the non-standard `readwrite-unsafe` mode: Chrome 121 only; Firefox
  and Safari explicitly `false`.
  ([BCD](https://github.com/mdn/browser-compat-data/blob/main/api/FileSystemFileHandle.json))

Resulting matrix for stock drift 2.35.0:

| Browser | With COOP/COEP | Without |
| --- | --- | --- |
| Firefox 111+ | `opfsShared` | `opfsShared` |
| Firefox private window | IndexedDB or in-memory (no File System Access API in private browsing, per drift docs) | same |
| Chrome desktop 69+ | `opfsLocks` | `sharedIndexedDb` |
| Chrome Android 148+ | `opfsLocks` | `sharedIndexedDb` |
| Chrome Android < 148 | `opfsLocks` | `unsafeIndexedDb` (data races across tabs) |
| Safari 16.4+ / iOS Safari 16.4+ | `opfsLocks` | `sharedIndexedDb` |
| Safari 16.0-16.3 | `sharedIndexedDb` (no nested workers before 16.4) | `sharedIndexedDb` |

Direct measurement, 2026-09-11: drift's own compatibility widget on
<https://drift.simonbinder.eu/platforms/web/> (a site which does serve
`cross-origin-opener-policy: same-origin` and `cross-origin-embedder-policy: require-corp`, verified
by `curl -I`) reported on Chromium 152:

```
Chosen implementation: WasmStorageImplementation.opfsLocks
Features missing: {MissingBrowserFeature.dedicatedWorkersInSharedWorkers}
```

I also probed directly in that page: constructing a `SharedWorker` and asking it for `typeof Worker`
returns `"undefined"` on Chromium 152. `opfsShared` remains Firefox-only as of today.

Note for the record: **iOS Safari 16.4+ is usable** and reaches OPFS with headers. Every browser
engine on iOS is WebKit, so Chrome/Firefox on iOS behave as Safari does. I did **not** verify this
empirically on a real iOS device or simulator (no simulators available on this machine); it is
inferred from MDN compat data plus drift's source gates, and drift's own 2023 table already recorded
Safari Technology Preview 17.0 with headers as "Full".

## Multiple tabs

This is the part that actually broke in production, recently.

- [drift#3840](https://github.com/simolus3/drift/issues/3840) (2026-07-23, drift 2.31.0, Chrome 150,
  COOP/COEP set, `opfsLocks`): two tabs produced `SqliteException(10): disk I/O error` and left the
  database "unusable for our users". Root cause analysis in the thread: drift's `opfsLocks` builds
  `WasmVfs` straight from `package:sqlite3`, which arbitrates cross-tab access only through the
  exclusivity of `createSyncAccessHandle`, retries six times with no backoff, and raises
  `SQLITE_IOERR` rather than `SQLITE_BUSY`, so `busy_timeout` never applies.
- The maintainer agreed, pushed navigator locks, and closed it with "Drift version 2.34.3 contains
  this fix". The reporter confirmed the fix works.
- The upstream issue is **still open**:
  [sqlite3.dart#240](https://github.com/simolus3/sqlite3.dart/issues/240). Worth reading in full. It
  contains a reliable two-tab reproducer, a maintainer explanation that the WebLocks fix may be
  masking rather than eliminating the race ("it does look like the WebLocks delay is helping here,
  but it's still surprising"), and a maintainer acknowledgement that reads may still be affected
  because an implicit lock is only closed asynchronously.
- The same thread carries reports of `database disk image is malformed` under `sharedIndexedDb`,
  which the maintainer could not explain ("I'm also not understanding the corruption issue"). That is
  the mode you land in on Chrome and Safari **without** headers. It is a further argument for
  condition 1.
- [drift#3856](https://github.com/simolus3/drift/issues/3856) (2026-09-09): concurrent
  `WasmDatabase.open` calls raced on the shared `_drift_feature_detection` OPFS probe file and killed
  the probe worker. Fixed the same day in 2.35.0.

Read together: drift's web multi-tab story was genuinely broken until 2026-07-27 and has been
repaired twice in the last seven weeks. It is being actively maintained, but the code is young enough
that "pin the newest version and test two tabs yourself" is a real requirement, not boilerplate
advice.

A shared worker is **not** required for correctness on Chrome or Safari (they cannot use one for OPFS
anyway); the `navigator.locks` interceptor is what makes `opfsLocks` multi-tab safe. Firefox uses a
shared worker and gets single-owner semantics for free.

## iOS Safari and storage eviction

The database can be deleted without the user doing anything.

- WebKit's storage policy applies to "localStorage, Cache API, IndexedDB, Service Worker, and File
  System"; OPFS is the File System entry. Cookies and HTTP cache are excluded.
  ([WebKit, Updates to Storage Policy](https://webkit.org/blog/14403/updates-to-storage-policy/))
- MDN states the rule exactly: "Safari proactively evicts data when cross-site tracking prevention is
  turned on. If an origin has no user interaction, such as click or tap, in the last seven days of
  browser use, its data created from script will be deleted."
  ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria))
  This is the seven-day cap introduced in Safari 13.1 / iOS 13.4
  ([WebKit, 2020](https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/)).
- Eviction is per-origin and total: "all of its data, not parts of it, is deleted at the same time".
  There is no partial loss to recover from, only an empty database.
- Two exemptions matter. Storage in persistent mode is exempt, and WebKit "currently grants a request
  based on heuristics like whether the website is opened as a Home Screen Web App". A Home Screen web
  app is also outside Safari's day counter entirely, per the 2020 post. So: call
  `navigator.storage.persist()` (Safari 15.2+, per
  [BCD](https://github.com/mdn/browser-compat-data/blob/main/api/StorageManager.json)) and encourage
  Add to Home Screen if the iOS web client matters.
- Quota, for sizing: browser apps get 60% origin quota and 80% overall; Home Screen / dock web apps
  get the same treatment as browser apps. Nowhere near a constraint for a habits and tasks database.
- Chrome and Firefox evict LRU only under storage pressure, not on a timer.

Practical read for zarya: an iOS user who opens the web client, then does not touch it for a week,
comes back to an empty local database. With a sync server that is a resync, not data loss. Without
one it would be fatal. This is the strongest argument in this document that web-plus-local-first only
works because sync exists.

## Payload size and load time

Measured from the drift 2.35.0 release assets, downloaded 2026-09-11:

| File | Raw | gzip -9 | brotli -q11 |
| --- | --- | --- | --- |
| `sqlite3.wasm` | 748,686 B (731 KiB) | 350 KiB | 307 KiB |
| `drift_worker.js` | 357,220 B (349 KiB) | 106 KiB | 87 KiB |

`sqlite3mc.wasm` (SQLite3 Multiple Ciphers, if encryption is ever wanted) is 776 KiB raw.

So roughly **395 KiB brotli of additional transfer** on first load, on top of Flutter's own payload
(CanvasKit alone is around 1.5 MB). Both files are fetched lazily, when the database is first opened,
not during initial page paint. Arithmetic, not measurement: at 5 Mbps that is about 0.65 s, at 20 Mbps
about 0.16 s, plus WebAssembly compilation. Both are cacheable static assets, so it is a first-visit
cost. Note the drift worker script is fetched during feature probing, which spawns both a shared and a
dedicated worker from the same file.

I did not benchmark query throughput on any web backend. Drift's docs describe the IndexedDB fallbacks
as "slightly slower" without numbers, and `sqlite3_web` documents the IndexedDB mode as keeping the
entire database in memory and flushing asynchronously, which "technically looses durability, but is
reasonably reliable in practice".

## Migrations

Migrations are not web-specific. The database class, `schemaVersion`, `MigrationStrategy`, the
generated `database.steps.dart` from `drift_dev make-migrations`, and the schema verification tests
are all platform-agnostic; the drift migration documentation contains no web-specific section, and the
same `MigrationStrategy` is executed by whichever worker hosts the connection. The generation and
verification tooling runs on the VM, so it is unaffected. I found no evidence of a separate web
migration path.

Three web-specific hazards around migrations, though:

1. **Storage relocation is not a migration.** Changing the COOP/COEP posture moves the database
   between OPFS and IndexedDB. `WasmDatabase.open(moveExistingIndexedDbToOpfs: true)` exists for the
   IndexedDB-to-OPFS direction only (added 2.30.0, bugfixed 2.30.1). There is no OPFS-to-IndexedDB
   path. `WasmProbeResult.existingDatabases` will tell you which storages actually hold a database.
   Note that the reporter of drift#3840 was using exactly `moveExistingIndexedDbToOpfs: true`.
2. **Two tabs on two app versions.** After a deploy, an old tab and a new tab share one database.
   The new tab migrates to schema N+1; the old tab then opens a database whose `user_version` is
   ahead of its own. Drift 2.31.0 added an explicit error for attempted downgrades in step-by-step
   migrations, so this surfaces as a thrown exception rather than silent corruption. That is
   inference from the changelog and the shared-worker architecture, not something drift documents;
   treat it as a scenario to test rather than a stated behaviour.
3. **`initializeDatabase` cannot import a WAL database**, and WAL is not supported on the web at all
   ([docs](https://drift.simonbinder.eu/platforms/web/#using-existing-databases)). Relevant when
   importing your own nooka/habbits data if that ever runs in a browser.

## The escape hatch: sqlite3_web / drift_sqlite_async

This is the most important finding for the header question, and it comes from the drift maintainer
himself in the drift#3840 thread.

`package:sqlite3_web` (also by simolus3, part of the sqlite3.dart repo, 0.9.4) is a newer web layer
that supersedes the one baked into drift. Its 0.8.0 changelog entry is unambiguous:

> **Breaking**: Remove `opfsAtomics` file system implementation. The new
> `opfsWithExternalLocksWorkaround` supports the same browsers while being faster and not requiring
> special headers.

Its `DatabaseImplementation` enum lists `opfsWithExternalLocks` (Chrome, using the non-standard
`readwrite-unsafe` sync-handle mode plus the Web Locks API) and `opfsWithExternalLocksWorkaround` ("A
design similar to `opfsWithExternalLocks` that also works in Firefox and Safari"). In other words the
COOP/COEP requirement is a limitation of drift's own older worker code, not of OPFS in Dart. PowerSync
independently reports the same outcome for their Flutter web SDK: "Starting with version 2.2.0, OPFS
is used on all major browsers (Chrome, Firefox and Safari)", eliminating the headers previously needed
on Safari
([PowerSync](https://docs.powersync.com/client-sdk-references/flutter/flutter-web-support)).

simolus3 on drift adopting it: "The `sqlite3_web` package was originally a spin-off from web code in
drift, but is arguably much better at this point and I want to adopt it in drift in a future major
release." He recommends `drift_sqlite_async` as a preview:

> As a "preview" for drift with better inner connection drivers before I manage to finish drift3, it
> might be worth trying out `drift_sqlite_async` (maintained by my employer PowerSync). It has its own
> worker, but implements regular drift connection APIs using the `sqlite3_web` package on the web and
> uses the same file format and locations as drift today. That package is likely quite a bit faster
> for OPFS since it doesn't need a second worker and atomics for OPFS.

`drift_sqlite_async` is published by the verified publisher powersync.com, version 0.3.1 (published
roughly three months ago), depends on `drift >=2.28.0 <3.0.0`, `sqlite3 ^3.2.0`, `sqlite_async
^0.14.0`. `sqlite_async` 0.14.3 supports `sqlite3_web` 0.9.x, so the headerless OPFS modes are
available on that path today. Its README calls web support "Beta" and lists two limitations:
read-only transactions are not supported in drift, and update notifications may be duplicated.

Assessment: this is a genuine alternative, not vapourware, but it swaps drift's default connection
layer for PowerSync's, in beta, with a different worker file, for a product that has no other reason
to depend on PowerSync. **Recommendation: build on stock `WasmDatabase.open` with the headers, and
keep `drift_sqlite_async` in the back pocket** for the case where the headers turn out to be
unaffordable (a third-party embed, an OAuth popup flow, a host that cannot set headers). Also note
that "drift3" adopting `sqlite3_web` is on the maintainer's roadmap, which makes the header
requirement likely temporary but with no date attached.

## What I could not verify

- **iOS Safari empirically.** No iOS simulator was available on this machine. The iOS rows in the
  matrix are inference from MDN compat data plus drift's source gates. Before committing, run drift's
  compatibility widget at <https://drift.simonbinder.eu/platforms/web/> on a real iPhone; the site is
  already cross-origin isolated, so it reports the with-headers answer directly.
- **Whether sqlite3.dart#240 is fully resolved.** It is open, the maintainer himself doubts the
  WebLocks fix fully closes the read path, and no one has closed the loop. This is the single largest
  residual risk on the multi-tab story.
- **`readwrite-unsafe` on Chrome 152 empirically.** MDN says Chrome 121+; my in-page probe was blocked
  by the test page's own COEP policy on blob workers. Not independently confirmed.
- **Query throughput** on any of the five backends. No numbers exist in drift's docs and I ran no
  benchmark.
- **Chrome's `navigator.storage.persist()` grant heuristics.** MDN does not enumerate them and I did
  not find a first-party Chromium statement.
- **Whether drift's own docs will be corrected.** The support matrix on the docs page still cites
  2023 browser versions and still attributes the multi-tab caveat only to Chrome-on-Android without
  headers, which drift#3840's analysis explicitly flagged as a "docs mismatch". As of today the
  `develop` branch source of that page is unchanged.

## Sources

Primary, drift and sqlite3.dart:

- <https://drift.simonbinder.eu/platforms/web/> and its source,
  <https://github.com/simolus3/drift/blob/develop/docs/content/platforms/web.md>
- <https://github.com/simolus3/drift/blob/develop/drift/CHANGELOG.md>
- <https://github.com/simolus3/drift/blob/develop/drift/lib/src/web/wasm_setup.dart>
- <https://github.com/simolus3/drift/blob/develop/drift/lib/src/web/wasm_setup/shared.dart>
- <https://github.com/simolus3/drift/blob/develop/drift/lib/src/web/wasm_setup/navigator_locks_interceptor.dart>
- <https://github.com/simolus3/drift/blob/develop/docs/lib/src/snippets/platforms/web.dart>
- <https://github.com/simolus3/drift/blob/develop/drift_flutter/CHANGELOG.md>
- <https://pub.dev/documentation/drift/latest/wasm/WasmStorageImplementation.html>
- <https://github.com/simolus3/drift/releases> (asset sizes, release dates)
- <https://github.com/simolus3/drift/issues/3840>, <https://github.com/simolus3/drift/issues/3856>,
  <https://github.com/simolus3/drift/issues/3792>
- <https://github.com/simolus3/sqlite3.dart/issues/240>
- <https://github.com/simolus3/sqlite3.dart/blob/main/sqlite3_web/CHANGELOG.md>
- <https://github.com/simolus3/sqlite3.dart/blob/main/sqlite3_web/lib/src/types.dart>
- <https://github.com/simolus3/sqlite3.dart/blob/main/sqlite3/CHANGELOG.md>
- <https://pub.dev/packages/drift_sqlite_async>
- <https://github.com/powersync-ja/sqlite_async.dart/blob/main/packages/sqlite_async/CHANGELOG.md>
- <https://docs.powersync.com/client-sdk-references/flutter/flutter-web-support>

Browser platform:

- <https://github.com/mdn/browser-compat-data/blob/main/api/FileSystemSyncAccessHandle.json>
- <https://github.com/mdn/browser-compat-data/blob/main/api/FileSystemFileHandle.json>
- <https://github.com/mdn/browser-compat-data/blob/main/api/SharedWorker.json>
- <https://github.com/mdn/browser-compat-data/blob/main/api/Worker.json>
- <https://github.com/mdn/browser-compat-data/blob/main/api/StorageManager.json>
- <https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria>
- <https://developer.mozilla.org/en-US/docs/Web/API/FileSystemSyncAccessHandle>
- <https://webkit.org/blog/14403/updates-to-storage-policy/>
- <https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/>
- <https://groups.google.com/a/chromium.org/g/blink-dev/c/pS1PDOa69CU>
- <https://chromiumdash.appspot.com/fetch_milestone_schedule?mstone=148>
- <https://crbug.com/40695450>, <https://webkit.org/b/265263>
- <https://docs.flutter.dev/platform-integration/web/wasm>

Local verification on this machine: `flutter build web --help` (Flutter 3.44.2), `curl -I` on the
drift docs origin, gzip/brotli sizing of the drift 2.35.0 release assets, and a live probe of
Chromium 152 against drift's compatibility widget.
