# Map: Zarya platform spec

## Destination

A written product + architecture spec for a new, greenfield **local-first, multi-device-synced**
productivity platform under `quotidianlabs` - habits, tasks, and a pomodoro timer - with the
**focus session as the spine** and a **unified timeline as the surface**. The map is done when
nothing is left to decide before implementation sessions can start building.

`zarya` is a **working name**. See [Final product name](issues/05-final-product-name.md).

## Notes

**Domain**: personal productivity. Habits, tasks, focus sessions, and one timeline that joins them.

**Skills every session should consult**: `grilling` and `domain-modeling` by default. `research`
for the AFK tickets. `prototype` where the question is "how should this look or behave".

**Bar**: a personal tool that happens to be published. Not a commercial product. No support
obligation, no on-call, no migration guarantees to strangers. When a ticket's answer hinges on
"but what if we sell this", the answer is: we don't, and that's a scoping question for a later
effort, not this map.

**Prior art in this org, to reuse and to respect**:

- `../nooka` - shipped Flutter to-do list, v1.4.0+8. Drift/SQLite, Riverpod, EN/RU. Its
  `CONTEXT.md` and nine ADRs are the best available thinking on the task domain.
- `../habbits` - shipped Flutter habit tracker, v1.0.1+2. Same stack. Eight ADRs.
- Both ship to RuStore; `habbits` ADR 0009 rules out Google Play.
- Both are **untouched by this effort**. They keep shipping, local-first and serverless.

### Settled while charting

These shape every ticket and are not up for re-litigation without redrawing the destination.

- **Greenfield.** Not a rewrite of `nooka`/`habbits`, not a migration of their users. Heavy code
  and model reuse, zero user-facing continuity.
- **The focus session is the spine.** You start a session *on* a habit or *on* a task. The
  session is the reason these pieces belong in one product.
- **The timeline is the surface.** One unified today-view where all types land.
- **Federated model, not unified.** Habit and Task stay separate entities with separate tables.
  The Session carries a polymorphic reference to either. There is no `Doable` supertype - that
  road ends in a bag of nullable columns, which is exactly what `nooka`'s model avoids.
- **The backend has one job: syncing one user's data across their own devices.** Not
  collaboration, not monetization, not accounts-as-a-product.
- **Local-first survives.** SQLite stays the source of truth on each device; the server is a
  replication mechanism. The app must work fully offline.
- **Flutter for all three clients**, web included. Android ships first.

### Standing consequences already accepted

- `nooka` ADR 0001 **"Drift rows are the domain model" does not survive sync.** Change tracking,
  tombstones, globally-unique ids and a conflict rule are unavoidable. See
  [The persistence model under sync](issues/07-persistence-model-under-sync.md).
- Flutter web + local-first means **Drift on sqlite-wasm with OPFS** - real, but the
  least-travelled part of this whole route. See
  [Is Drift on sqlite-wasm production-viable?](issues/04-drift-sqlite-wasm-viable.md).

## Decisions so far

<!-- one line per closed ticket: gist + link. -->

- [Survey of local-first sync options for Flutter](issues/03-sync-engine-survey.md): The field is
  thin. ElectricSQL pivoted away from write-path sync and was acquired by Databricks in Aug 2026;
  Turso tells you not to use libSQL for sync; cr-sqlite is stalled with no Dart binding. Only
  **PowerSync** has a first-party GA Flutter SDK with Drift support and web, and its server is
  source-available-with-non-compete, not open source. **Hand-rolling over a plain backend is a
  serious candidate, not a fallback.**
- [Is Drift on sqlite-wasm production-viable?](issues/04-drift-sqlite-wasm-viable.md): Yes, under
  seven conditions - the binding one is that the web app must be served **cross-origin isolated**
  (COOP/COEP), which constrains hosting. iOS Safari evicts the database after seven days of
  disuse, which makes the sync layer load-bearing for correctness on web, not just convenience.

## Not yet specified

In scope, but not yet sharp enough to ticket. Graduates as the frontier advances.

- **Reminders and notifications under sync.** Both existing apps fire local notifications from
  device state. With three devices synced, who notifies, and what stops a habit reminder firing
  three times? Blocked behind the sync model.
- **Conflict UX.** Whatever the conflict *rule* turns out to be, there is a separate question of
  what, if anything, the user is shown when one happens.
- **The pomodoro's own behaviour.** Break lengths, long-break cadence, what a partial or
  abandoned session records, whether a completed session can itself complete a habit. Downstream
  of the Session model.
- **Export, backup, and getting data out.** `nooka` has manual Google Drive backup; `habbits` has
  none. A synced product changes what backup even means.
- **Migrating my own existing nooka and habbits data in.** Not a user migration - a one-off
  personal import. Shape depends on the persistence model.
- **Distribution.** RuStore, App Store, and where a Flutter web build is hosted. Whether an
  Apple developer account is worth it for a personal tool.
- **Localisation scope.** Both existing apps are EN/RU. Presumably carried over, but the ARB
  layout and whether the brand name is ever localised are open.
- **Review and analytics surfaces.** Streaks, completion percent, session history. Both existing
  apps have carefully-designed derived metrics; how they combine across types is unexplored.

## Out of scope

Ruled beyond this destination. Does not graduate; returns only as a fresh effort.

- **Mindmaps.** Shares almost nothing with the session spine - spatial graph, own editor, own
  gestures, own conflict problems, and roughly the build cost of the other three combined.
  Cut from this map, not cut forever.
- **Collaboration.** Shared lists, shared habits, anything multi-user. Multiplies the domain
  model and the conflict model by an order of magnitude.
- **Monetization.** Subscriptions, licensing, payments, IAP.
- **Absorbing `nooka` and `habbits`.** Their users, their stores listings, their data. They keep
  shipping unchanged.
- **Commercial-grade compliance.** GDPR / 152-ФЗ operational programs, data residency, DPAs.
  Flows from the personal-tool bar; revisit only if the bar moves.
