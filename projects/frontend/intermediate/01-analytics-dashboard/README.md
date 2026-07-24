# Dashboard with Charts (Analytics)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build an analytics dashboard that turns a stream of raw records — orders, page views, signups — into charts, KPI cards, and a filterable overview a stakeholder can actually read. The interesting part is not drawing a single chart; it is aggregating data on the client, keeping several visualizations in sync behind one shared filter, and doing it without the page stuttering when the dataset grows. You will wire a charting library to fetched data, add a date-range filter that every widget reacts to, and let users drill from a summary into detail. It is a compact tour of data fetching, derived state, and responsive, accessible visualization.

## Prerequisites

- Comfort with a component framework (React, Vue, or Svelte) and its state model
- Fetching data over HTTP and handling promises (`fetch`, `axios`, or your framework's data layer)
- Array transforms for aggregation: `map`, `filter`, `reduce`, grouping
- Basic understanding of responsive layout with CSS grid or flexbox
- A charting library of your choice (Recharts, Chart.js, or Apache ECharts)

## Learning Objectives

By the end, you should be able to:

- Aggregate raw records into series and totals with pure, testable functions
- Drive multiple charts from a single source of truth so filters stay in sync
- Render responsive charts that resize with their container without distortion
- Represent loading, empty, and error states distinctly for every widget
- Make charts accessible with text alternatives and keyboard-reachable controls
- Reason about re-render cost and memoize expensive aggregations

## Functional Requirements

1. The dashboard must fetch a dataset and display at least three chart types (e.g. line, bar, pie) plus KPI summary cards.
2. A date-range filter must recompute every widget from the same filtered dataset.
3. KPI cards must show a headline number and a comparison against the previous period.
4. Each widget must show a loading state while fetching and a distinct error state on failure, with a retry affordance.
5. When a filter yields no data, widgets must render an explicit empty state rather than a broken chart.
6. Charts must resize responsively and remain legible on a narrow viewport.
7. Every chart must expose an accessible text alternative (summary, data table, or `aria-label`) for non-visual users.
8. Clicking a segment (a bar, a slice) must drill down into a filtered detail view of that segment.

## Suggested Milestones

1. **Milestone 1 — Fetch & render:** Load the dataset, aggregate it, and render one static chart plus KPI cards.
2. **Milestone 2 — Shared filter:** Add a date-range control and route all widgets through the filtered, memoized data.
3. **Milestone 3 — States & drill-down:** Add loading/empty/error states, responsive sizing, and click-to-drill-down.

## Data & Interface Sketch

```text
Layout
  ┌───────────────────────────────────────────┐
  │ Header: title            [ date range ▼ ]  │
  ├───────────────────────────────────────────┤
  │ [KPI: Revenue] [KPI: Orders] [KPI: Users]  │
  ├──────────────────────┬────────────────────┤
  │ Line: revenue / day  │ Pie: sales by cat.  │
  ├──────────────────────┴────────────────────┤
  │ Bar: orders by region (click → drill down) │
  └───────────────────────────────────────────┘

Record (raw, fetched)
  id: string
  date: ISO-8601 string
  category: string
  region: string
  amount: number

Derived state
  filter:     { from: Date, to: Date }
  series:     { label: string, points: { x: string, y: number }[] }[]
  kpis:       { label, value, deltaPct }[]

GET /api/records?from=<iso>&to=<iso>
  -> 200 [ { id, date, category, region, amount }, ... ]
```

## Stretch Goals

- Add real-time updates: poll or open an `EventSource`/WebSocket and merge new records into the charts.
- Let users toggle series on and off via an interactive legend.
- Add a CSV/PNG export of the current view.
- Persist the selected filter in the URL query string so a view is shareable.

## Definition of Done

- [ ] All widgets update from a single filtered dataset with no state drift between them.
- [ ] Loading, empty, and error states are visibly distinct and error offers retry.
- [ ] Charts resize cleanly from wide desktop down to a phone-width viewport.
- [ ] Each chart has a working text alternative usable without sight.
- [ ] Expensive aggregations are memoized and do not recompute on unrelated renders.

## Common Pitfalls

- Aggregating inside the render path so every keystroke recomputes the whole dataset — memoize on the filter.
- Letting each chart fetch and filter its own copy of the data, so widgets disagree after a filter change.
- Treating an empty result as an error, or rendering a chart with zero points as a blank box.
- Shipping color-only charts that fail contrast checks and are unreadable to colorblind users.
- Fixed pixel chart widths that overflow or clip on small screens.

## Resources

- [MDN: Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) — the fundamentals of loading data.
- [Recharts documentation](https://recharts.org/en-US/) — composable React charts.
- [Chart.js documentation](https://www.chartjs.org/docs/latest/) — framework-agnostic canvas charts.
- [web.dev: Accessible data visualizations](https://web.dev/articles/accessible-data-viz) — text alternatives and contrast.
- [MDN: Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) — the workhorse of client-side aggregation.
