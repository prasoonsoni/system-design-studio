---
name: system-design-studio
description: Create a senior-level system design (HLD) guide for any product or company: requirements, capacity, architecture & ER diagrams, schema, APIs and trade-offs, in light and dark themes.
---

# System Design Studio

Use when the user asks to design, explain, or learn the high-level design (HLD) of a system: "design Uber", "system design of Netflix", "HLD for a payment gateway", "design a URL shortener", "interview prep for designing X". The deliverable is one self-contained, navigable HTML design-doc published as an artifact, with light and dark modes, optionally with an interactive canvas companion. When the user wants a file to run locally instead, deliver the guide and canvas as one HTML page with tabs (§9).

Reference examples in this repository show the quality bar: `examples/stock-broker/` (default theme, guide and canvas as separate pages) and `examples/uber/` (product theme, hand-drawn architecture SVG, guide and canvas in one file).

## 1. Decide the subject and the theme

First classify the request. This decides both the content angle and the look.

- **Named product or company** (Uber, Netflix, WhatsApp, Swiggy, Zerodha, Airbnb, Spotify, Stripe, Instagram, ...): the guide is "System design of <Company>" and is **themed after that product's world** (see §5.2). Use the company's own domain vocabulary everywhere (Uber: riders, drivers, trips, dispatch, surge, ETA; Netflix: titles, playback, CDN, encoding ladder; WhatsApp: chats, delivery receipts, last seen).
- **Generic system** (URL shortener, chat app, payment gateway, rate limiter, notification service, "a ride-sharing app" with no company named) or an obscure product you cannot place: use the **default template theme** (§5.1).
- **The user names a style** ("make it look like Stripe docs", "plain and minimal"): follow that over both rules.

For a named company, research before designing: run web searches for its public engineering blog posts and talks (e.g. Uber: H3 geospatial index, Ringpop, Schemaless, Cadence; Netflix: Open Connect, EVCache, Zuul). Use what is public, mark it as such ("per Uber Engineering's blog"), list sources in the guide's footer, and label everything else as an assumption. Never present an inferred design as the company's actual internal architecture.

## 2. Brainstorm before building

Before writing any HTML, work out (and state in one short message to the user):

- **The governing insight** of the domain: the one boundary or idea that, if missed, makes the whole design wrong. Examples: "the broker routes, the exchange matches"; "a payment gateway never holds funds, the acquirer does"; "Uber's dispatch is a geospatial matching problem under a seconds-level latency budget". Put it in the overview as a callout.
- **The planes**: which paths need strong consistency (money, orders, inventory, bookings → CP) and which favour availability (feeds, search, presence, location pings, analytics → AP). Separate them at the edge.
- **The hard problems**: idempotency, ordering, fan-out, hot keys, geo-partitioning, reconciliation, exactly-once claims.
- Scope in / scope out (what is treated as an external integration).
- **The theme decision** from §1: which product theme, or the default.

If the request is truly ambiguous (which product? interview depth vs production depth?), ask one question; otherwise start.

## 3. Required sections (numbered, in this order)

1. **Overview**: what the system is, governing-insight callout, in-scope / out-of-scope lists.
2. **Requirements**: functional table (domain → capabilities, using the domain's real vocabulary and enums) and non-functional table (attribute → target → notes). Always include latency, availability, consistency (CP/AP per path), durability, scalability, security, idempotency.
3. **Capacity estimation**: table of assumption → derived number (users, DAU, peak concurrency, writes/s average AND peak, reads/s, fan-out egress, storage/day). Show the arithmetic so it can be checked; end with a takeaway callout about which axis each plane scales on. For a real company, anchor on its publicly reported scale where available and cite it.
4. **Architecture**: a rendered diagram (see §6) plus prose for each plane and the event backbone.
5. **Component deep-dives**: for each core service: responsibility, how it works mechanically (state machines, protocols, algorithms), data owned, failure handling.
6. **Core flow lifecycle**: the product's flagship end-to-end flow (Uber: request a ride → match → trip; Netflix: press play → stream; WhatsApp: send message → delivered → read) as a step table (#, step, owner, state/event), plus a callout on sync vs async boundaries.
7. **Data model & schema**: an ER/class diagram (see §6), then non-relational stores (cache key patterns, time-series layout, geo index, search index), then event topics (topic, partition key, events) with one sample event JSON. Do NOT dump raw SQL DDL as the main schema view.
8. **API design**: conventions table (auth, idempotency, error envelope, rate limits, pagination, versioning), endpoint table with coloured method pills, one worked request/response pair for the key write (showing status code semantics, e.g. 202 for async), status-code table, and the streaming protocol (WebSocket/SSE/gRPC stream) if any.
9. **Consistency & reliability**: where each guarantee applies, sagas + compensations, idempotency layers, failure-scenario table.
10. **Scaling & performance**: sharding keys, stateless autoscaling, partitioning, caching, spike handling.
11. **Security & compliance**: authn/z, data protection, regulatory retention, abuse protection.
12. **Observability**: golden metrics, tracing, reconciliation dashboards, alerts.
13. **Trade-offs**: decision → choice & why table, ending with an "if you remember five things" callout.

## 4. Accuracy bar (check before publishing)

- Every number in capacity is derived from stated assumptions and is internally consistent (average vs peak multiplier stated).
- Every component in the architecture diagram appears in the deep-dives or flow, and vice versa.
- Every entity in the ER diagram is used by some flow or API; every API's write path touches a real entity.
- Kafka/queue topics in the schema section match the edge labels in the architecture diagram.
- Money is modelled as a double-entry ledger where money exists; idempotency keys appear in the schema (unique constraint) AND the API (header).
- Use the domain's real terminology (order types, protocols like FIX/ISO 8583, regulations) and say when something is an assumption.
- For a named company: every claim about its real stack is sourced; everything else is marked as assumed.
- Never claim true exactly-once; describe idempotency + reconciliation.

## 5. Theming, light mode and dark mode

Every guide ships with **both light and dark modes**: it follows the reader's system setting by default, and a toggle in the header overrides it (remembered per browser). Load the `artifact-design` skill before writing the page.

### 5.0 Token system (both modes)

All colours are CSS custom properties. Never hard-code a colour in a rule, an SVG, or a diagram; always use a token. Define light values on `:root`, then dark values twice (system preference unless the reader chose light, and the explicit dark choice):

```css
:root{ color-scheme:light; /* light tokens */ }
@media (prefers-color-scheme: dark){
  :root:not([data-theme="light"]){ color-scheme:dark; /* dark tokens */ }
}
:root[data-theme="dark"]{ color-scheme:dark; /* dark tokens, repeated */ }
body{ background:var(--bg); color:var(--ink); }
```

Required token roles (each theme sets all of them, in both modes). Use plain hex values (no `rgba(...)`), because the diagram renderer splits on commas:

| Token | Role |
|---|---|
| `--bg`, `--surface`, `--surface-2`, `--line`, `--line-soft` | page, cards, raised cards, borders |
| `--ink`, `--ink-mid`, `--ink-dim` | headings/strong text, body, captions |
| `--accent`, `--accent-ink` | the theme's signature colour, and text drawn on it |
| `--svc`, `--svc-bg` | services in diagrams |
| `--bus`, `--bus-bg` | queue / event bus |
| `--ext`, `--ext-bg` | external systems |
| `--store`, `--store-bg` | data stores |
| `--ok`, `--warn`, `--danger`, `--info` | callouts, method pills, CP/AP pills |
| `--code-bg`, `--code-ink`, `--code-key`, `--code-str`, `--code-num`, `--code-com` | code blocks |
| `--font-display`, `--font-body`, `--font-mono` | type families |

Contrast: body text (`--ink-mid` on `--bg` and `--surface`) at least 4.5:1 in both modes; accent text on `--surface` at least 4.5:1. A bright accent that works on dark usually needs a darker shade for light mode, so pick each mode's values separately rather than reusing them.

Put this in `<head>` before the stylesheet, so a saved choice applies without a flash:

```html
<script>
(function(){var t=null;try{t=localStorage.getItem('sdg-theme')}catch(e){}
if(t==='light'||t==='dark')document.documentElement.dataset.theme=t;})();
</script>
```

Toggle button in the header (`<button class="theme-toggle" id="themeToggle" aria-label="Switch colour theme"></button>`) and its script near the end of `<body>`:

```html
<script>
(function(){
  var root=document.documentElement, mq=matchMedia('(prefers-color-scheme: dark)');
  function current(){return root.dataset.theme||(mq.matches?'dark':'light');}
  function label(){var b=document.getElementById('themeToggle');if(b)b.textContent=current()==='dark'?'☀ Light':'☾ Dark';}
  document.getElementById('themeToggle').addEventListener('click',function(){
    var next=current()==='dark'?'light':'dark';root.dataset.theme=next;
    try{localStorage.setItem('sdg-theme',next)}catch(e){}
    label();document.dispatchEvent(new Event('themechange'));});
  mq.addEventListener('change',function(){if(!root.dataset.theme){label();document.dispatchEvent(new Event('themechange'));}});
  label();
})();
</script>
```

### 5.1 Default template theme (generic systems)

The house style for anything not tied to a named product:

- Type: Space Grotesk (display), Inter (body), JetBrains Mono (identifiers, enums, topics, paths).
- Dark: midnight base. `--bg #0a0e17`, `--surface #141b2c`, `--surface-2 #1a2336`, `--line #28324e`, `--ink #eaf0fb`, `--ink-mid #aab6d2`, `--ink-dim #7a86a4`, `--accent #5ec7df`, `--svc #4fd0e0`/`--svc-bg #152036`, `--bus #f3ab54`/`--bus-bg #241d12`, `--ext #fb7f95`/`--ext-bg #2a1622`, `--store #93a3c0`/`--store-bg #131b2c`.
- Light: cool paper. `--bg #f6f8fc`, `--surface #ffffff`, `--surface-2 #eef2f8`, `--line #d6deea`, `--ink #0f1729`, `--ink-mid #45536b`, `--ink-dim #5f6c85`, `--accent #0b7287`, `--svc #0b7287`/`--svc-bg #e3f3f7`, `--bus #a35a00`/`--bus-bg #fdf0dd`, `--ext #b4234a`/`--ext-bg #fde7ec`, `--store #55627a`/`--store-bg #eef1f6`.
- Layout: hero (eyebrow, title, one-paragraph summary with the governing insight, 4 TL;DR cards: peak scale, connections, CP path, AP path), then a two-column body with a sticky table of contents and scrollspy on desktop and a floating "☰ Contents" drawer on mobile.

### 5.2 Product themes (named company)

The page should feel like it belongs to that product's world, while staying clearly an independent study. Derive a full token set (both modes) from the product's public visual character: its palette, its type mood, its shape language (sharp vs rounded), its density, and one signature motif. Starting points (approximate, evoke rather than copy):

| Product | Palette direction | Type (free substitute) | Shape & motif |
|---|---|---|---|
| Uber | Monochrome black/white, one green accent | Inter Tight, heavy display weights | Sharp corners, high contrast; map route lines and pickup/drop-off dots |
| Netflix | Near-black with a deep red accent; light mode warm white + red | Bebas Neue display, Inter body | Cinematic large headings, poster-card rows |
| WhatsApp | Teal-green on soft neutral; dark mode deep green-grey | Inter / system UI | Chat-bubble callouts, double-tick status marks |
| Airbnb | Warm white with coral accent | Nunito Sans | Generous rounding, soft shadows, friendly spacing |
| Spotify | Black with bright green; light mode white + green | Figtree or DM Sans | Pill buttons, waveform/progress-bar motif |
| Stripe | Light-first, indigo/violet accents, soft gradients | Inter | Crisp hairlines, code blocks as first-class |
| Swiggy / Zomato | Warm orange or red on cream; dark brown-black | Poppins | Rounded cards, order-tracking timeline |
| Zerodha | Clean white with blue accent; dark slate | Inter | Dense data tables, ticker strip |

For a company not in the table, apply the same method: identify its signature colour, mood and motif, then build both modes.

Guardrails for product themes:

- **Inspired by, never copied.** No logos, wordmarks, app icons, mascots, screenshots or copied UI screens, and no proprietary typefaces (Uber Move, Netflix Sans, Circular, Cereal): use the free substitutes. The motif is drawn generically (a route line, a chat bubble), not lifted from the product.
- Title it as a study: "System Design of Uber" or "Designing an Uber-scale ride-hailing platform", never as if Uber published it.
- Footer, always: "Independent educational study. Not affiliated with or endorsed by <Company>. Architecture reconstructed from public sources and stated assumptions." plus the sources list.
- Diagrams, tables, callouts and pills still use the same component structure as the default template, recoloured through the tokens. The product theme changes the look, not the 13-section structure.

### 5.3 Shared components

- Tables in rounded scroll wrappers; callouts (key / warn / info / note) with a coloured left bar; method pills (GET `--ok`, POST `--info`, PUT `--warn`, DELETE `--danger`); CP/AP pills.
- Code blocks only for JSON payloads, key patterns and protocol messages, with a header bar naming the language or endpoint and syntax colours from the `--code-*` tokens.
- Phone-width safe: 16px gutters, no horizontal page scroll (tables and figures scroll inside their own wrappers).

## 6. Diagrams (theme-aware)

Use Mermaid inside `<pre class="mermaid">` blocks, wrapped in `<figure class="fig">` with a numbered `figcap`. Do NOT put `%%{init}%%` directives or hex colours in diagram source. Colour classes with `var(--token)` references; the render script swaps in the current theme's values and re-renders whenever the theme changes.

**The page MUST load Mermaid with this script**, or diagrams render as raw text and won't follow the theme. Put it just before `</body>`, after the toggle script:

```html
<script type="module">
import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs";
const blocks=[...document.querySelectorAll('pre.mermaid')];
blocks.forEach(b=>{b.dataset.src=b.textContent;});
const v=n=>getComputedStyle(document.documentElement).getPropertyValue('--'+n).trim();
async function render(){
  mermaid.initialize({startOnLoad:false,securityLevel:'strict',theme:'base',fontFamily:v('font-body')||'Inter, sans-serif',
    themeVariables:{background:v('surface'),mainBkg:v('surface-2'),primaryColor:v('surface-2'),
      primaryBorderColor:v('accent'),primaryTextColor:v('ink'),secondaryColor:v('surface-2'),
      tertiaryColor:v('surface'),lineColor:v('ink-dim'),textColor:v('ink'),nodeBorder:v('line'),
      clusterBkg:v('surface'),clusterBorder:v('line'),edgeLabelBackground:v('surface'),classText:v('ink')}});
  for(const b of blocks){b.removeAttribute('data-processed');
    b.textContent=b.dataset.src.replace(/var\(--([a-z0-9-]+)\)/g,(_,n)=>v(n));}
  try{await mermaid.run({nodes:blocks});}catch(e){console.error(e);}
}
await render();
document.addEventListener('themechange',render);
</script>
```

**Architecture (Figure 1)**: `flowchart TB` with `subgraph`s for Edge, the CP plane, Consumers, Data Stores. Shape by type: databases `[("Postgres")]`, queue/bus `[["Kafka"]]`, external systems `{{"Exchange"}}`, services `["OMS"]`. Label edges with protocol or topic (`-->|REST|`, `<-->|FIX|`, `-->|order.executed|`). Class colours:

```
classDef svc fill:var(--svc-bg),stroke:var(--svc),color:var(--ink);
classDef bus fill:var(--bus-bg),stroke:var(--bus),color:var(--bus);
classDef ext fill:var(--ext-bg),stroke:var(--ext),color:var(--ext);
classDef store fill:var(--store-bg),stroke:var(--store),color:var(--ink);
```

Explain the colour roles in the caption by role (services / bus / external / stores), not by hue, since hues differ between themes.

**Size Mermaid output to its natural width.** By default the SVG shrinks to the column and its text becomes unreadable. After rendering, set the SVG's `width` to its own `max-width` and let the figure scroll inside an `overflow-x:auto` wrapper.

**When Mermaid tangles, hand-draw Figure 1.** A hub-and-spoke graph (Kafka in the middle, 20+ edges crossing subgraphs) defeats Mermaid's layout: subgraph `direction` is ignored once nodes link outside it, and the result comes out about 2,500px wide with crossing edges. Measure the rendered width; if it is over ~1,600px or edges cross through nodes, draw the architecture as inline SVG instead, using the same tokens (see `examples/uber/`):

- Grid layout: clients and edge gateways in a left column; the **CP plane** in a dashed frame on top; **Kafka as a horizontal bus** across the middle; the **AP plane** in a frame below; consumers, stores and external systems in a right column. Producers and consumers attach to the bus with short vertical edges, which removes most crossings.
- Style through classes, never literals: `.nd rect{fill:var(--svc-bg);stroke:var(--svc)}`, store nodes rounded with `--store`, external nodes as hexagons with `--ext`, the bus with `--bus`. Mark the one or two nodes where the planes meet (e.g. Dispatch, Supply Index) with an accent stroke.
- Edge types: sync solid `--ink-dim`, Kafka solid `--bus`, store dotted `--store`, external dashed `--ext`; one arrow marker per colour with `style="fill:var(--…)"`.
- Edge labels in the mono face at 10px with `paint-order:stroke` and a `--surface` stroke so they stay legible over lines. Keep labels short ("CAS", "capture") where space is tight.
- Use distinctive class names inside the SVG; a short class such as `.st` can collide with a page-level pill class and silently change fonts.

**Data model (Figure 2)**: `classDiagram` with `direction LR`. Each class lists key fields, one per line, with 🔑 for PK and 🔗 for FK. Relationships with cardinality and verb labels:

- composition `*--` for part-of, lifecycle-bound (Order owns Trades)
- aggregation `o--` for has-a (User has Holdings)
- association `-->` for references (Order references Instrument)

Add a notation legend under the diagram (◆ composition, ◇ aggregation, ▸ association, 1 · 0..* cardinality).

Optional: a `sequenceDiagram` for the core flow in section 6.

## 7. Optional companion: interactive HLD canvas

Offer (don't force) a second artifact when the user wants to explore interactions. It uses the same theme tokens and toggle as the guide.

- SVG node graph; each node has a tinted icon badge with a **type-accurate icon** (database cylinder for Postgres, cylinder + bolt for Redis, cylinder + clock for time-series, a graph mark for Kafka, server/router for gateways, shield for risk, building for an exchange or bank, bell for notifications). Icons are generic line icons, never company logos.
- Typed edges: sync call (solid), queue/event (solid, different hue, topic label), stream (dashed), store read/write (dotted), external protocol (distinct hue, bidirectional).
- Kafka/queues drawn as a tall central spine: producers enter on one side, consumer groups leave on the other.
- Hover a node: highlight its edges and neighbours, fade the rest. Flow chips (the product's real flows, e.g. "Request a ride", "Driver location ping") light up and animate the exact path and print the hop sequence. Click a node: side panel with What it does / How it works / Data owned / Scale & failure / Events emitted & consumed.
- Pan (drag), zoom (wheel + buttons), fit-to-screen.

Build notes (what made `examples/uber/` work):

- Keep it data-driven: a `nodes` array (id, label, sub-label, x, y, type, icon, and the five panel fields), an `edges` array (id, from, to, type, label, per-edge anchor offsets so parallel edges don't overlap), and `flows` as ordered lists of `[edgeId, reversed?, hop text, topic]`. Geometry and the panel are generated from these.
- Flow playback: a dot travels each edge with `getPointAtLength`, the matching hop in a numbered list highlights, clicking a hop replays from there, and **Replay** restarts. Under `prefers-reduced-motion`, highlight the path without animating.
- Open in a working state: the flagship flow is selected and playing. Don't auto-open the side panel on load; it covers a third of the graph.
- **Fit** reserves space for everything that floats over the canvas: the legend (left), the zoom controls (right), the open side panel, and the hop list on phones.
- Start a drag only after the pointer moves about 4px, so a click on a node still opens its panel. Support two-finger pinch.
- Scroll the hop list with its own `scrollTop`, never `scrollIntoView`, which also scrolls the page behind an overlay.
- A one-screen app grid needs `grid-template-columns:minmax(0,1fr)`, or a horizontally scrolling chip row stretches the whole page wider than the phone.

## 8. Check, publish, finish

Before publishing, render the page with Playwright in both `colorScheme: 'light'` and `'dark'`, click the toggle once in each, and look at the screenshots: text readable, diagrams drawn (not raw text) and re-coloured, nothing invisible against its background. Mermaid's CDN may be blocked from the sandbox; if so, install `mermaid@11.4.1` from npm and point the test copy's import at the local file (for example with Playwright's `page.route('**/mermaid@11.4.1/dist/**', …)`).

Also check, at 1280px and 390px:

- `document.documentElement.scrollWidth` equals `clientWidth` (no sideways scroll on phones).
- No console errors, and every diagram container holds an `<svg>`.
- For the canvas: hover a node, click a node with the panel closed, play one flow, and look at the screenshot for nodes hidden under the legend, zoom controls or hop list.

Publish with the Artifact tool, then reply with 2–4 sentences: what the guide covers, the theme it uses and why (product theme or default), the key design decisions worth noticing, and one offer for a next step (sequence diagram, deep-dive on one component, interview cheat-sheet, PDF export).

## 9. Single-file local delivery (on request)

When the user asks for one HTML file, something to "run on localhost", or the guide and canvas together, build one page with tabs instead of two artifacts. `examples/uber/system-design-guide.html` is the reference.

- Write a full document (`<!doctype html>`, `<head>` with charset, viewport, `<title>`, fonts and styles), since there is no artifact skeleton to wrap it.
- One sticky header holds the brand, two tabs (**Design guide** / **Interactive canvas**, shortened to Guide / Canvas on phones) and the single theme toggle. Both views read the same tokens, so one `themechange` event updates both.
- Route on the hash: `#canvas` shows the canvas, any other hash shows the guide. When returning to the guide on a section hash (`#data`), scroll that section into view after un-hiding it.
- **Start the canvas lazily** the first time its tab is shown (inside `requestAnimationFrame`), because `getTotalLength()` and fit need a laid-out SVG. On later visits only re-fit. Size the canvas view to `calc(100dvh - header height)` and hide page scroll while it is shown.
- **Render Mermaid only while the guide is visible.** Remember which theme it was rendered for; on `themechange` or on switching back to the guide, re-render if the theme differs. Keep diagram sources in `<script type="text/plain">` and draw them with `mermaid.render()` into plain containers, so no other Mermaid handler double-processes them.
- Namespace canvas classes that clash with the guide (`.legend`, `.pill`, `.foot` became `.cv-legend`, `.cv-pill`, `.cv-foot`), and watch element selectors from the guide (`section{padding-top:…}`) leaking onto canvas elements; reset them on the canvas element.
- Before handing it over, serve it with `python3 -m http.server` and check both tabs on localhost, the theme carrying across, and `#canvas` opening directly. Tell the user the command to run it.
