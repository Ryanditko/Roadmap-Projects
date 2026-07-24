# Data-Heavy Dashboard (virtualization)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build an analytics dashboard that presents tens of thousands of data points across tables and charts, stays responsive while the user filters and drills down, and never freezes the browser under the weight of its own data. This is the frontier where data visualization meets performance engineering: a naive render of 50,000 rows or a chart with 100,000 points will lock the main thread and drop frames. You will combine virtualization (only render what is visible), server- or worker-side aggregation (summarize before you draw), and canvas/WebGL rendering for the heaviest visuals. The goal is a dashboard that feels instant regardless of dataset size, backed by measured proof rather than a "feels fine on my machine" claim.

## Prerequisites

- Comfortable rendering charts with a visualization library or the Canvas API
- Understanding of virtualization/windowing for long lists
- Familiarity with data aggregation concepts (grouping, bucketing, downsampling)
- Awareness of the browser's frame budget (~16ms per frame for 60fps)

## Learning Objectives

By the end, you should be able to:

- Virtualize large tables and lists so the DOM stays small regardless of row count
- Aggregate and downsample data so charts draw a meaningful summary, not every raw point
- Choose the right rendering surface (SVG vs. Canvas vs. WebGL) for a given data volume
- Keep filtering and drill-down interactions responsive by offloading heavy work
- Measure and defend interaction latency against a frame budget

## Functional Requirements

1. A table must display a dataset of at least 50,000 rows and scroll smoothly via virtualization.
2. Charts must render large series without dropping below an acceptable frame rate, using aggregation/downsampling as needed.
3. Filtering the dataset must update tables and charts responsively, without freezing the UI.
4. Drill-down (e.g. click a bar to see its underlying rows) must load and render the detail quickly.
5. Sorting and searching large data must not block the main thread perceptibly.
6. Memory use must stay bounded as the user navigates, with no unbounded growth.
7. Every heavy interaction must have a measured latency that meets a stated budget.

## Suggested Milestones

1. **Milestone 1 — Virtualized table:** Render 50k+ rows with windowing; verify constant DOM size.
2. **Milestone 2 — Aggregated charts:** Draw large series with downsampling on an appropriate surface (Canvas/WebGL).
3. **Milestone 3 — Filter & drill-down:** Add responsive filtering and click-through to detail views.
4. **Milestone 4 — Offload & measure:** Move sorting/aggregation off the main thread and record latencies.

## Data & Interface Sketch

```text
   Raw dataset (50k+ rows)
        │
        ├─▶ Aggregation layer (worker / server)
        │      group · bucket · downsample  ──▶ chart-ready summary
        │
        └─▶ Virtualized table
               render only visible rows (windowing)
               ┌──────────── viewport ────────────┐
               │ row 8,201 ... row 8,230 (~30 DOM) │
               └───────────────────────────────────┘

   Rendering surface by volume:
     < 1k points     SVG (interactive, accessible)
     1k–50k points   Canvas 2D
     > 50k points    WebGL

Interaction:  filter -> recompute aggregate (worker) -> repaint
Non-functional targets:
  table scroll         60 fps, DOM node count constant
  filter -> update     < 200 ms perceived
  sort 50k rows        off main thread, UI stays interactive
  memory               bounded across navigation
```

## Stretch Goals

- Add incremental/streaming data loading so the dashboard populates progressively.
- Add a "zoom + pan" time-series view that re-aggregates at the visible resolution.
- Add CSV/Parquet export of the current filtered view.
- Add a heatmap or geographic visualization for a large categorical dimension.

## Definition of Done

- [ ] A 50,000-row table scrolls at 60fps with a constant DOM node count.
- [ ] A large chart renders and stays interactive using aggregation/downsampling.
- [ ] Applying a filter updates all views within the stated latency budget.
- [ ] Sorting a large dataset does not visibly freeze the interface.
- [ ] Memory stays bounded through repeated filtering and drill-down.

## Common Pitfalls

- Rendering every data point to SVG, creating tens of thousands of DOM nodes that grind the browser.
- Aggregating on the main thread, so a filter change freezes the UI for seconds.
- Downsampling naively (dropping points) and hiding real spikes the user needs to see.
- Virtualizing the table but re-mounting rows on every scroll tick, defeating the optimization.
- Reporting "feels fast" without measuring interaction latency on a realistic dataset.

## Resources

- [TanStack Virtual](https://tanstack.com/virtual/latest) — a headless library for virtualizing large lists, tables, and grids.
- [web.dev: Rendering performance](https://web.dev/articles/rendering-performance) — the frame budget and how to hit 60fps.
- [MDN: Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) — drawing large visuals without per-point DOM nodes.
- [Observable Plot](https://observablehq.com/plot/) — a concise grammar for building data visualizations.
