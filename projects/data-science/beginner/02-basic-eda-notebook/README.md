# Basic EDA Notebook

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Before anyone trains a model, someone has to actually look at the data. Exploratory Data Analysis (EDA) is that first honest look — you describe each variable, plot its distribution, check how features relate, and surface the questions worth asking next. In this project you build an EDA notebook on a dataset of your choice and, crucially, end with written findings a non-technical reader could follow. The deliverable is not a wall of charts but a narrative: here is what the data contains, here is what surprised me, here is what I would investigate further.

## Prerequisites

- Basic Python and a dataframe library (pandas)
- A plotting library (Matplotlib or Seaborn)
- Comfort running a Jupyter or Colab notebook
- A tabular dataset with a mix of numeric and categorical columns — the [UCI Adult / Census Income dataset](https://archive.ics.uci.edu/dataset/2/adult) or a Kaggle CSV are good picks

## Learning Objectives

By the end, you should be able to:

- Summarize a dataset's shape, types, and descriptive statistics
- Choose the right chart for a variable (histogram, box plot, bar chart, scatter)
- Read a correlation matrix and reason about relationships, not just numbers
- Spot missing values, outliers, and suspicious distributions
- Turn observations into clear, prioritized questions and a written summary

## Functional Requirements

1. The notebook must load the dataset and report its dimensions and per-column data types.
2. It must produce descriptive statistics for numeric columns and value counts for categorical ones.
3. It must visualize the distribution of at least three variables with appropriate chart types.
4. It must show relationships between at least two pairs of variables (e.g. scatter or grouped bar).
5. It must include a correlation matrix for numeric features with a short interpretation.
6. It must explicitly report missing values and any outliers found.
7. It must end with a written summary of findings and follow-up questions.

## Suggested Milestones

1. **Milestone 1 — Describe:** Load the data, report shape and types, generate descriptive stats and value counts.
2. **Milestone 2 — Visualize:** Plot distributions and relationships; build the correlation matrix.
3. **Milestone 3 — Synthesize:** Document missing values, outliers, and a prioritized list of findings and next questions.

## Data & Interface Sketch

```text
Notebook structure (top to bottom)
  1. Setup & load        -> df, shape (rows, cols), dtypes
  2. Univariate          -> describe() for numerics, value_counts() for categoricals
                            histograms + box plots
  3. Bivariate           -> scatter (num vs num), grouped bar (cat vs num)
  4. Correlation         -> numeric correlation matrix + heatmap
  5. Data quality        -> null counts per column, outlier notes
  6. Findings            -> 3-5 bullet insights + open questions

Chart-to-question mapping
  distribution of one variable   -> histogram / box plot
  category frequencies           -> bar chart
  relationship between two nums   -> scatter plot
  numeric split by category      -> grouped/box by group
```

## Stretch Goals

- Add an automated profiling report (ydata-profiling / pandas-profiling) and compare it to your hand-made analysis.
- Formulate one hypothesis and test it with a simple statistical test (t-test or chi-square).
- Add interactive charts with Plotly so a reader can hover and filter.
- Segment the analysis by a key category and compare distributions across segments.

## Definition of Done

- [ ] The notebook runs top to bottom without errors on a fresh kernel.
- [ ] Every chart has a title, axis labels, and a one-line takeaway.
- [ ] Missing values and outliers are quantified, not just mentioned.
- [ ] The correlation matrix is interpreted in words, not left as a raw grid.
- [ ] The final summary states findings a non-technical reader could understand.

## Common Pitfalls

- Plotting dozens of charts with no narrative, so the reader learns nothing.
- Reading correlation as causation — a strong coefficient is a hint, not a conclusion.
- Using a histogram for a categorical variable or a bar chart for a continuous one.
- Ignoring axis scales, so an outlier flattens every other bar into invisibility.

## Resources

- [pandas: Essential basic functionality](https://pandas.pydata.org/docs/user_guide/basics.html) — `describe`, `info`, `value_counts` and friends.
- [Seaborn: Overview of plotting functions](https://seaborn.pydata.org/tutorial/function_overview.html) — picking the right chart.
- [From Data to Viz](https://www.data-to-viz.com/) — a decision tree from data shape to appropriate chart.
- [ydata-profiling docs](https://docs.profiling.ydata.ai/) — automated EDA reports for the stretch goal.
