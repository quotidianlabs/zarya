# Zarya

A local-first, multi-device-synced productivity platform for iOS, Android and the web, in English
and Russian. It holds habits, tasks and a focus timer, joined by the focus session and surfaced as
one timeline.

## Language

A term is listed only when there is a synonym to reject, or a meaning subtle enough that code
and docs must agree on it. General programming vocabulary does not belong here.

**Focus session**:
An immutable record of one sitting spent focusing on one target. It is the spine of the product:
the reason habits, tasks and a timer belong in one app. The type is `FocusSession`, never
`Session`, because this product has a backend and "session" there means something else.
_Avoid_: session, for the entity. "Session" alone is fine in UI copy.

**Sitting**:
One continuous stretch of focus, start to stop, however many timer intervals and breaks it
contains internally. One sitting is one focus session, so a two-hour afternoon on one task is a
single record rather than four.

**Target**:
What a focus session is on: a habit, a task, or nothing in particular. It is one of exactly
three things and is never absent, so there is no null to interpret and no session with an unknown
subject.

**Freeform**:
The target of a focus session that is deliberately on nothing. It is a first-class kind of
target, not a missing one, and it is how "just focus for a while" is recorded.

**Focused time**:
The time a focus session actually spent focusing, which is not the time between its start and its
end: pauses and breaks fall in the gap. Both are recorded, and every figure derived from sessions
is built from focused time.

**Note**:
Optional free text on a focus session, written while it runs and frozen when it ends. It is what
the timeline shows for a freeform session and optional colour for any other.

**Local date**:
A calendar day in a wall-clock zone, with no time of day, and the unit the timeline groups by.
A focus session is filed under the local date it *started*, derived from its start instant and
the UTC offset in force at that moment, so crossing midnight never splits a sitting and
travelling never re-files history.

**Sync**:
Replicating one user's data across that user's own devices. It never means recomputing
notifications, which is what the word means in `habbits` and does not mean here.
