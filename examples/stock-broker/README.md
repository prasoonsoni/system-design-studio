# Example: Stock Broker Trading Platform

A complete high-level design of a retail stock-broking platform, generated with the System Design Studio skill.

The idea the whole design rests on: **the broker is not the exchange.** Orders are matched in the exchange's matching engine. The broker handles everything around that: identity, funds, risk checks, order routing over FIX, market-data distribution, portfolio and settlement.

| File | What it is |
|---|---|
| [`system-design-guide.html`](system-design-guide.html) | The full 13-section guide, with light and dark mode |
| [`hld-canvas.html`](hld-canvas.html) | Interactive architecture canvas: hover to trace connections, pick a flow to highlight its path, click a node for details |
| [`hld-explorer.html`](hld-explorer.html) | Layered architecture explorer with a step-through order lifecycle |

Download this folder and open the files in a browser. GitHub shows HTML as source code, and the pages need an internet connection for their fonts and diagrams.

## Key decisions in this design

- **Two planes, split at the edge.** The order path (order management, risk, funds, exchange gateway) is strongly consistent. The market-data path (feed ingest, cache, WebSocket fan-out) favours availability.
- **An order state machine with write-ahead persistence**, so a crash never loses or double-sends an order.
- **Money as a double-entry ledger**, not a balance field.
- **Asynchronous order placement**: the API returns `202 Accepted`, and the final state arrives over WebSocket.
- **"Exactly-once" in practice** means idempotency keys plus end-of-day reconciliation with the exchange.
- **Kafka as the event backbone**, partitioned by user for orders and funds, and by instrument for market ticks.
