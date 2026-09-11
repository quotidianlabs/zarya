# Which sync engine?

Type: grilling
Status: open
Blocked by: 02, 03

## Question

Given the field established by [Survey of local-first sync options for Flutter](03-sync-engine-survey.md)
and the identity shape settled by [Accounts or paired devices?](02-accounts-or-paired-devices.md),
choose how this product syncs.

The decision has to answer:

- Which engine, library, or hand-rolled protocol
- What it costs to run, and who runs it
- What it forces on the schema, and whether that is acceptable
- What happens when the sync service is unreachable, slow, or permanently gone - a personal tool
  should not become a brick because a vendor shut down
- Whether the choice is reversible, and at what cost

The single largest failure mode here is picking a vendor-hosted engine whose free tier or
existence is not guaranteed, for a product intended to outlive any particular company's
roadmap. Weigh self-hostability accordingly.

This is the widest fork on the map. Several tickets are blocked on it.
