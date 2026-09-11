# Where the backend runs

Type: grilling
Status: open
Blocked by: 06

## Question

Once [Which sync engine?](06-which-sync-engine.md) is settled, the server side becomes concrete.

- **Where does it physically run?** A VPS, a managed platform, a box at home, or a vendor's
  cloud. For a personal tool serving one user, a $5 VPS and a managed Postgres are very different
  propositions from a vendor free tier.
- **What is the actual monthly cost**, and what happens when a free tier changes?
- **Who operates it?** Backups, certificate renewal, upgrades, and what happens when it falls
  over while you are on holiday. Local-first means an outage is survivable - confirm that is
  actually true of the chosen design, because it is the main argument for the whole architecture.
- **Where is it hosted geographically**, given RU is a target locale and latency and
  reachability both matter?
- **Is there a Dart backend at all**, or does the sync engine's server suffice? Writing one in
  Dart shares types and tooling with the clients; not writing one is less to run.
- **How is the Flutter web build served**, and is it the same box?

Cost and operational burden are the deciding axes here, not throughput. There is one user.
