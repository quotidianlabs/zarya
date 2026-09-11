# Survey of local-first sync options for Flutter

Type: research
Status: resolved
Blocked by: -

## Question

Establish the actual, current field of ways to sync a SQLite/Drift local-first Flutter app across
iOS, Android and web, as of late 2026. Facts, not a recommendation - the choice is
[Which sync engine?](06-which-sync-engine.md).

For each viable option, report:

- What it is, who maintains it, licence, and whether it is production-ready or a research project
- **Flutter/Dart support specifically** - a real, maintained package, or a community binding
- **Whether it works with Drift**, and if so how much of the Drift model has to change
- **Web support**, given the app must run in the browser on sqlite-wasm
- Self-hostable or vendor-only, and what hosting actually costs for one user with three devices
- The conflict model it imposes (last-write-wins, per-field, CRDT, operational transform)
- Whether it brings its own auth, and whether that auth can be bypassed or replaced
- What it demands of the schema: id types, tombstones, version columns, change tables

Candidates to cover at minimum: PowerSync, ElectricSQL, Turso / libSQL embedded replicas,
cr-sqlite, Ditto, Supabase (and its Flutter offline story), PocketBase, Firebase/Firestore,
Automerge and Yjs as CRDT libraries, and hand-rolling a change-log protocol over a plain Dart
backend.

Also report honestly on anything that has died, stalled, or changed licence recently - this
ecosystem moves, and stale advice is the main risk here.

## Answer

**The field is far thinner than the candidate list assumed.** Full survey, with a 13-row
comparison table and a section per candidate:
[research/03-sync-engine-survey.md](../research/03-sync-engine-survey.md).

Four findings that invalidate most advice written before mid-2026:

1. **ElectricSQL is dead for this use case, twice over.** It pivoted to a *read-path-only*
   Postgres-to-HTTP sync engine with no write path and no client SQLite - their own docs say
   "Electric does not do write-path sync". Then on **2026-08-11 it was acquired by Databricks**
   and Electric Cloud is winding down. The domain moved to `electric.ax`; the community Dart
   client is discontinued and its repo archived since 2024-07.
2. **Turso tells you not to use libSQL for sync**, in their own 2026-04-24 benchmark post. The
   libSQL server binary has not been released since 2025-02-14; the replacement Turso Sync is
   cloud-only and the new server is closed source. Dart support is one community publisher, no web.
3. **cr-sqlite is stalled** - last release 2024-01-17, maintainer's last commit the same day, wasm
   build frozen 2023-12-16, and **no Dart binding exists at all**.
4. **PowerSync's server is FSL-1.1-ALv2** - source-available with a non-compete, converting to
   Apache-2.0 after two years. Not open source. The Dart SDK itself is Apache-2.0.

What is actually left:

- **PowerSync** is the only option with a first-party GA Flutter SDK, an official Drift
  integration (`drift_sqlite_async` 0.3.1, beta) and working web on sqlite3.wasm + OPFS (beta).
  Its price is structural: PowerSync tables are **views over a schemaless JSON store**, so the
  schema is declared twice and only `text`, `integer` and `real` exist client-side.
- **Ditto** is the only other first-party Flutter SDK, but it is proprietary, replaces Drift
  entirely, and its web store is RAM-only.
- **Automerge and Yjs** have no usable Dart binding - the pub.dev `automerge` package is an empty
  20 KB placeholder whose repo 404s - and both turn Drift into a blob store.
- **Supabase, PocketBase and Firestore provide no sync engine at all.** They are transports you
  would hand-roll a protocol against. Hand-rolling is therefore a serious candidate, not a
  fallback.

Two candidates the ticket did not name, both found and covered: `serverpod_offline_sync` (0.0.5,
published six days ago; delta-CRDT with field-level HLC merge, but bound to Serverpod's model
layer rather than Drift, and self-declared not production-ready) and `crdt_lf` (active Dart CRDT
with a Drift adapter, but Drift is used as a blob store for CRDT changes; months old, ~0 likes).

Cross-ticket: the COOP/COEP requirement and the iOS Safari seven-day eviction from
[Is Drift on sqlite-wasm production-viable?](04-drift-sqlite-wasm-viable.md) are folded into the
web baseline. The eviction matters here specifically - on web the server is the only durable
copy, which changes how the online-only and RAM-only web tiers fail.
