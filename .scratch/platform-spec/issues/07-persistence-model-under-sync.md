# The persistence model under sync

Type: grilling
Status: open
Blocked by: 06

## Question

`nooka` ADR 0001 says "Drift rows are the domain model", and both existing apps use
autoincrement integer primary keys. Neither survives multi-device sync: two devices offline will
mint the same integer id for different rows, and a naive row has nowhere to record that it was
deleted.

Settle the storage shape:

- **Primary keys**: UUIDv4, UUIDv7, ULID, or something the sync engine dictates. Note that
  UUIDv7 sorts by creation time, which may or may not matter given explicit sort orders already
  exist in `nooka`.
- **Deletes**: tombstones, or does the engine handle it? If tombstones, when are they purged, and
  what stops a purge resurrecting a row on a device that was offline longer than the retention?
- **Change tracking**: per-row version, per-field timestamps, a change log table, or engine-provided
- **What replaces ADR 0001.** If the raw row is no longer the domain model, what is? A mapped
  domain object, or a row plus a projection? This is a real cost - ADR 0002 makes the repository
  the data test-seam precisely because the row *was* the model.
- **Sort order under sync.** `nooka` uses explicit contiguous integers reassigned on reorder.
  Two devices reordering offline will collide badly. Fractional indexing?
- Whether `habbits`' `(habit, local date)` completion uniqueness survives two devices checking
  off the same day.

**Constraint surfaced by [the sync survey](03-sync-engine-survey.md):** if PowerSync wins
[Which sync engine?](06-which-sync-engine.md), much of this ticket is decided for you and not in
your favour. PowerSync tables are views over a schemaless JSON store, so the schema is declared
twice and only `text`, `integer` and `real` exist client-side. That removes column-level type
guarantees from Drift, which is a direct hit on `nooka` ADR 0001's premise. Weigh that as a cost
of the engine, not a detail to sort out afterwards.

Whatever is decided here almost certainly wants an ADR in the new repo.

**Constraint from [What is a Session?](01-what-is-a-session.md):** `FocusSession` rows are
create-and-tombstone only, never updated, so whatever change-tracking mechanism this ticket picks
has to carry no per-field versioning for them. Two further demands: the **in-flight sitting** is
device-local state that must persist across an app kill and must *never* be synced, so the storage
design needs a place for durable local-only state outside the replicated tables; and a session
stores its **UTC offset at start** alongside the start instant, which is a column the prior-art
schemas have no equivalent of.
