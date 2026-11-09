# Example: System Design of Uber

A complete high-level design of an Uber-scale ride-hailing platform, generated with the System Design Studio skill. It shows a **product-themed** guide (monochrome with one green accent, a pickup → route → drop-off motif) and the **single-file** delivery: the guide and the interactive canvas live in one HTML page with tabs.

The idea the whole design rests on: **driver positions are a disposable, high-volume stream; the trip and the money are the only strongly consistent truth.** Dispatch is geospatial matching over positions that are stale the moment they arrive, and a driver can hold at most one live offer, enforced by a lease.

| File | What it is |
|---|---|
| [`system-design-guide.html`](system-design-guide.html) | One page with two tabs: the full 13-section guide and the interactive architecture canvas, in light and dark mode |

## Run it

GitHub shows HTML as source, so clone the repo (or download this folder) and serve it locally:

```bash
cd examples/uber
python3 -m http.server 8765
```

Then open <http://localhost:8765/system-design-guide.html>. Add `#canvas` to the address to open the canvas tab directly. Opening the file straight from disk also works. An internet connection is needed for the fonts and the data-model diagram.

## Key decisions in this design

- **Two planes.** Locations, the supply index, ETA and surge favour availability and live in memory, sharded by H3 cell. The trip state machine, the driver lease and the ledger are strongly consistent.
- **One offer per driver.** Dispatch takes a 15-second lease on the driver with compare-and-set before sending an offer; the trip's version check is the backstop.
- **Asynchronous request.** `POST /v1/trips` returns `202 Accepted` once the trip is durable; the match arrives over WebSocket.
- **Money as a durable workflow** (Cadence) into a double-entry ledger, with idempotency keys and daily reconciliation, never "exactly once".
- **Capacity anchored on public numbers**: 13.57B trips in FY2025, giving ~375k location pings/s and ~4.6M open sockets at peak, with the arithmetic shown.

## What the canvas does

- Hover a component to trace its connections; click it for what it does, how it works, the data it owns, scale and failure, and the events it emits and consumes.
- Pick a flow (Get a fare quote, Request a ride, Driver accepts, Driver location ping, Complete & pay) to light up its path and play it hop by hop.
- Drag to pan, scroll or pinch to zoom, and use **Fit** to see everything.

Independent educational study. Not affiliated with or endorsed by Uber. Components credited to Uber (H3, Ringpop, Docstore, Cadence, DeepETA, Jaeger, M3) come from its public engineering material, cited in the guide's footer; everything else is a stated assumption.
