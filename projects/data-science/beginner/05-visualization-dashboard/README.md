# Data Visualization Dashboard

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

A static chart answers one question; a dashboard lets a user ask their own. In this project you take a dataset and build a small interactive dashboard where a reader can filter, select, and drill into the data to reach conclusions you did not pre-write. The challenge is restraint: a good dashboard has a clear purpose and three or four well-chosen views, not fifteen charts competing for attention. You will practice choosing the right chart per question, wiring filters to update every view at once, and designing a layout that reads top-to-bottom like an argument.

## Prerequisites

- Basic Python and pandas
- A dashboard framework (Streamlit or Dash) and a plotting library (Plotly, Matplotlib, or Seaborn)
- Comfort loading a dataset into a dataframe
- A dataset with categories to filter on and numbers to aggregate — a [Kaggle](https://www.kaggle.com/datasets) sales, weather, or sports CSV works well

## Learning Objectives

By the end, you should be able to:

- Define a dashboard's single purpose and the questions it should answer
- Match each question to an appropriate chart type
- Wire interactive filters that update multiple linked views together
- Aggregate data on the fly in response to user selections
- Lay out KPI summaries and charts so the story reads clearly

## Functional Requirements

1. The dashboard must load a dataset and display at least three distinct chart types.
2. It must provide at least one filter (dropdown, slider, or date range) that updates the views.
3. Changing a filter must update every affected chart, not just one.
4. It must show at least two summary KPI figures (totals, averages, counts).
5. Charts must have titles, axis labels, and readable legends.
6. The layout must group related views and read in a sensible order.
7. It must handle an empty filter result gracefully (no crash, a clear message).

## Suggested Milestones

1. **Milestone 1 — Static views:** Load the data and render the core charts and KPIs without interactivity.
2. **Milestone 2 — Interactivity:** Add filters and wire them so all views update from the same selection.
3. **Milestone 3 — Polish:** Arrange the layout, handle empty states, and add hover detail or annotations.

## Data & Interface Sketch

```text
Dashboard layout
  +-----------------------------------------------------+
  |  Title + one-line purpose                           |
  |  [Filter: category v] [Filter: date range]          |
  +------------------+----------------+-----------------+
  | KPI: total       | KPI: average   | KPI: count      |
  +------------------+----------------+-----------------+
  |  Trend chart (line, time on x)                      |
  +-----------------------------------------------------+
  |  Breakdown (bar) |  Distribution (histogram/box)    |
  +-----------------------------------------------------+

Interaction model
  filter change -> re-query dataframe -> recompute KPIs -> redraw all charts
  empty result  -> show "No data for this selection" instead of blank charts
```

## Stretch Goals

- Add drill-down: clicking a bar filters the other charts to that segment.
- Add an export button that downloads the currently filtered data as CSV.
- Add a date-range comparison (this period vs previous) with delta indicators.
- Deploy the dashboard publicly (Streamlit Community Cloud or similar) and share the link.

## Definition of Done

- [ ] The dashboard has a stated purpose and three or more chart types.
- [ ] At least one filter updates every dependent view simultaneously.
- [ ] KPIs recompute correctly when filters change.
- [ ] Every chart is labeled and legible without explanation.
- [ ] An empty filter selection shows a message, not a crash or blank screen.

## Common Pitfalls

- Cramming in every chart you can make instead of the few that serve the purpose.
- Filters that update one chart but leave the KPIs or other views stale.
- Recomputing the full dataset on every keystroke, making the dashboard sluggish.
- Color choices that look nice but encode nothing, or that fail for colorblind users.

## Resources

- [Streamlit documentation](https://docs.streamlit.io/) — the fastest way to a Python dashboard.
- [Plotly Python graphing library](https://plotly.com/python/) — interactive charts with built-in hover and zoom.
- [Dash documentation](https://dash.plotly.com/) — a more customizable dashboard framework.
- [Google Material: Data visualization](https://m2.material.io/design/communication/data-visualization.html) — layout and color guidance.
