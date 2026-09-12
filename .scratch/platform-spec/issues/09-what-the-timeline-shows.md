# What the timeline shows

Type: prototype
Status: open
Blocked by: 01

## Question

The unified timeline is the surface that makes this one product rather than three. It is also
the easiest thing to get wrong on a whiteboard, because four object types in one list either
reads as a coherent day or as a mess.

Build a cheap, throwaway prototype - a rough UI, not production code - to answer:

- What does today actually look like when it holds habits due, tasks due, and sessions already
  logged? Is it one list, or sections?
- Are logged sessions shown inline in the same list as things still to do, or in a separate
  track? Past and future in one column is the hard part.
- How does a habit visually differ from a task at a glance, given they are separate models with
  different affordances (check off vs complete, streak vs recurrence)?
- Where does "start a focus session" live - on each row, or as a global action that then asks
  what to focus on? This is the spine's main entry point and it has to feel obvious.
- What does the timeline show on an empty day, and on a day with thirty items?
- Does it scroll through past and future days, or is it strictly today?

Link the prototype from this ticket. Do not build it into a real app.

**Settled by [What is a Session?](01-what-is-a-session.md), narrowing this ticket:** sessions are
strictly past-facing (there are no planned sessions), so the timeline's future half holds only
habits and tasks due. A session row carries its target's name, its focused time, and an optional
note which is the whole label for a **freeform** session, so the prototype must show what a
freeform row looks like next to a targeted one. A sitting is one row however long it ran, and
sessions may legally overlap, so the layout has to survive two simultaneous rows without looking
broken. Sessions are filed under the local date they *started*, so one crossing midnight appears
on the earlier day. The "where does start-a-session live" question is unchanged and still the most
important one here.
