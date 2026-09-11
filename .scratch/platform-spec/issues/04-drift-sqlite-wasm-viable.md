# Is Drift on sqlite-wasm production-viable?

Type: research
Status: open
Blocked by: -

## Question

The decision to ship a Flutter web client *and* stay local-first rests on Drift running against
sqlite-wasm with OPFS in the browser. This is the least-travelled part of the route and it is
load-bearing: if it does not hold, either the web client or local-first has to give.

Establish, with sources:

- The current state of Drift's web support - which storage backends it offers, which are
  recommended, and which are deprecated
- **OPFS browser support in practice**: Chrome, Safari (desktop and iOS), Firefox. Which need
  workers, which need cross-origin isolation headers, and what the fallbacks degrade to
- Behaviour with **multiple tabs open** - locking, corruption risk, and whether a shared worker
  is required
- Whether **iOS Safari** specifically is usable, including any storage eviction rules that could
  delete the database out from under the user
- Payload size of the wasm bundle and its effect on load time
- Migration story: do Drift schema migrations work the same way on web?
- Real reports of people shipping this - issues, blog posts, known sharp edges

If the answer is "it works but only under conditions X, Y, Z", state the conditions precisely.
If it is "no", say so plainly - that is a more useful answer than a hedge, and it sends
[Which sync engine?](06-which-sync-engine.md) and the client strategy back to the drawing board.
