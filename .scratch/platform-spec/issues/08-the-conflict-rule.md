# The conflict rule

Type: grilling
Status: open
Blocked by: 06, 01

## Question

Two devices edit the same thing offline. What happens?

The engine chosen in [Which sync engine?](06-which-sync-engine.md) will impose or offer a default,
but the *product* answer differs per entity and has to be decided deliberately:

- A **task's** title edited on two devices - last-write-wins is probably fine
- A **task completed** on one device and **deleted** on the other - completion and deletion are
  not commutative, and `nooka` is explicit that deletion never reaches the Archive
- A **habit checked off** for the same local date on two devices - `habbits` guarantees at most
  one completion per (habit, date), so this must idempotently converge, not duplicate
- A **recurring task** completed on both devices - does it go dormant once or twice, and which
  next-due instant wins?
- **Reordering** on two devices simultaneously
- **Sessions**, which are append-only history and should never conflict at all - confirm that

The output is a per-entity rule, not a single global policy. Where a rule loses data, say so
explicitly and decide whether that is acceptable at the personal-tool bar.
