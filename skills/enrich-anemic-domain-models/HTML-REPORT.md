# HTML Report Format

The anemic-domain review is rendered as one self-contained HTML file in the OS temp directory. Do not place the report in the repository.

Use Tailwind via CDN for layout and Mermaid via CDN for diagrams. Mermaid works for call graphs and state transitions. Hand-built boxes or inline SVG work better for showing behavior moving into a rich model.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Anemic domain review — {{domain/context}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      .leak { stroke: #dc2626; stroke-width: 2px; }
      .rich { background: linear-gradient(135deg, #0f172a, #1e293b); color: white; }
      .rule { border-color: #dc2626; }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-6xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## Header

Include:

- domain/context name
- date
- repository name if known
- compact legend: red = leaked behavior, dark = rich model, pale boxes = callers/services, dashed = optional side effect/event

No long introduction. Go straight to candidates.

## Candidate card

Each candidate is one `<article>` with:

- **Title** — short domain-language name, e.g. `Move Order cancellation into Order`
- **Badge row** — recommendation strength and behavior type (`invariant`, `state transition`, `calculation`, `collection rule`, `creation rule`, `domain service`)
- **Files** — monospaced list
- **Before / After diagram** — centerpiece, side by side
- **Anemia signal** — one sentence
- **Leaked behavior** — one sentence or short bullets
- **Refactor direction** — one sentence
- **Benefits** — bullets using locality, discoverability, invariant protection, and test surface

Keep prose short. If the candidate needs a paragraph to explain, redraw the diagram.

## Diagram patterns

### Behavior leakage map

Use when rules are scattered across services and handlers.

```html
<pre class="mermaid">
flowchart LR
  Handler[CancelOrderHandler] --> Rule1{checks shipped?}
  Controller[OrderController] --> Rule2{checks shipped?}
  Job[AutoCancelJob] --> Rule3{checks shipped?}
  Rule1 --> Order[(Order data)]
  Rule2 --> Order
  Rule3 --> Order
  classDef leak fill:#fee2e2,stroke:#dc2626,color:#7f1d1d;
  class Rule1,Rule2,Rule3 leak;
</pre>
```

After diagram: callers point to `order.cancel(reason)` and rule sits inside `Order`.

### State transition diagram

Use when callers manually set status fields.

Before: show callers writing states directly.
After: show named transition methods: `submit`, `approve`, `cancel`, `expire`.

### Collection encapsulation diagram

Use when callers mutate aggregate children through getters.

Before: callers add/remove items from exposed list.
After: callers use `addItem`, `removeItem`, or other domain-language methods.

### Creation rule diagram

Use when public setters or constructors allow invalid domain objects.

Before: object built in several steps with missing invariant windows.
After: factory or constructor creates a valid object in one step.

### Domain service diagram

Use when the right refactor is not an entity method.

Before: application service mixes orchestration with a cross-aggregate rule.
After: application service delegates to a named domain service, then persists.

## Recommendation strength

Use these labels only:

- **Strong** — duplicated rules, invalid states, exposed collections, or dangerous state transitions.
- **Worth exploring** — clear behavior owner, but limited duplication or lower risk.
- **Speculative** — possible improvement, but the domain may be too simple to justify change.

## Top recommendation

End with one card:

- candidate name
- one sentence explaining why it comes first
- anchor link to that candidate

Then ask the user which candidate to explore first. Do not suggest code edits in the report step.

## Tone

Use plain English and domain-language names. Be concise.

Use exactly: domain model, anemic domain model, rich domain model, invariant, behavior leakage, application service, domain service, aggregate, entity, value object, domain event.

Avoid vague labels: manager, helper, utility, logic layer, business layer, clean up, better structure.
