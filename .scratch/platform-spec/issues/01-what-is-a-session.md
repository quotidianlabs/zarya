# What is a Session?

Type: grilling
Status: resolved
Assignee: Artur Shiriev
Blocked by: -

## Question

The focus session is the spine of the whole product - it is the reason habits, tasks and a timer
belong in one app rather than three. Before anything else can be modelled, the Session itself has
to be pinned down.

- What can a Session point at? A habit, a task, both, neither? Is a freeform session with no
  target a first-class thing or an absence?
- Can one Session reference more than one target, or is it exactly one?
- Is the polymorphic reference nullable, and if so what does a null mean - "not chosen yet",
  "deliberately unfocused", or "the target was deleted"?
- What happens to a Session when its target is deleted? Does it survive orphaned, cascade, or
  denormalise the target's name at start time?
- Does finishing a Session *do* anything to its target - complete the habit for today, advance
  the task - or is it purely a record alongside it?
- What is recorded: start and end instants, or a duration? What about a session that is paused,
  abandoned, or still running when the app is killed?
- Is a Session ever edited after the fact, or is it append-only history?

Resolving this settles the join at the centre of the model. Nearly everything else on the map
reads it.

## Answer

**A `FocusSession` is an immutable record of one sitting on one target.** It is created once, when
the sitting stops, and is never updated. The whole shape is written up in the new repo:
[CONTEXT.md](../../../CONTEXT.md) for the vocabulary and
[ADR 0001](../../../docs/adr/0001-focus-sessions-are-immutable-side-effect-free-history.md) for
the decision and its costs.

| | |
|---|---|
| **Granularity** | One sitting, start to stop. Timer cadence is behaviour, not model. |
| **Target** | `OnHabit(id) \| OnTask(id) \| Freeform`. Exactly one, never absent. |
| **Stored** | `startedAt` + UTC offset at start, `endedAt`, accumulated focused seconds, target kind + id + name snapshot, optional note. |
| **Day** | Local date derived from `startedAt` and its stored offset. Crossing midnight belongs to the start day. |
| **Written** | On stop only, above a 60s floor. |
| **Mutation** | Create and delete. No field edits. |
| **Side effects** | None. |
| **Starting** | Active tasks and any habit. Retroactive creation accepts any target. |
| **Overlap** | Allowed, unvalidated. |
| **Orphaning** | Survives target deletion: live name when the id resolves, snapshot when it does not. |

**The answer to almost every sub-question was decided by one thing: a row that is only ever
created or tombstoned converges without a merge rule.** Each convenience rejected below was
rejected because it would have added a per-entity conflict rule to
[The conflict rule](08-the-conflict-rule.md):

- **No planned sessions.** A scheduled intention is a task with a due time, which `nooka`'s model
  already expresses. Planned sessions would add a second kind of future-dated object to the
  timeline plus a `planned → running → completed → missed` state machine.
- **No synced running timer.** The in-flight sitting is device-local state, heartbeated every
  ~30s, outside the synced tables. Start on the phone and the laptop shows nothing until you
  stop. On crash recovery the app truncates to the last heartbeat and asks whether to save or
  discard; it never writes a phantom and never silently loses one.
- **No field edits.** A forgotten stop is delete-and-recreate. Manual retroactive entry is
  therefore a real feature, not a nicety, and it is a create.
- **No side effects on the target.** Completing a habit after a sitting is a UI prompt through
  the ordinary completion path, so `habbits`' at-most-one-completion-per-(habit, local date) and
  `nooka`'s dormant-vs-archived rules keep exactly one write path each.
- **No outcome field.** A sitting is however long it was; "abandoned" is just short. A 60s floor
  stops mis-taps littering the timeline.

**Costs accepted, in the open:**

1. **Pomodoro counts are not derivable.** Intervals are not recorded, so "4 pomodoros today"
   cannot be produced. Focused time is the unit every derived figure is built from. Reversing
   this means child interval rows.
2. **A freeform sitting cannot be retro-attached to a task.**
3. **A running timer does not follow you between devices.**

**Two vocabulary hazards found while resolving, both from inheriting prior-art thinking:**

- **`habbits` defines "Sync" as recomputing notifications, explicitly not moving data off
  device.** This product needs the word for the real thing, so that entry cannot be inherited.
  Fixed in the new glossary.
- **The entity is `FocusSession`, never `Session`.** [Accounts or paired devices?](02-accounts-or-paired-devices.md)
  is about to give "session" a second meaning the moment there is a server. `nooka` already
  carries this scar: its glossary rejects "to-do item" while the code still says `TodoDao`.

**Downstream:** the per-entity constraint this puts on
[The conflict rule](08-the-conflict-rule.md) (sessions confirmed conflict-free by construction,
not by hope), on [The persistence model under sync](07-persistence-model-under-sync.md) (create +
tombstone only; the in-flight record is local-only and must survive an app kill without being
synced), and on [What the timeline shows](09-what-the-timeline-shows.md) (what a session row
holds, and that the timeline is strictly past-facing for sessions).
