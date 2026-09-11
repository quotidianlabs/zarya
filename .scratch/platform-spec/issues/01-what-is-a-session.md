# What is a Session?

Type: grilling
Status: open
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
