# Is Drift on sqlite-wasm production-viable?

Type: research
Status: resolved
Blocked by: -

## Question

The decision to ship a Flutter web client *and* stay local-first rests on Drift running against
sqlite-wasm with OPFS in the browser. This is the least-travelled part of the route and it is
load-bearing: if it does not hold, either the web client or local-first has to give.

Establish, with sources:

- The current state of Drift's web support - which storage backends it offers, which are
  recommended, and which are deprecated
- **OPFS browser support in practice**: Chrome, Safari (desktop and iOS), Firefox. Which need
  workers, which need cross-origin isolation headers, and what the fallbacks degrade to
- Behaviour with **multiple tabs open** - locking, corruption risk, and whether a shared worker
  is required
- Whether **iOS Safari** specifically is usable, including any storage eviction rules that could
  delete the database out from under the user
- Payload size of the wasm bundle and its effect on load time
- Migration story: do Drift schema migrations work the same way on web?
- Real reports of people shipping this - issues, blog posts, known sharp edges

If the answer is "it works but only under conditions X, Y, Z", state the conditions precisely.
If it is "no", say so plainly - that is a more useful answer than a hedge, and it sends
[Which sync engine?](06-which-sync-engine.md) and the client strategy back to the drawing board.

## Answer

**Viable under seven conditions.** Full findings, with sources:
[research/04-drift-sqlite-wasm-viable.md](../research/04-drift-sqlite-wasm-viable.md).

Drift on sqlite-wasm is real and maintained (2.35.0, released 2026-09-09), but the OPFS path is
gated behind cross-origin isolation and multi-tab safety was only repaired in 2.34.3, six weeks
ago.

The conditions:

1. Serve the web app **cross-origin isolated** - COOP `same-origin` + COEP `require-corp`.
   Verified against drift's own source: `wasm_setup.dart` gates `opfsLocks` on
   `supportsNestedWorkers && canAccessOpfs && supportsSharedArrayBuffers`. There is no way to get
   OPFS out of stock drift on Chrome or Safari without it.
2. Build Flutter with `--no-web-resources-cdn`, or COEP breaks CanvasKit.
3. Pin `drift >= 2.34.3` with version-matched `sqlite3.wasm` and `drift_worker.js`.
4. Fix the header posture before launch and **never change it** - switching silently relocates
   the database between OPFS and IndexedDB with no automatic migration.
5. Treat the web database as a **replica, never the only copy**.
6. Check `chosenImplementation` at runtime rather than assuming a backend.
7. Accept that all statements serialize behind a `navigator.locks` lock.

Findings that matter beyond the yes/no:

- **iOS Safari (16.4+) works, but WebKit deletes the database after seven days of browser use
  without user interaction with the origin.** OPFS counts as "File System" in the evicted list
  and eviction is all-or-nothing per origin. Mitigations are `navigator.storage.persist()` and
  Add to Home Screen. This is the strongest argument that web + local-first only works *because*
  sync exists - it makes the sync layer load-bearing for correctness on iOS web, not just
  convenience.
- **Chrome Android gained SharedWorker in Chrome 148 (stable 2026-05-05)**, closing the old
  "no multi-tab safety on Android" hole. Drift's published support matrix is stale and still
  cites 2023 browser versions.
- **Multi-tab genuinely broke in production** - drift#3840, July 2026: two Chrome tabs under
  `opfsLocks` produced `disk I/O error` and an unusable database. Fixed with navigator locks in
  2.34.3, but the upstream `sqlite3.dart#240` is **still open**, with the maintainer saying the
  WebLocks fix may be masking rather than eliminating the read-path race. Largest residual risk.
- **Payload** is ~395 KiB brotli, fetched lazily on first DB open.
- **Migrations are unchanged on web**, but two tabs running two app versions can hit a downgrade
  error, and WAL is unsupported.
- **Escape hatch exists.** The maintainer's own `sqlite3_web` 0.8.0 dropped the atomics VFS
  because `opfsWithExternalLocksWorkaround` "supports the same browsers while being faster and
  not requiring special headers". The COOP/COEP requirement is a limitation of drift's older
  worker code, not of OPFS in Dart. `drift_sqlite_async` (PowerSync, beta) exposes it today, and
  the maintainer intends to adopt `sqlite3_web` in drift proper in a future major release, with
  no date. Recommendation is stock `WasmDatabase.open` with headers, keeping this in reserve.

Delta from `nooka`/`habbits` is small - both already use `drift_flutter ^0.3.0`, so it is a
`web: DriftWebOptions(...)` argument plus two static files in `web/`.

Not verified: iOS Safari empirically, whether `sqlite3.dart#240` is genuinely closed, query
throughput on any backend, and Chrome's `persist()` grant heuristics.
