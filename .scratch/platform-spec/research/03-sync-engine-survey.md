# Survey of local-first sync options for Flutter

Research output for [03-sync-engine-survey](../issues/03-sync-engine-survey.md). Facts, not a
recommendation - the choice belongs to [06-which-sync-engine](../issues/06-which-sync-engine.md).

**Researched 2026-09-11.** Every version, licence, price and platform claim below was read on that
date from pub.dev's API, the project's own `LICENSE` file, the GitHub REST API, or the vendor's own
documentation. Anything that could not be established from a primary source is marked
**unverified** rather than guessed.

Evaluation context assumed throughout: Flutter on iOS + Android + **web**, Drift/SQLite as the
on-device source of truth, fully offline-capable, one user with three of their own devices, no
collaboration, and a strong preference for not depending on a vendor's continued existence.

---

## 1. Headline: four things that changed recently

Read these before anything else - they invalidate most sync advice written before mid-2026.

1. **ElectricSQL is not what blog posts say it is, twice over.** It abandoned the client-side
   SQLite + CRDT product and is now a *read-path-only* Postgres→HTTP sync engine
   ([electric.ax/docs/sync](https://electric.ax/docs/sync/)); and on **2026-08-11 it was acquired by
   Databricks**, with the announcement stating "Electric Cloud is winding down. Cloud users will
   need to self-host or move to another provider"
   ([electric.ax/blog/2026/08/11/electric-joining-databricks](https://electric.ax/blog/2026/08/11/electric-joining-databricks),
   [neon.com/blog/electric-joins-neon](https://neon.com/blog/electric-joins-neon)). The domain
   `electric-sql.com` now 301-redirects to `electric.ax`. The community Dart client is
   discontinued and its repo is archived
   ([github.com/SkillDevs/electric_dart](https://github.com/SkillDevs/electric_dart), archived,
   last push 2024-07-24).
2. **Turso now tells you not to use libSQL for sync.** Their own 2026-04-24 benchmark post:
   "regardless of the situation and use case, you should be using Turso, not libSQL, if you are
   using sync" ([turso.tech/blog/sync-benchmark](https://turso.tech/blog/sync-benchmark)). The
   libSQL server release stream has been static since **2025-02-14**
   (`libsql-server-v0.24.32`), while the Rust rewrite ships weekly prereleases. The new Turso
   cloud server is **closed source**
   ([turso.tech/blog/upcoming-changes-to-the-turso-platform-and-roadmap](https://turso.tech/blog/upcoming-changes-to-the-turso-platform-and-roadmap),
   2025-01-21).
3. **cr-sqlite is effectively stalled.** Last release `v0.16.3` on **2024-01-17**; the original
   maintainer's last commit is also **2024-01-17**; everything since is drive-by packaging fixes
   from outside contributors ([github.com/vlcn-io/cr-sqlite](https://github.com/vlcn-io/cr-sqlite),
   commits API). The browser wasm build
   ([github.com/vlcn-io/js](https://github.com/vlcn-io/js)) was last touched **2023-12-16**. It has
   **no Dart binding at all**.
4. **PowerSync's server is source-available, not open source.** `powersync-service` is under
   **FSL-1.1-ALv2** - a non-compete licence that converts to Apache-2.0 two years after each
   release
   ([LICENSE](https://github.com/powersync-ja/powersync-service/blob/main/LICENSE)). The Dart
   client SDK is plain Apache-2.0
   ([LICENSE](https://github.com/powersync-ja/powersync.dart/blob/main/packages/powersync/LICENSE)).

---

## 2. Comparison table

Sorted roughly by how close each is to being usable on this stack today.

| Option | Real Flutter SDK? | Drift interop | Flutter web | Self-host | Conflict model | Own auth? | Maturity signal |
|---|---|---|---|---|---|---|---|
| **PowerSync** | **Yes, first-party** - `powersync` 2.4.0, 2026-09-02, publisher `powersync.com` | **Yes, official** - `drift_sqlite_async` 0.3.1 (beta), publisher `powersync.com` | Yes, **beta**: sqlite3.wasm + OPFS/IndexedDB | Yes (Docker, "Open Edition", GA) | Server-authoritative; your backend decides; default is per-field LWW | No - verifies your JWT/JWKS | Dart SDK **GA**; web Beta; service v1.26.1 (2026-09-11) |
| **Hand-rolled change log over a Dart backend** | n/a - you write it | Total control | Yes (Drift web is stable) | Yes, by definition | Whatever you specify | Whatever you build | Zero dependencies to rot; all the work is yours |
| **Supabase (as dumb transport)** | `supabase_flutter` 2.17.2, 2026-08-14, first-party | None. It has no local store, so Drift stays authoritative | Yes (it's just HTTP) | Yes (Docker; 4 GB RAM min) | **None provided** - row-level last-writer-lands | Supabase Auth, bypassable via open RLS + custom key | GA product, but **no first-party offline story** |
| **PocketBase (as dumb transport)** | `pocketbase` 0.25.1, 2026-09-05, first-party | None; same as Supabase | Yes (HTTP + SSE) | Yes - single binary, ~$4/mo VPS | **None provided** | Yes; API rules can be left open | **Pre-1.0**, explicitly "NOT recommended for production critical applications yet" |
| **`sql_crdt` / `sqlite_crdt` / `drift_crdt`** | Community (cachapa; Drift bridge by JanezStupar) | `drift_crdt` 2.3.0, 2026-08-15 - but **no migrations**, no web | `sqlite_crdt` web is "experimental"; `drift_crdt` declares no web platform | Yes - `crdt_sync` is a Dart WebSocket server | Row-level LWW by hybrid logical clock, with tombstones | No | `sql_crdt` last commit 2025-05-03; `crdt`/`crdt_sync` last commit **2024-11-02** |
| **Turso / libSQL embedded replicas** | Community only - `libsql_dart` 0.9.0+0.9.30, 2026-03-31, publisher `kucingtelon.com`. Turso's own docs call it "community maintained" | `drift_libsql` 0.1.0 (2025-06-05), community; `drift_hrana` is remote-only | **No offline path on web** (Rust FFI binding) | sqld yes (but 2025-02-14 binary); **Turso Sync is cloud-only** | Row-level **last-push-wins**; concurrent edits to different fields of one row lose a side | Database-scoped JWTs, no user identity | libSQL deprioritised by its own vendor; Turso Database pre-1.0 |
| **Firebase / Firestore** | `cloud_firestore` 6.9.0, 2026-08-24, first-party | None - it's a document store with its own opaque cache; you'd run two local stores | Yes, but IndexedDB-based and off by default; multi-tab is a known sharp edge | **No.** Emulators are explicitly not a self-host path | Document-level **last write wins** | Firebase Auth (rules can be opened, which means a public DB) | GA and stable; maximal vendor lock-in |
| **Ditto** | **Yes, first-party** - `ditto_live` 5.1.0, 2026-08-19, publisher `ditto.live` | **Replaces** Drift entirely - own store, own DQL query language | Supported but **RAM-only, no persistence across reloads, no P2P**; 38 MB wasm asset | Kubernetes Operator, **Private Preview** | Version-vector CRDT: LWW registers, add-wins maps | Yes, plus an auth webhook you host | Mature, but **proprietary binary licence** and no published paid prices |
| **Automerge** | **No.** `automerge` 0.0.0 (2026-08-18) is an **empty placeholder**, no publisher, repo 404s | Drift becomes a **blob store**; CRDT contents are opaque to SQL | Would need a custom JS-interop path you write | Yes - Node sync server, MIT, one Docker command | Full op-based JSON CRDT | No | Core is excellent (MIT, active); **Dart binding does not exist** |
| **Yjs** | **No usable one.** `y_crdt` 0.2.0 (no web); `yjs_dart` 1.1.15 (1 star, unproven); `ydart` 0.0.1 (2023-12-11) | Same blob problem | `y_crdt` **no web**; `yjs_dart` claims it, unverified | Yes - y-websocket, Hocuspocus, y-sweet, all MIT | Full op-based JSON CRDT | No | Core is excellent (MIT, 22.7k stars); **all Dart bindings are hobby-grade** |
| **`crdt_lf`** (new) | `crdt_lf` 4.2.0, 2026-09-07 + `crdt_lf_drift` 0.3.0 + `crdt_socket_sync` 0.8.0 | Drift used as **blob store for CRDT changes/snapshots**, not as the domain model | `crdt_lf` declares web; `crdt_lf_drift` does **not** | Yes, `crdt_socket_sync` is Dart | Op-based; Fugue for text, Observed-Remove for sets | No | Self-described "**in progress**"; drift adapter first published 2026-07-13, 0 likes |
| **ElectricSQL** | **None.** Discontinued community packages only | None | n/a | Yes, Docker + Postgres 14+ | **None - no write path at all** | No, BYO HTTP proxy | Engine GA, but **cloud winding down after Databricks acquisition** |
| **`serverpod_offline_sync`** | 0.0.5, 2026-09-05, publisher `serverpod.dev` | **No** - uses Serverpod's own model layer, not Drift | Not documented; pub.dev declares only `platform:windows` | Yes (Serverpod is self-hosted) | Delta-CRDT, field-level merge by HLC, causal-length tombstones | Serverpod's | "still in development and is not yet ready for production use"; needs Serverpod `4.0.0-rc.2` |
| **cr-sqlite** | **None** | None | Wasm build frozen at 2023-12-16, and it's wa-sqlite not sqlite3.wasm | Yes (MIT extension, no service) | **Per-column** CRDT: LWW registers, fractional index, OR-sets | No | Stalled; see §1 |

Baseline for the web column: **Drift's own web support is stable** - "Web support is now stable, but
please continue to report all issues you find"
([drift.simonbinder.eu/platforms/web](https://drift.simonbinder.eu/platforms/web/)) - on
sqlite3.wasm with OPFS preferred and IndexedDB fallback. Caveats it names: WAL mode is unsupported
on web, COOP/COEP headers give best performance but "may conflict with certain authentication
flows", Chrome on Android is limited without those headers, and Firefox private browsing falls back
to IndexedDB/in-memory.

[04-drift-sqlite-wasm-viable](../issues/04-drift-sqlite-wasm-viable.md) has since closed yes, under
seven conditions, and two of its findings bear directly on this survey:

- The web app must be served **cross-origin isolated (COOP/COEP)**. That is a hosting constraint,
  and it collides with any candidate whose auth flow uses a popup or a cross-origin script - worth
  checking against Firebase Auth, Supabase Auth and Ditto's wasm CDN loading before committing.
- **iOS Safari evicts the database after seven days of disuse**, which makes the sync layer
  load-bearing for *correctness* on web, not just convenience. Any option whose web tier is
  online-only or RAM-only (Ditto, §11) or has no web tier at all (Turso, cr-sqlite) fails this
  differently than it fails on mobile: on web the server is the only durable copy.

Also worth recording: Drift's author has stated Drift will **not** ship its own sync. On 2024-02-04
in [simolus3/drift#2880](https://github.com/simolus3/drift/discussions/2880) he called automatic
synchronisation "a monumental effort" better handled by third parties, and asked for Electric and
PowerSync to be surfaced in Drift's docs instead.

---

## 3. PowerSync

**What it is.** A sync service plus client SDKs. The service replicates from a backend database,
"partition[s] it based on what data each user should receive, and stream[s] real-time updates to
clients" ([docs.powersync.com](https://docs.powersync.com/)). The client holds a local SQLite
database. Reads stream down via buckets and checkpoints; writes land in a local `ps_crud` upload
queue that **your own backend** drains
([architecture/powersync-protocol](https://docs.powersync.com/architecture/powersync-protocol),
[architecture/client-architecture](https://docs.powersync.com/architecture/client-architecture)).
Backend sources: Postgres, MongoDB, MySQL (Beta), SQL Server (Beta), Azure DocumentDB (Alpha),
Convex (Experimental).

**Who maintains it, licence.** Journey Mobile, Inc. / JourneyApps, Denver CO; spun out of the
JourneyApps Platform in 2022 ([powersync.com/company](https://www.powersync.com/company)). Funding
and investors: **unverified** - no primary announcement found.

- Server: **FSL-1.1-ALv2**, "Copyright 2023-2026 Journey Mobile, Inc.", with a Competing-Use
  restriction and conversion to Apache-2.0 "on the second anniversary of the date we make the
  Software available"
  ([LICENSE](https://github.com/powersync-ja/powersync-service/blob/main/LICENSE)). This is
  *source-available*, not OSI open source, for the first two years of each release.
- Dart SDK: **Apache-2.0**
  ([LICENSE](https://github.com/powersync-ja/powersync.dart/blob/main/packages/powersync/LICENSE)),
  confirmed by pub.dev's `license:apache-2.0`, `license:osi-approved` tags.

**Maturity.** [Feature status](https://docs.powersync.com/resources/feature-status) lists
Dart/Flutter SDK **GA**, Sync Streams **GA**, Open Edition (self-hosting) **GA**, Enterprise
Self-Hosted **GA**; **Flutter Web Support Beta**; CLI Beta. Latest releases as of 2026-09-11:
`powersync-service` v1.26.1 (2026-09-11), `powersync.dart` v2.4.0 (2026-09-02).

**Flutter/Dart packages** (all first-party, verified publisher `powersync.com`):

| Package | Version | Published |
|---|---|---|
| [`powersync`](https://pub.dev/packages/powersync) | 2.4.0 | 2026-09-02 |
| [`powersync_core`](https://pub.dev/packages/powersync_core) | 1.8.0 | 2026-03-05 |
| [`sqlite_async`](https://pub.dev/packages/sqlite_async) | 0.14.5 | 2026-08-27 |
| [`drift_sqlite_async`](https://pub.dev/packages/drift_sqlite_async) | 0.3.1 | 2026-06-04 |

`powersync` carries platform tags android/ios/web/linux/macos/windows, 160/160 pub points, 153
likes, ~32k downloads in 30 days ([pub.dev API](https://pub.dev/api/packages/powersync)).

**Web.** Documented and working, in beta.
[flutter-web-support](https://docs.powersync.com/client-sdk-references/flutter/flutter-web-support):
storage is **OPFS** ("A fast and modern file system implementation") with an **IndexedDB** fallback
("Highly compatible with different browsers, but performance is slow"), auto-detected. Engine is
**sqlite3.wasm** via `package:sqlite3`, plus `powersync_db.worker.js` and
`powersync_sync.worker.js`, installed with `dart run powersync:setup_web`. Status quote: "Since
version 1.9.0, web support for Flutter is in a **beta** release. It is functionally ready for
production use, provided that you've tested your use cases." Web limitations: import
`package:sqlite3/sqlite3_common.dart` rather than `sqlite3.dart`; **no connection concurrency**
(single connection); no synchronous DB access and no `computeWithDatabase`.

**Drift interop.** Official, via
[`drift_sqlite_async`](https://pub.dev/packages/drift_sqlite_async), documented at
[client-sdks/orms/flutter-orm-support](https://docs.powersync.com/client-sdks/orms/flutter-orm-support)
and described there as "currently in a beta release" and "recommended for Flutter developers who
already know Drift". Its README states "This package is developed by PowerSync". Supported: all
queries, transactions and nested transactions (SAVEPOINT), cross-propagating watch/update
notifications, concurrent reads, and "Drift migrations are supported (optional)". Not supported:
Drift read-only transactions; duplicate update events are possible.

**How much of the Drift model has to change - the real cost.** PowerSync tables are **SQLite views
over a schemaless JSON store**, not tables: "the tables defined in your client-side schema are
usable in SQL queries as if they were actual SQLite tables, while in reality they are created as
SQLite views based on the schemaless JSON data being synced"
([client-architecture](https://docs.powersync.com/architecture/client-architecture)). Consequences:

- The schema is declared **twice** - once in PowerSync's `Schema`/`Table` DSL, once in Drift.
- Drift's own `CREATE TABLE` migrations do not create the synced tables.
- Escape hatch: [Raw Tables](https://docs.powersync.com/client-sdks/advanced/raw-tables) (Dart SDK
  1.18.0+) give real SQLite tables back, restoring foreign keys with `ON DELETE CASCADE`,
  expression and `GENERATED` indexes, typed columns and custom triggers. **Whether Raw Tables are
  Drift-compatible is not stated in the docs - unverified.**
- Documented gotcha: for local-only tables whose `viewName` differs from the internal table name,
  you must supply `transformTableUpdates` mapping (e.g.) `local_items` → `items`, or Drift `watch()`
  streams silently miss notifications.

**Self-host and cost.** Self-hostable via Docker image `journeyapps/powersync-service`; the CLI
sets up a Docker Compose stack
([self-hosting/getting-started](https://docs.powersync.com/self-hosting/getting-started)). Stated
limitation: "The PowerSync Dashboard is currently not available when self-hosting PowerSync."

Cloud pricing ([powersync.com/pricing](https://www.powersync.com/pricing)):

- **Free, $0/mo** - "Up to 2 GB data synced / month", "Up to 500 MB of data hosted on PowerSync
  Service", "Up to 50 peak concurrent clients", "Up to 2 PowerSync Service instances". **Projects
  deactivate after one week of inactivity.**
- **Pro, from $49/mo** - 30 GB synced/mo, 10 GB hosted, 1,000 peak concurrent clients, email
  support, no deactivation.
- Team from $599/mo; Enterprise custom.

For one user with three devices, none of the volume limits bind. The **one-week inactivity
deactivation** on the free tier is the binding constraint, and $49/mo is a steep step up from it.
Self-hosting sidesteps both, at the cost of running a Postgres plus the sync service.

**Conflict model.** PowerSync itself is unopinionated and the server is authoritative:
"The developer's app backend therefore dictates how mutations from clients are processed and
applied against the source database"
([handling-update-conflicts](https://docs.powersync.com/usage/lifecycle-maintenance/handling-update-conflicts)).
The simplest backend gives **per-field last-write-wins**; the docs suggest "deletes always win".
Mechanics: an ordered, blocking FIFO upload queue → your backend API → source DB → replicated back
down. Write checkpoints ensure a client has uploaded its own mutations before applying downloads.

**Auth.** Brings none. It verifies JWTs
([installation/authentication-setup](https://docs.powersync.com/installation/authentication-setup)):
development tokens for dev; existing providers (Supabase, Firebase, Auth0, Clerk, Keycloak, Cognito,
…); or **custom JWTs signed by your own backend and verified via your own JWKS endpoint**. Fully
replaceable - relevant to [02-accounts-or-paired-devices](../issues/02-accounts-or-paired-devices.md).

**Schema demands.**

- `id` is **implicit and mandatory, type `text`**: "The schema does not explicitly specify an `id`
  column, since PowerSync automatically creates an `id` column of type `text`". UUIDs are
  recommended, not required, because they can be generated offline
  ([define-your-schema](https://docs.powersync.com/usage/installation/client-side-setup/define-your-schema)).
  For Postgres sources the docs require "a single text-type primary key column called `id`", UUIDv4
  ([sync-rules/client-id](https://docs.powersync.com/usage/sync-rules/client-id)).
- **Only three client column types: `text`, `integer`, `real`.** No boolean (use 1/0). Timestamps
  become text. No JSON type - objects and arrays are serialised to text. Postgres
  `numeric`/`decimal` → text. Binary "can be accessed in the Sync Streams / Sync Rules, but cannot
  be used as parameters" and must be hex/base64 encoded
  ([sync-rules/types](https://docs.powersync.com/usage/sync-rules/types)).
- **No tombstone or version columns are required in your schema.** The machinery lives in
  PowerSync-owned tables `ps_data__<table>` (JSON row store) and `ps_crud` (upload queue); deletes
  and versioning ride the bucket/checkpoint protocol.

---

## 4. ElectricSQL

**Do not plan around anything written about Electric before 2026.** Two independent invalidations.

**What it is today.** A read-path-only sync engine for Postgres over HTTP. Verbatim: "Electric Sync
is a read-path sync engine for Postgres. It syncs data out of Postgres into local clients over HTTP
using a primitive called a Shape" ([electric.ax/docs/sync](https://electric.ax/docs/sync/)). And,
explicitly: "Electric does read-path sync. It syncs data out-of Postgres, into local apps and
services. **Electric does not do write-path sync**"
([docs/guides/writes](https://electric.ax/docs/guides/writes)). The server is Elixir/Erlang; the
client-side embedded database, where used at all, is **PGlite** (Postgres-in-WASM), **not SQLite**,
and **no CRDTs are documented**. The GitHub tagline is now "The agent platform built on sync", and
2026 blog output is dominated by an agents pivot ([electric.ax/blog](https://electric.ax/blog)).

**Company status.** Electric DB Inc., a Delaware corporation
([electric.ax/about](https://electric.ax/about)), **acquired by Databricks, announced 2026-08-11**,
team joining Neon. "Electric Cloud is winding down. Cloud users will need to self-host or move to
another provider." "Everything Electric has previously open sourced stays open source: Postgres
Sync, PGlite, TanStack DB, Durable Streams."
([electric.ax/blog/2026/08/11/electric-joining-databricks](https://electric.ax/blog/2026/08/11/electric-joining-databricks),
[neon.com/blog/electric-joins-neon](https://neon.com/blog/electric-joins-neon)). Deal terms
unverified.

**Licence and code health.** Apache-2.0
([LICENSE](https://github.com/electric-sql/electric/blob/main/LICENSE)). Repo not archived;
10,358 stars; last push 2026-09-09; latest `@core/sync-service@1.8.1` (2026-09-07). The engine
reached GA in March 2025 ("With version 1.0.0, Electric is now in GA. The APIs are stable and the
sync engine is ready for mission critical, production apps" -
[1.0 release post](https://electric.ax/blog/2025/03/17/electricsql-1.0-released)). The risk is
stewardship, not code quality.

**Flutter/Dart.** **None, first- or third-party, that is alive.** Electric documents a TypeScript
client and an Elixir client only ([electric.ax/llms.txt](https://electric.ax/llms.txt)). On pub.dev:

| Package | Status | Last version | Published |
|---|---|---|---|
| [`electricsql`](https://pub.dev/packages/electricsql) | **discontinued** | 0.8.2 | 2024-07-24 |
| [`electricsql_flutter`](https://pub.dev/packages/electricsql_flutter) | **discontinued** | 0.8.0 | 2024-06-11 |

Both are community packages from [SkillDevs/electric_dart](https://github.com/SkillDevs/electric_dart),
which the GitHub API reports as **archived** with last push 2024-07-24. Both carry
`isDiscontinued: true` with no replacement. They target the 2024 SQLite-based Electric that no
longer exists.

**Web / Drift / conflict / auth / schema.** No Flutter SDK, therefore no Flutter web story and no
Drift integration. No conflict model, because there is no write path - "you can implement writes in
any way you like" ([writes guide](https://electric.ax/docs/guides/writes)) lists four patterns of
increasing complexity, all of which leave merge logic entirely to you. Auth is bring-your-own via a
proxy or gatekeeper tokens: "The golden rule with Electric is that it's all just HTTP"
([auth guide](https://electric.ax/docs/guides/auth)). Schema demands land on the Postgres side only
(v14+, logical replication, a `REPLICATION`-attributed role) -
[deployment guide](https://electric.ax/docs/sync/guides/deployment).

**Self-host.** Docker image [`electricsql/electric`](https://hub.docker.com/r/electricsql/electric),
any standard Postgres 14+, plus a persistent filesystem (`ELECTRIC_STORAGE_DIR`) that survives
restarts - "Electric trades storage for low memory use and fast sync".

**Stale-page warning:** [electric.ax/pricing](https://electric.ax/pricing) still advertises Cloud
tiers (PAYG with "Under $5/mo waived", Pro $249/mo) with no wind-down notice, and the deployment
guide still calls Electric Cloud "the simplest way to use Electric". Both contradict the 2026-08-11
announcement. Whether Cloud still accepts signups is **unverified**.

---

## 5. Turso / libSQL embedded replicas

**Two products from one team.** libSQL is the C fork of SQLite plus `sqld`/libsql-server
([github.com/tursodatabase/libsql](https://github.com/tursodatabase/libsql), **MIT**). Turso
Database is the Rust rewrite, formerly "Limbo"
([github.com/tursodatabase/turso](https://github.com/tursodatabase/turso), **MIT**). The libSQL
README: "libSQL is actively maintained, but new features are being developed in Turso… If you're
starting a new project, you probably want to look into Turso." The docs say the same:
"our focus is Turso Database, a full rewrite of SQLite built from scratch"
([docs.turso.tech/libsql](https://docs.turso.tech/libsql)).

**Health.** Neither repo is archived and neither carries a deprecation banner, but the split is
stark:

| | libSQL | Turso Database |
|---|---|---|
| Last commit | 2026-08-23 | 2026-09-11 |
| Latest release | **`libsql-server-v0.24.32`, 2025-02-14** (no server release in ~19 months) | `v0.8.0-pre.11`, 2026-09-11 (prerelease) |
| Stars | 17.2k | 24.2k |

Turso Database is **pre-1.0**: "Yes - Turso powers production applications today at multiple
organizations" but "we have not yet reached 1.0", with a recommendation to keep independent backups
([README](https://github.com/tursodatabase/turso)).

Timeline of the direction change, from Turso's own blog:

- **2025-01-21** - the new multitenant cloud server "we've decided to keep this new implementation
  closed source"; "everything that runs on the client will remain strictly open source"; edge
  replicas discontinued for new users; "Users who prefer an open-source path can still self-host
  using libSQL"
  ([post](https://turso.tech/blog/upcoming-changes-to-the-turso-platform-and-roadmap)).
- **2025-03-31** - offline sync public beta, TypeScript and Rust only, "conflict detection (but
  resolution is not yet implemented)", "not yet recommended for production use… there are no
  durability guarantees, which means data loss is possible"
  ([post](https://turso.tech/blog/turso-offline-sync-public-beta)).
- **2025-10-08** - Turso Sync beta on the new engine
  ([post](https://turso.tech/blog/introducing-databases-anywhere-with-turso-sync)).
- **2026-04-24** - "you should be using Turso, not libSQL, if you are using sync"
  ([post](https://turso.tech/blog/sync-benchmark)).

**Flutter/Dart - community only.** Turso's SDK index lists TypeScript, Python, Go and Rust as
official; "Flutter / Dart" appears only under **Community SDKs**
([docs.turso.tech/sdk](https://docs.turso.tech/sdk)), and the Flutter quickstart says verbatim:
"This SDK is community maintained and may not be officially supported by Turso, or up to date with
the latest features" ([quickstart](https://docs.turso.tech/sdk/flutter/quickstart)).

| Package | Version | Published | Publisher |
|---|---|---|---|
| [`libsql_dart`](https://pub.dev/packages/libsql_dart) | 0.9.0+0.9.30 | 2026-03-31 | `kucingtelon.com` |
| [`drift_libsql`](https://pub.dev/packages/drift_libsql) | 0.1.0 | 2025-06-05 | `kucingtelon.com` |
| [`turso_dart`](https://pub.dev/packages/turso_dart) | 0.1.0 | 2026-06-14 | `kucingtelon.com` |
| [`drift_hrana`](https://pub.dev/packages/drift_hrana) | 1.0.5 | 2025-05-03 | `simonbinder.eu` (Drift's author) |
| [`hrana`](https://pub.dev/packages/hrana) | 0.4.3 | 2026-09-02 | `simonbinder.eu` |

Usage is very small: `libsql_dart` 311 downloads/30d and 20 likes; `drift_libsql` 34 downloads/30d.
Also note `drift_libsql` is pinned to `libsql_dart: ^0.6.0` while `libsql_dart` is at 0.9.x -
version skew between the two community packages.

**Web.** No offline path. `libsql_dart` is a Rust FFI binding (`flutter_rust_bridge: 2.12.0`,
`native_toolchain_rust` - verified in its
[pubspec](https://pub.dev/api/packages/libsql_dart)), and embedded replicas need a local file plus
`sync()`. Neither its README nor Turso's quickstart mentions web. Its pub.dev `platform:web` tag is
contradicted by `drift_libsql`, which depends on it and declares **no** web - treat that tag as an
analyser artifact. Browser support exists only in the JS stack: "Browser applications require the
dedicated `@tursodatabase/sync-wasm` package"
([post](https://turso.tech/blog/introducing-databases-anywhere-with-turso-sync)); no Dart
equivalent. `drift_hrana` does work on web but is **remote-only**, with no offline capability, and
Drift's own page warns "streamed queries will not work across different clients connecting to the
same database like this, as sqld has no support for that at the moment"
([drift.simonbinder.eu/platforms/libsql](https://drift.simonbinder.eu/platforms/libsql/)).

**Drift interop.** Documented by Drift, not by Turso, at
[drift.simonbinder.eu/platforms/libsql](https://drift.simonbinder.eu/platforms/libsql/), which also
notes `drift_hrana` is "not part of the core Drift project". **No Drift integration exists for the
new Turso engine or Turso Sync.**

**Self-host and cost.** `sqld`/libsql-server is self-hostable
([setup docs](https://docs.turso.tech/libsql/server/setup),
[Docker](https://github.com/tursodatabase/libsql/blob/main/docs/DOCKER.md), image
`ghcr.io/tursodatabase/libsql-server:latest`) - but with a 2025-02-14 binary. **Turso Sync on the
new engine is documented as cloud-only**, requiring "your Turso Cloud URL (`turso://...`)" plus an
auth token ([docs.turso.tech/sync/usage](https://docs.turso.tech/sync/usage)). Pricing
([turso.tech/pricing](https://turso.tech/pricing)): **Free $0/mo** - 100 databases, 5 GB storage,
500M monthly rows read, 10M monthly rows written, 3 GB monthly syncs, 1-day PITR. **Developer
$4.99/mo** - unlimited databases, 9 GB storage, 2.5B rows read, 25M rows written, 10 GB monthly
syncs, 10-day PITR. Cheapest paid tier of anything surveyed.

**Conflict model.** Embedded replicas: **writes go to the remote primary by default**, not local -
"Writes are sent to the remote primary database configured at `syncUrl` by default. They are NOT
written to the local file first"; local writes need `offline: true`
([embedded replicas](https://docs.turso.tech/features/embedded-replicas/introduction)). Turso Sync:
local writes always, `push()` sends logical mutations, `pull()` applies remote page changes, and
the strategy is **last-push-wins** - "if the same row is modified on different devices, the version
that is pushed last will take precedence" ([sync/usage](https://docs.turso.tech/sync/usage)); a
transform hook allows custom resolution. **This is row-level, not per-column**: two devices editing
different fields of the same row lose one side.

**Auth.** Database-scoped JWTs, `full-access` or `read-only`, with an expiry
([cli/db/tokens/create](https://docs.turso.tech/cli/db/tokens/create)). No end-user identity system.
Tokens cannot be retrieved after creation and cannot be revoked individually.

**Schema demands.** None for libSQL embedded replicas - it is page-level physical replication of an
ordinary SQLite file. Turso Sync uses logical CDC; no schema annotation requirement is documented,
and how schema changes behave under Turso Sync is **unverified**.

---

## 6. cr-sqlite

**What it is.** A runtime-loadable SQLite/libSQL extension adding multi-master CRDT replication:
"CR-SQLite is a run-time loadable extension for SQLite and libSQL. It allows merging different
SQLite databases together that have taken independent writes"
([README](https://github.com/vlcn-io/cr-sqlite)). Maintainer Matt Wonlaw (`tantaman`) / One Law LLC.
Licence **MIT**, "Copyright (c) 2023 One Law LLC"
([LICENSE](https://raw.githubusercontent.com/vlcn-io/cr-sqlite/main/LICENSE)). Stated cost:
"inserts into CRRs are 2.5x slower than inserts into regular SQLite tables. Reads are the same
speed."

**Health - stalled.** Latest release **`v0.16.3`, 2024-01-17**; no release in ~20 months. npm
`@vlcn.io/crsqlite-wasm` frozen at **0.16.0, 2023-12-16**. The maintainer's last commit is
**2024-01-17**; everything after is outside contributors, and the 2026 commits (2026-08-04,
2026-08-10) are pure packaging: "build all android abis", "build windows arm64 loadable", "fix ios
simulator build", "align android loadable to 16 kb page size". The JS/wasm repo
[vlcn-io/js](https://github.com/vlcn-io/js) last committed **2023-12-16**. No archive banner
(GitHub API `archived: false`); 3,790 stars; still ~5.3k npm downloads/month for the wasm build.

**Flutter/Dart - none.** pub.dev has no cr-sqlite binding; `cr_sqlite`, `crsqlite`, `crsql` all 404
on the pub.dev API. (`sqlite_crdt` and `drift_crdt`, which surface in searches, are a *different*
project - see §9.) A theoretical route exists via `package:sqlite3`'s
[`SqliteExtension`](https://pub.dev/documentation/sqlite3/latest/sqlite3/SqliteExtension-class.html),
but that documentation notes "In sqlite3 builds created through sqlite3_flutter_libs, dynamic
extensions are omitted from sqlite3 due to security concerns", so you would statically link and
register `sqlite3_crsqlite_init` yourself. **Nobody has published this; feasibility unverified.**

**Web.** A wasm build exists but lives in the dead JS repo and is built on `@vlcn.io/wa-sqlite`,
**not** the `sqlite3.wasm` Drift's `WasmDatabase` uses. Getting cr-sqlite into Flutter web under
Drift would mean compiling cr-sqlite into a custom sqlite3 wasm - **no such artifact or
documentation found**.

**Conflict model - the best on offer, on paper.** Per the README (Approach 1, "History-free CRDTs",
what ships today): "Keeps no history / only keeps the current state"; "Automatically handles merge
conflicts. No options for manual merging"; "Tables are Grow Only Sets or variants of Observe-Remove
Sets"; "Rows are maps of CRDTs. The column names being the keys, column values being a specific
CRDT type"; "Columns can be counter, fractional index or last write wins CRDTs." Default is LWW per
column, ties broken by taking the largest value. Availability caveat from the README itself: "LWW,
Fractional Index, Observe-Remove sets are available now. Counter and rich-text CRDTs are still
being implemented", confirmed at
[vlcn.io/docs/cr-sqlite/crdts/column-crdts](https://vlcn.io/docs/cr-sqlite/crdts/column-crdts)
("Counter CRDTs are currently not yet supported"). Approach 2 (causal event log, developer-defined
resolution) is "To be implemented in v2" - not shipped. Change transport is the `crsql_changes`
virtual table (`table, pk, cid, val, col_version, db_version, site_id, cl, seq`): select rows where
`db_version > x` to extract, insert into it to merge.

**Self-host / auth.** No service, no pricing, no auth, no network layer at all. Transport is yours.
The reference WebSocket server in `vlcn-io/js` is from 2023.

**Schema demands - restrictive.** Every synced table needs `SELECT crsql_as_crr('table_name')` and
a **primary key**. `ALTER TABLE` on a CRR is not allowed directly; you must wrap it in
`crsql_begin_alter` / `crsql_commit_alter`. Prohibited on CRRs
([vlcn.io/docs/cr-sqlite/constraints](https://vlcn.io/docs/cr-sqlite/constraints)): **checked
foreign-key constraints** ("Foreign keys and joins are allowed but the constraint cannot be
checked"), **unique constraints other than the primary key**, and **check constraints depending on
other columns**. The extension must be loaded as the first operation on every connection, and
`SELECT crsql_finalize()` must be called before closing it.

---

## 7. Supabase

**No first-party offline story.** `supabase.com/docs/guides/local-development/offline-first` returns
**404**, and the
[Flutter quickstart](https://supabase.com/docs/guides/getting-started/quickstarts/flutter) never
mentions offline or local caching. Supabase's own `local-first` blog tag
([supabase.com/blog/tags/local-first](https://supabase.com/blog/tags/local-first)) holds exactly
three posts, all from 2024, all recommending third parties; the Flutter one
([offline-first-flutter-apps](https://supabase.com/blog/offline-first-flutter-apps), 2024-10-08)
recommends **Brick** (a GetDutchie package) and does not mention Drift. Partner pages exist for
PowerSync, ElectricSQL, RxDB and Replicache under `supabase.com/partners/` - their body text is
client-rendered and **unverified**, only their existence is confirmed.

**Package.** [`supabase_flutter`](https://pub.dev/packages/supabase_flutter) **2.17.2**, published
**2026-08-14**, verified publisher `supabase.io`, MIT, platforms Android/iOS/Web/macOS/Windows/Linux,
`sdk >=3.9.0`, `flutter >=3.35.0`. A `3.0.0-dev.3` prerelease also exists. Repo
[supabase/supabase-flutter](https://github.com/supabase/supabase-flutter), last push 2026-09-11.
Dependencies include `shared_preferences` for session persistence and **no local database at all** -
it is an online client with no offline write queue.

**Self-host.** Docker Compose ([docs](https://supabase.com/docs/guides/self-hosting/docker)) running
Studio, Envoy/Kong, Auth, PostgREST, Realtime, Storage, postgres-meta, Postgres, Edge Runtime,
Supavisor, imgproxy. Stated minimums **4 GB RAM / 2 CPU / 40 GB SSD**; recommended 8 GB+ / 4 cores.
Self-hosted is one project only, without branching, managed backups/PITR, or the management API, and
support is community-only. The local-dev CLI is explicitly "not hardened for production and must not
be exposed to external traffic".

**Cost.** [supabase.com/pricing](https://supabase.com/pricing): **Free** - 500 MB database, 5 GB
egress, 50,000 MAU, 1 GB storage, but "Free projects are paused after 1 week of inactivity. Limit of
2 active projects". **Pro $25/month** - 100,000 MAU, 8 GB disk, 250 GB egress, 100 GB storage,
7-day daily backups, $10/mo compute credits.

**Auth.** Not mandatory. RLS is "not technically required" but "A table in an exposed schema without
RLS is readable and writable by any role with a grant on it"
([RLS docs](https://supabase.com/docs/guides/database/postgres/row-level-security)). The
[securing-your-api](https://supabase.com/docs/guides/api/securing-your-api) guide explicitly covers
apps that "don't use Supabase Auth… and rely on the `anon` role", with application-managed API keys
checked in a pre-request function. The `apikey` header is mandatory and non-configurable. The
`service_role` key bypasses RLS entirely but must never ship in a client.

**Conflict model / schema demands.** None, in both directions. It is plain Postgres + PostgREST: no
tombstones, no version columns, no id-type requirement - and correspondingly no conflict resolution
and no client write queue. Last write to land wins at row level.

**As transport with Drift: yes, cleanly.** `supabase_flutter` has no local store, so Drift stays
authoritative and PostgREST is just a REST endpoint. You then hand-roll everything in §15. Whether
Supabase Realtime replays messages missed during a disconnect is **unverified** - the
[Realtime docs](https://supabase.com/docs/guides/realtime) do not state it, and that matters for a
change-log design.

---

## 8. PocketBase

**What it is.** "PocketBase is an open source backend consisting of embedded database (SQLite) with
realtime subscriptions, builtin auth management, convenient dashboard UI and simple REST-ish API"
([README](https://github.com/pocketbase/pocketbase)). A single prebuilt binary for
Linux/Windows/macOS on x64 and ARM64.

**Licence.** **MIT**, "Copyright (c) 2022 - present, Gani Georgiev", verified from
[LICENSE.md](https://raw.githubusercontent.com/pocketbase/pocketbase/master/LICENSE.md).

**Still pre-1.0, and says so.** Latest release **v0.40.3, 2026-09-06**. Verbatim from
[pocketbase.io/docs](https://pocketbase.io/docs/): "Please keep in mind that PocketBase is still
under active development and full backward compatibility is not guaranteed before reaching v1.0.0.
PocketBase is NOT recommended for production critical applications yet, unless you are fine with
reading the changelog and applying some manual migration steps from time to time." v0.40.0
(2026-08-23) shipped a breaking change. No v1.0 date is published.

**Offline / sync: none.** Neither the README, the docs, nor the FAQ mentions offline support, a
local cache or a sync engine. SQLite lives on the **server**; clients talk REST + SSE.

**Dart SDK.** [`pocketbase`](https://pub.dev/packages/pocketbase) **0.25.1**, published
**2026-09-05**, verified publisher `pocketbase.io`, first-party ("Official Multi-platform Dart SDK
for interacting with the PocketBase Web API"), MIT, all six platforms, 291 likes, 160 pub points.
Note the SDK is itself pre-1.0 and versions independently of the server.

**Cost.** Self-hosted only; no vendor. The FAQ ([pocketbase.io/faq](https://pocketbase.io/faq/))
cites handling "10 000+ persistent realtime connections on a cheap $4 Hetzner CAX11 VPS" - the
closest thing to a stated floor. No formal hardware minimum is published, and
[going-to-production](https://pocketbase.io/docs/going-to-production/) covers only file-descriptor
tuning. Scaling is vertical, single-server. There is **no built-in data import/export tooling**.

**Single-maintainer risk, quantified.** GitHub contributors API: 44 contributors, of which
`ganigeorgiev` has **2,478 commits** and the next-highest has **5**. The FAQ states it is "a
personal open source project with no paid team behind it" with "no promises for maintenance and
support beyond what is already available"; donations are no longer accepted. 61k stars.

**Auth.** Stateless bearer tokens; multiple auth collections; password, email OTP, OAuth2 (15+
providers), MFA since v0.23 ([docs](https://pocketbase.io/docs/authentication/)). Authorization is
per-collection API rules, which can be left open. `_superusers` bypass all rules.

**Conflict model / schema demands.** No conflict model - last HTTP write overwrites. Auth
collections force system fields `email`, `emailVisibility`, `verified`, `password`, `tokenKey`
([collections docs](https://pocketbase.io/docs/collections/)); an `AutodateField` exists for
`created`/`updated`. Record `id` format/length and any built-in soft-delete are **unverified** -
the collections page does not document them, and no tombstone mechanism is mentioned.

**As transport with Drift: yes, trivially** - the Dart SDK is a thin REST/SSE client with no local
persistence. The oddity is that PocketBase *is* a SQLite server, so you would run SQLite on both
ends with a hand-written protocol between them, which is very close to §15 with a dashboard
attached.

---

## 9. `sql_crdt` / `sqlite_crdt` / `drift_crdt` (the cachapa stack)

A Dart-native CRDT layer over SQL, plus a community Drift bridge. This is the closest thing to a
"pure Dart, no vendor" answer that already exists.

| Package | Version | Published | Publisher | Last repo commit |
|---|---|---|---|---|
| [`crdt`](https://pub.dev/packages/crdt) | 5.1.3 | 2024-11-02 | `cachapa.net` | 2024-11-02 |
| [`sql_crdt`](https://pub.dev/packages/sql_crdt) | 3.0.3 | 2025-05-03 | `cachapa.net` | 2025-05-03 |
| [`sqlite_crdt`](https://pub.dev/packages/sqlite_crdt) | 3.0.4 | 2025-10-27 | `cachapa.net` | 2025-10-27 |
| [`postgres_crdt`](https://pub.dev/packages/postgres_crdt) | 3.0.3 | 2025-05-03 | `cachapa.net` | - |
| [`crdt_sync`](https://pub.dev/packages/crdt_sync) | 1.0.10 | 2024-11-02 | `cachapa.net` | **2024-11-02** |
| [`drift_crdt`](https://pub.dev/packages/drift_crdt) | 2.3.0 | 2026-08-15 | `janezstupar.com` | 2026-08-15 |

**Schema demands.** From [sql_crdt](https://github.com/cachapa/sql_crdt): "Every table gets 3
columns automatically added: `is_deleted`, `hlc`, and `modified`." `drift_crdt`'s README requires
**four** columns per table - `is_deleted` (integer), `hlc` (string), `node_id` (string), `modified`
(string) - configurable per table via `onlyCrdtTables` / `excludeCrdtTables`
([drift_crdt README](https://github.com/JanezStupar/drift_crdt)). Deletes are soft only, and
`sql_crdt` warns that "Because deleted records are only flagged as deleted, they may need to be
sanitized in order to be compliant with GDPR and similar legislation." The precise conflict rule
(row-level vs field-level) is **not stated in the README - unverified**; the column layout (one
`hlc` per row, not per column) implies **row-level last-write-wins by hybrid logical clock**, but
that is an inference, not a quote.

**Drift bridge caveats - significant.** From `drift_crdt`'s own README: "At the moment migrations
are not supported" because it works by hijacking SQL queries; "Hasn't been tested on iOS and
Android yet"; and it requires local **dependency overrides** for `sqlite_crdt`, `postgres_crdt` and
`sql_crdt` because of upstream modifications not yet accepted. pub.dev grants it **50/160 points**,
16 likes, 225 downloads/30d, and declares **no platform tags at all** (no android, ios or web) -
compare `sqlite_crdt`, which declares web and describes it as "experimental support for Flutter Web,
thanks to sqflite_common_ffi_web". `drift_crdt` is built on `sqflite_common`, which is a different
web path from Drift's own sqlite3.wasm/OPFS stack.

**Sync server.** [`crdt_sync`](https://github.com/cachapa/crdt_sync), "A dart-native turnkey
solution for painless network synchronization" - a Dart WebSocket server you host. Last commit
**2024-11-02**.

**Net:** the core (`sqlite_crdt`) is alive but slow-moving; the Drift bridge is a one-person side
project with no migrations and no verified mobile or web support; the sync server has not been
touched in ~10 months.

---

## 10. Firebase / Cloud Firestore

**Offline persistence - what it actually guarantees.** From
[enable-offline](https://firebase.google.com/docs/firestore/manage-data/enable-offline), verbatim:
"Offline persistence is supported only in Android, Apple, and web apps" and "Pipeline operations
don't support offline persistence". Conflict model, verbatim: **"For multiple changes to the same
document, it's last write wins"** - document-level, no field merge, no conflict hook. Transactions
**fail offline**: "Transactions will fail when the client is offline"
([transactions](https://firebase.google.com/docs/firestore/manage-data/transactions)). Default cache
threshold **100 MB**, configurable down to 1 MB or `CACHE_SIZE_UNLIMITED`.

**Web.** Works, but differently and with sharp edges. On mobile the disk cache is on by default; on
web it is **off by default** and IndexedDB-backed, and you must pick
`persistentSingleTabManager()` or `persistentMultipleTabManager()`. The legacy `enablePersistence()`
path fails with `failed-precondition` when multiple tabs are open and `unimplemented` on
unsupporting browsers. All tabs must share the same persistence configuration. On the Flutter side
`Settings.persistenceEnabled` is documented only as "Attempts to enable persistent storage, if
possible"
([Settings class](https://pub.dev/documentation/cloud_firestore/latest/cloud_firestore/Settings-class.html)),
and FlutterFire has long-standing issues on exactly this surface -
[#12034](https://github.com/firebase/flutterfire/issues/12034),
[#9929](https://github.com/firebase/flutterfire/issues/9929),
[#12553](https://github.com/firebase/flutterfire/issues/12553),
[#10259](https://github.com/firebase/flutterfire/issues/10259) (current open/closed status
**unverified**).

**Package.** [`cloud_firestore`](https://pub.dev/packages/cloud_firestore) **6.9.0**, published
**2026-08-24**, verified publisher `firebase.google.com`, BSD-3-Clause, platforms Android, iOS,
macOS, Web, Windows, Flutter Favorite. Its own description advertises offline support "on Android
and iOS" and does not claim it for web.

**Self-host: not possible.** From [emulator-suite](https://firebase.google.com/docs/emulator-suite):
"Do not attempt to use these emulators as 'self-hosted' versions of Firebase services. They are
built for accuracy, not performance or security, and are not appropriate to use in production."

**Cost.** [firebase.google.com/pricing](https://firebase.google.com/pricing): **Spark (free)** -
1 GiB stored, 50K document reads/day, 20K writes/day, 20K deletes/day, 10 GiB/month egress. No
inactivity-pausing policy is stated, unlike Supabase and PowerSync. **Blaze** - pay-as-you-go on
stored data, egress and per-operation counts. Firebase Auth is free to 50K MAU on both plans.

**Auth.** Not required - `allow read: if true` is a documented pattern
([rules basics](https://firebase.google.com/docs/rules/basics)) - but the docs are blunt that
"Firebase allows clients direct access to your data, and Firebase Security Rules are the only
safeguard blocking access for malicious users". For a three-device personal app, open rules mean a
publicly writable database.

**Lock-in.** Managed export writes a Firestore-proprietary format to a Cloud Storage bucket,
**requires Blaze**, and "Exporting data from Cloud Firestore will incur one read operation per
document exported" ([export-import](https://firebase.google.com/docs/firestore/manage-data/export-import)).
Import targets are Firestore or BigQuery only; there is no documented path to a portable format.

**Cost to a Drift app specifically.** `cloud_firestore` always maintains its own cache - you can
switch it to memory-only but not off. A Drift-authoritative design therefore runs **two local
stores side by side**: Drift's SQLite (queryable, relational, authoritative) and Firestore's own
opaque IndexedDB/disk cache. Every listener event must be translated document-by-document into Drift
rows. Firestore has no joins, no SQL, no schema migrations and no relational constraints, so the
federated Habit/Task/Session model would be hand-mapped into collections and back on every sync,
with per-document read/write billing on each pass under Blaze.

---

## 11. Ditto

**What it is.** An "Edge Sync Platform": an embedded document store on device ("Small Peer") plus an
optional server ("Big Peer" / "Ditto Server")
([docs.ditto.live](https://docs.ditto.live/home/introduction),
[ditto.com/products/server](https://www.ditto.com/products/server)). Vendor is **DittoLive
Incorporated**. Reported $82M Series B in March 2025 appears only in secondary press
([TechCrunch](https://techcrunch.com/2025/03/12/ditto-lands-82m-to-synchronize-data-from-the-edge-to-the-cloud/));
no primary announcement found - **unverified**.

**Licence - proprietary.** The `LICENSE` inside the `ditto_live` 5.1.0 tarball is the **"Ditto
Binary License", © 2024 DittoLive Incorporated**: binary-only redistribution, with "You agree not to
attempt to decompile, disassemble, reverse engineer or otherwise discover the source code." pub.dev
classifies it `license:unknown`
([score API](https://pub.dev/api/packages/ditto_live/score)). The auxiliary `ditto_flutter_tools` is
MIT but is debug tooling only. **This is the only closed-source option in the survey.**

**Flutter SDK - first-party and current.** [`ditto_live`](https://pub.dev/packages/ditto_live)
**5.1.0, published 2026-08-19**, verified publisher **`ditto.live`**, with weekly dev builds
(`5.2.0-dev-weekly.20260910.2377`, 2026-09-10). 15 likes, 5,277 downloads/30d. Requires Flutter
≥3.24.0 ([compatibility](https://docs.ditto.live/sdk/latest/compatibility/flutter)).

**Web - the deal-breaker for local-first.** Ditto's own Flutter install guide, "Considerations for
Web": *"The web platform utilizes an in-memory Ditto store, meaning data is not retained across page
reloads"* and *"direct peer-to-peer synchronization with other devices is not supported"*
([install guide §6](https://docs.ditto.live/sdk/latest/install-guides/flutter#step-6-web-browser-support)).
The JS SDK docs and [FAQ](https://docs.ditto.live/home/faq) confirm no `localStorage`,
`sessionStorage` or `IndexedDB`. On web, Ditto is **online-only**. Also note payload: the bundled
`lib/assets/ditto.wasm` is **37.9 MB uncompressed** (whole package 38 MB); the docs recommend
serving it compressed from a CDN via `wasmUrl`/`wasmShimUrl`.

**Pricing - partially public.** [ditto.com/pricing](https://www.ditto.com/pricing) lists Free
(10 cloud device connections, 2 GB storage, no SLA), Pro (1,000+ connections, 50 GB, 99% SLA -
"Contact Us") and Enterprise (custom, 99.95% SLA - "Contact Us"). **No dollar figures are published
for any paid tier.** Three devices fit inside the Free limits as stated.

**Self-hosting - Private Preview.** The Ditto Operator deploys "Big Peer (aka Ditto Server) … to
your own self-hosted Kubernetes environment", labelled **Private Preview**, requiring Kubernetes
≥1.31, Helm and cert-manager, pulling `oci://quay.io/ditto-external/ditto-operator`
([operator quickstart](https://docs.ditto.live/ditto-server/operator/operator-quickstart)). A
lighter "Ditto Edge Server" is **Coming Soon / waitlist**
([products/edge-server](https://www.ditto.com/products/edge-server)). Commercial terms for
self-hosting are not published.

**Conflict model.** CRDTs over per-document **version vectors**, with three value types: `REGISTER`
(last-write-wins), `MAP` (add-wins, field-level delta sync), `ATTACHMENT` (LWW pointer plus
on-demand blob). Docs explicitly warn *"Avoid using `arrays` in Ditto"* because of merge conflicts.
Causal consistency holds within a database ID
([syncing data](https://docs.ditto.live/key-concepts/syncing-data)).

**Data model.** JSON-like documents in collections, `_id` primary key (can be composite), soft size
limit 256 KiB / hard 5 MiB ([document model](https://docs.ditto.live/key-concepts/document-model)).
**DQL is a real SQL-ish query language** - `SELECT * FROM cars WHERE color = 'blue'` - and 5.1 added
`JOIN` across local collections plus an `ADVISE` index advisor
([release notes](https://docs.ditto.live/sdk/latest/release-notes/flutter)). Sync is driven by
subscription queries written in DQL.

**Auth.** Three mechanisms
([auth docs](https://docs.ditto.live/key-concepts/authentication-and-authorization)): Development
Mode / Online Playground (a single shared token, explicitly *"not recommended for production"*);
Online with Authentication (your IdP issues a JWT → Ditto Server → **an auth webhook you write and
host** → per-collection read/write permission queries); and Offline Shared Key. Hard constraint:
*"Ditto enforces that permissions can only be specified on the immutable `_id` field"* - access
control must be baked into document ids. The local database is **not encrypted at rest** (FAQ).

**Cost to a Drift app.** Ditto **replaces** SQLite/Drift as the source of truth: its own embedded
store, on-disk format, query language, observers and indexes. The data layer is rewritten against
DQL documents; Drift codegen, typed DAOs and migrations go away; the federated Habit/Task/Session
rows become `_id`-keyed documents that must avoid arrays. And on web you would still need a second,
persistent store, because Ditto's web store is RAM-only.

---

## 12. Automerge and Yjs as CRDT libraries

Both are mature, MIT-licensed, actively developed - **in JavaScript and Rust**. Neither has a usable
Dart binding, and both sit *beside* SQLite rather than in it.

### Automerge

Core: [github.com/automerge/automerge](https://github.com/automerge/automerge), **MIT**, 6,592
stars, last push 2026-09-11, maintained full-time by Alex Good and Orion Henry at Ink & Switch (per
the README "Status" section). Current versions: JS `@automerge/automerge` **3.4.1** (2026-08-12);
Rust crate `automerge` **0.11.0** ([crates.io](https://crates.io/crates/automerge)). A C FFI lives
in-tree at `rust/automerge-c`.

**Dart bindings - effectively none.** [`automerge`](https://pub.dev/packages/automerge) on pub.dev
is **0.0.0, published 2026-08-18, no publisher** (`publisherId: null`). Its tarball contains only
`CHANGELOG.md` ("## 0.0.0 – WIP"), `LICENSE`, `README.md` ("WIP"), `analysis_options.yaml` and
`pubspec.yaml` - **no `lib/` directory and no code**, 20 KB total. It is a name placeholder, and its
pub.dev platform and `wasm-ready` tags are meaningless because an empty package trivially supports
everything. Its `repository:` points at `github.com/graddotdev/automerge`, which returns **HTTP
404**. `automerge_dart` and `dart_automerge` do not exist on pub.dev. On GitHub the only Dart work
is [aran/automerge-flutter](https://github.com/aran/automerge-flutter) (a `flutter_rust_bridge`
proof of concept, last push 2023-10-12, 4 stars) and
[savaki/dart-automerge](https://github.com/savaki/dart-automerge) (last push 2021-02-22, 1 star).
The Automerge org publishes JS, Rust, Swift, Java, Go, Python and C - **no Dart**.

Adopting it means writing your own FFI layer over `automerge-c`/Rust for iOS and Android **plus a
separate JS-interop path for web**, and maintaining both.

**Sync servers.** [automerge-repo](https://github.com/automerge/automerge-repo) (MIT, last push
2026-09-11; last stable npm `2.5.6` on 2026-05-18, current `latest` dist-tag is `2.6.0-alpha.3`) has
**no Dart implementation**. [`automerge-repo-sync-server`](https://github.com/automerge/automerge-repo-sync-server)
is MIT Node/Express, runnable as `docker run ghcr.io/automerge/automerge-repo-sync-server:main` with
just `PORT` + `DATA_DIR`; its README calls it "unsecured … partly for demonstration purposes but
it's also a reasonable way to run a public sync server". Last push 2025-10-20.
[automerge-repo-rs](https://github.com/automerge/automerge-repo-rs) states its disk layout and
WebSocket protocol are **not compatible** with the JS implementation; the compatible Rust effort is
the experimental [alexjg/samod](https://github.com/alexjg/samod).

**Storage shape.** automerge-repo's `StorageAdapter` is a plain key/value blob store; documents are
**binary incremental-change chunks** keyed by `[docId, "incremental", hash]`
([storage docs](https://automerge.org/docs/reference/repositories/storage/)). Drift would host a
blob table. **The CRDT is opaque to SQL** - no `WHERE`, no `JOIN`, no index into document contents.
Querying means materialising the document in memory, or maintaining a hand-written projection into
real Drift tables rebuilt on every merge. That projection layer is the actual cost.

### Yjs

Core: [github.com/yjs/yjs](https://github.com/yjs/yjs), **MIT** (LICENSE file reads "The MIT License
(MIT), Copyright (c) 2023 Kevin Jahns / RWTH Aachen"; GitHub's API reports `NOASSERTION`), 22,782
stars, last push 2026-09-07. npm `latest` = **13.6.32** (2026-08-04); `v14.0.0-rc.26` tagged
2026-09-07, so v14 is still pre-release. Rust port [y-crdt/y-crdt](https://github.com/y-crdt/y-crdt)
(`yrs` **0.27.4**, 2026-08-22), MIT.

**Dart - three options, none solid.**

| Package | Version | Published | Publisher | Web? | Signal |
|---|---|---|---|---|---|
| [`y_crdt`](https://pub.dev/packages/y_crdt) | 0.2.0 | 2026-07-25 | none (`publisherId: null`) | **No** | Only 2 versions ever (0.0.1 Mar 2024). Not a Dart port: runs `yrs` as a WASM component inside [wasm_run](https://github.com/juancastillo0/wasm_run); its dependency `wasm_wit_component` has no web support. 7 likes, 125 dl/30d |
| [`yjs_dart`](https://pub.dev/packages/yjs_dart) | 1.1.15 | 2026-02-22 | none | Declares web | Pure-Dart translation of yjs v14.0.0-22, claims binary compat. Repo [jagtesh/yjs-dart](https://github.com/jagtesh/yjs-dart) has **1 star**, last push 2026-02-22. Tarball ships a `GEMINI.md` and the README's install instructions are wrong (`dart pub add yjs`) - an LLM-assisted port that looks unexercised. **1 like, 162 dl/30d** |
| `y_dart` / `dart_yjs` / `yjs` | - | - | - | - | **Do not exist on pub.dev.** Unpublished attempts: [britannio/y_dart](https://github.com/britannio/y_dart) (2024-09-15), [lyming99/ydart](https://github.com/lyming99/ydart) (2024-09-30), [graknol/yjs-dart-crdt](https://github.com/graknol/yjs-dart-crdt) (2025-09-09) |

Note `ydart` **does** exist on pub.dev at 0.0.1, published **2023-12-11**, a Dart binding of Yrs -
three years stale, 5 downloads/30d.

**Sync servers - many, all self-hostable, protocol language-agnostic.** The Yjs README lists
y-websocket, y-redis, y-sweet, ypy-websocket (Python), yrs-warp (Rust) and Hocuspocus as
interchangeable. [`y-websocket`](https://github.com/yjs/y-websocket) MIT, npm 3.1.0 (2026-08-06).
[`hocuspocus`](https://github.com/ueberdosis/hocuspocus) MIT, `@hocuspocus/server` 4.7.0
(2026-09-09), 2,576 stars, SQLite persistence and auth built in - the most actively maintained.
[`y-sweet`](https://github.com/jamsocket/y-sweet) MIT, Rust, S3 or filesystem persistence, last push
2025-12-04 (~9 months stale). Because the protocol is on the wire, any correct Dart implementation
of y-protocols can talk to all of them - but that correctness rests entirely on whichever shaky Dart
binding you pick.

**Storage shape.** Same as Automerge: a `Y.Doc` is an opaque binary update log, persisted in Flutter
as BLOB rows. Nothing inside is SQL-queryable or indexable. Yjs is the strongest of the three for
*text* (YText plus UndoManager) and the weakest for "query my data" ergonomics - which is the wrong
trade for habits, tasks and sessions.

### The structural point common to both

Automerge and Yjs are **document CRDTs**, not row CRDTs. Adopting either inverts the premise that
Drift rows are the queryable truth: Drift becomes a blob store, and every list, filter, join and
aggregate in the timeline has to be served either from an in-memory materialised document or from a
hand-maintained projection table. `sqlite_crdt` (§9) and `crdt_lf`'s SQL-facing side are the only
Dart CRDT options that keep data in SQL rows instead of opaque blobs.

---

## 13. `crdt_lf` - a newer Dart-native CRDT

Worth recording because it is the only actively-developed Dart CRDT stack with a Drift adapter, but
it is very young.

| Package | Version | Published | First published | Likes / dl30 |
|---|---|---|---|---|
| [`crdt_lf`](https://pub.dev/packages/crdt_lf) | 4.2.0 | 2026-09-07 | - | 7 / 599 |
| [`crdt_lf_drift`](https://pub.dev/packages/crdt_lf_drift) | 0.3.0 | 2026-09-07 | **2026-07-13** | 0 / 184 |
| [`crdt_lf_sqlite`](https://pub.dev/packages/crdt_lf_sqlite) | 0.3.0 | 2026-09-07 | - | 0 / 182 |
| [`crdt_lf_flutter`](https://pub.dev/packages/crdt_lf_flutter) | 0.5.0+1 | 2026-08-29 | 2026-07-17 | 0 / 169 |
| [`crdt_socket_sync`](https://pub.dev/packages/crdt_socket_sync) | 0.8.0 | 2026-09-07 | 2025-06-14 | 2 / 220 |

MIT, single maintainer ([MattiaPispisa/crdt](https://github.com/MattiaPispisa/crdt)). Algorithms:
**Fugue** for text (to minimise interleaving), **Observed-Remove** for conflict resolution, movable
lists, nested CRDTs via flat references
([core README](https://raw.githubusercontent.com/MattiaPispisa/crdt/main/packages/core/crdt_lf/README.md)).
Maturity statement, verbatim: "This library is currently **in progress** and under active
development. While all existing functionality is thoroughly tested, we are continuously working on
improvements and new features."

**The important structural point:** `crdt_lf_drift` is described as a "drift storage adapter for
CRDT LF library objects, providing persistence for **Change and Snapshot objects**". Drift is used
as a **blob store for CRDT operations**, not as the relational domain model. That inverts the
premise of this project - Drift rows would no longer be the queryable truth; the CRDT document
would be, with Drift as its log. Also note `crdt_lf_drift` and `crdt_socket_sync` declare **no web
platform** on pub.dev, while `crdt_lf` and `crdt_lf_flutter` do.

---

## 14. `serverpod_offline_sync`

A late-breaking option, recorded because it is the only Dart-native full-stack sync product with an
official-looking publisher.

Packages `serverpod_offline_sync`, `serverpod_offline_sync_client`,
`serverpod_offline_sync_server`, all **0.0.5, published 2026-09-05**, verified publisher
**`serverpod.dev`**, BSD-3-Clause, repo
[marcelomendoncasoares/serverpod_offline_sync](https://github.com/marcelomendoncasoares/serverpod_offline_sync)
(a personal repo despite the publisher). Only five versions exist. They depend on
**`serverpod 4.0.0-rc.2`**, while the released `serverpod` on pub.dev is
[3.4.13](https://pub.dev/packages/serverpod) (2026-08-28) - so this requires a release candidate.

Readiness, verbatim from the README: "This package is still in development and is not yet ready for
production use. Although the package is functional and feature-complete, it will still pass through
some refactors and breaking changes for a better integration on real Serverpod projects."

Conflict model: delta-CRDT with **field-level merges ordered by hybrid logical clocks**, and "a
monotone causal-length tombstone governs row existence, so add/delete/restore can never oscillate".
It claims to honour foreign-key `onDelete` actions and unique constraints after every merge - the
strongest relational-integrity claim of anything surveyed.

Costs of adoption here: it is bound to **Serverpod's own model layer and code generator, not
Drift** - schema is declared in Serverpod model files with `database: sync`. Documented, permanent
constraints: "Unique indexes must include `scopeId` together with the target columns"; "All 1:1
relations must have the foreign-key column nullable (`optional` relation)"; "Non-nullable
foreign-key relations must be declared as `deferred`" - and "Most are fundamental to the design and
can never be lifted." Flutter web support is **not addressed in the README**, and pub.dev assigns
these packages only `platform:windows`, which suggests platform detection failed or dependencies
block the mobile/web targets. **Unverified whether it runs on Flutter web or mobile at all.**

---

## 15. Hand-rolling a change-log protocol over a plain Dart backend

No product to survey, so this section records only what the ecosystem supplies and what the design
would have to contain.

**Server-side Dart options** (pub.dev API, 2026-09-11):

| Package | Version | Published | Publisher |
|---|---|---|---|
| [`shelf`](https://pub.dev/packages/shelf) | 1.4.2 | 2024-06-21 | `tools.dart.dev` (Dart team) |
| [`dart_frog`](https://pub.dev/packages/dart_frog) | 1.2.6 | 2025-11-03 | `dart-frog.dev` |
| [`serverpod`](https://pub.dev/packages/serverpod) | 3.4.13 | 2026-08-28 | `serverpod.dev` |

`shelf` is maintained by the Dart team and is a minimal HTTP middleware layer; its low release
cadence reflects stability, not abandonment. `dart_frog` builds on it. `serverpod` is a much larger
framework with its own ORM and code generation.

**What you inherit rather than avoid.** The map already accepts
([map.md](../map.md), "Standing consequences") that change tracking, tombstones, globally-unique
ids and a conflict rule are unavoidable under sync. Hand-rolling means owning all of them -
concretely: globally-unique ids generated offline (UUIDv4 or v7); a per-row version or HLC; soft
deletes with tombstone retention; an append-only change log or per-table `updated_at` cursor; a
per-device high-water mark; idempotent apply; and a decision about whether the merge is row-level or
per-field. Those are the same obligations §3 and §9 impose, minus the library. See
[07-persistence-model-under-sync](../issues/07-persistence-model-under-sync.md) and
[08-the-conflict-rule](../issues/08-the-conflict-rule.md).

**What it buys.** Nothing to rot: no vendor pricing page, no FSL clause, no pre-1.0 warning, no
community binding that skews against its own dependency. Drift stays exactly what it is on all
three platforms, because nothing sits between Drift and SQLite. Drift's own web support is stable
today (§2), and every other option in this survey either replaces or wraps that layer.

**What it costs.** Everything above is code you write and test, including the parts that are easy to
get subtly wrong: clock skew between devices, resurrection of deleted rows, partial-sync recovery
after a crash mid-apply, and schema migration on three devices that are not all upgraded at once.

---

## 16. Explicitly unverified

Recorded so a later session does not mistake these for established facts.

- PowerSync's funding and investors; whether **Raw Tables** are Drift-compatible.
- Electric's pre-acquisition funding, the Databricks deal terms, and whether Electric Cloud still
  accepts new signups. Its pricing page appears stale.
- Whether `libsql_dart` actually functions on Flutter web (its pub.dev `platform:web` tag is
  contradicted by its own Rust-FFI dependencies and by `drift_libsql`).
- How schema changes behave under Turso Sync.
- Whether a cr-sqlite Dart binding is feasible via static linking - nobody has published one.
- The precise conflict granularity (row vs field) of `sql_crdt` - the README does not state it.
- Body text of Supabase's partner pages; whether Supabase Realtime replays messages missed during a
  disconnect.
- PocketBase record `id` format, and whether any built-in versioning or soft-delete exists.
- Current open/closed status of the cited FlutterFire web-persistence issues.
- Whether `serverpod_offline_sync` runs on Flutter web or mobile at all.
- Ditto's funding (secondary press only, no primary announcement) and the actual dollar price of any
  Ditto paid tier; also the commercial terms for self-hosting Big Peer.
- Whether `yjs_dart`'s claimed Yjs binary compatibility and web support actually hold - the package
  has one GitHub star and shows no sign of having been exercised.
- Whether any candidate's auth flow survives cross-origin isolation (COOP/COEP), which
  [04](../issues/04-drift-sqlite-wasm-viable.md) makes mandatory for the web build.
