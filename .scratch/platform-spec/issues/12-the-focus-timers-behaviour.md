# The focus timer's behaviour

Type: prototype
Status: open
Blocked by: 09

## Question

[What is a Session?](01-what-is-a-session.md) deliberately pushed the timer's interval cadence out
of the model: one sitting is one row, and the focus/break rhythm inside it is behaviour, invisible
to storage. That makes this ticket cheap to change but it does not make it decided, and it is the
screen the user stares at more than any other.

Parts of the old fog patch are already answered and are **not** in scope here: a finished sitting
never completes its habit or task (it prompts), there is no abandoned/completed outcome, and
anything under 60 seconds is not recorded.

What is left:

- **Intervals.** Focus and break lengths, long-break cadence, and whether any of it is
  configurable at all or is one opinionated rhythm. A personal tool can get away with the latter.
- **Do breaks end the sitting?** A sitting accumulates *focused* time, so a break is a gap inside
  one session rather than the end of one. Confirm that holds when the break is long, and decide
  whether there is a length past which the sitting should just stop itself.
- **What the running screen shows**, and whether it is a screen at all or a persistent bar on the
  timeline. This is the prototype's main job.
- **Backgrounding and lock screen.** The timer has to keep time with the app closed, which on both
  target platforms means a notification or live activity. What does it show, and can you pause or
  stop from there?
- **Where the note is typed.** Settled that it is written while the session runs and frozen at
  stop, so the running surface needs somewhere to put it.
- **Manual retroactive entry.** Committed to as a feature by ticket 01, and it needs a surface:
  pick a target, a start, and a duration. Where does it live?

Blocked by [What the timeline shows](09-what-the-timeline-shows.md) because the timer's entry
point is a timeline question and the running surface has to sit alongside whatever that prototype
lands on.

Build it rough. Do not build it into a real app.
