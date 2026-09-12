# Focus sessions are immutable, side-effect-free history

**Decision:** A focus session is written once, when the sitting stops, and is never updated
afterwards. It can be created and it can be deleted; no field of it can be edited. Finishing one
changes nothing else in the database: it does not check off its habit, does not advance its task,
and does not touch any other row.

The obvious alternative is the one every timer app reaches for: write a row when the sitting
starts, update it as the timer runs, and let finishing a session complete the habit it was on.
That gives a running timer that is visible on every device, lets a mistake be corrected in place,
and makes the habit check-off automatic. It was rejected because this product syncs across
devices and every one of those conveniences is a merge rule someone has to write and a class of
conflict someone has to resolve. A row that is only ever created or tombstoned converges without
a rule, on any engine, including a hand-rolled one. Sessions are the highest-volume entity in the
product and the one whose history matters most; making them the part of the model that cannot
conflict is worth more than making them editable.

Side-effect freedom is the same argument aimed at a different target. Completing a habit and
completing a task both already have real semantics inherited from `habbits` and `nooka`: at most
one completion per habit per local date, and a completed recurring task going dormant rather than
archived. Firing those from session-end would give each of them a second write path that has to
agree with the first, under sync, offline, on three devices. The prompt after a sitting ends
instead goes through the ordinary completion path, so there is exactly one way each of those
things happens.

Four consequences follow and are deliberate. **A sitting in progress does not exist on your other
devices**, because the in-flight timer is device-local state outside the synced tables; start on
the phone and the laptop shows nothing until you stop. **A forgotten stop is fixed by deleting
and re-adding**, not by editing the duration, which is why manual retroactive entry is a feature
rather than a nicety. **A freeform session cannot be retro-attached to a task** once written, for
the same reason. And **the number of pomodoro intervals in a sitting is not recorded**, so
"pomodoros completed today" is not a metric this model can produce; focused time is the unit
everything derives from.

**Revisit trigger:** wanting a running timer that follows you between devices, or wanting
per-interval records. The first is the expensive one: it makes sessions mutable and hands the
conflict rule a live-state problem, and it is the change this decision exists to avoid.
