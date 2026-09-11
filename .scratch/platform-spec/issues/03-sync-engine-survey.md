# Survey of local-first sync options for Flutter

Type: research
Status: claimed
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
