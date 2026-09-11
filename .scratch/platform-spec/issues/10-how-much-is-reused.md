# How much of nooka and habbits is reused?

Type: grilling
Status: open
Blocked by: 07

## Question

Both existing apps are Flutter + Drift + Riverpod with the same layer layout
(`lib/{ui,data,domain,l10n}`), carefully modelled domains, and ADRs explaining the hard parts.
Reuse is the main reason this is tractable at all. But
[The persistence model under sync](07-persistence-model-under-sync.md) changes the storage layer
underneath everything, which changes what is worth reusing.

Decide:

- **Copy, extract, or rewrite?** Fork the files into the new repo and edit; extract shared Dart
  packages that all three repos depend on; or re-derive from the ADRs and CONTEXT docs and write
  fresh.
- If **extract**: does that mean a monorepo, or published packages, and does it force changes
  back into two shipped apps that explicitly do not want them? (It probably does, and that alone
  may kill it.)
- Which layers are actually reusable given the storage change - the domain rules (streak,
  completion percent, dormancy, recurrence) look portable; the DAOs look doomed.
- Are the **CONTEXT.md glossaries** merged, or does the new product get its own? Two glossaries
  that define "task" and "completion" slightly differently is a live hazard.
- What about the ADRs - are they inherited, superseded, or re-litigated?

The bias should be toward reusing *thinking* freely and *code* only where the storage change
does not invalidate it.
