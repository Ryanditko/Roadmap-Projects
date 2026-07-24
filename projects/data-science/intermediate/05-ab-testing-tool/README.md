# A/B Testing Analysis Tool

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Someone ran an experiment: version A got one button, version B got another, and now there is a spreadsheet of conversions. Did B actually win, or is the difference noise? This project builds the tool that answers that question with statistical rigour instead of vibes. You will compute the sample size a test *should* have had before it started, run the right significance test for the metric type, report a confidence interval and effect size rather than a bare p-value, and guard against the classic ways experiments lie — peeking early, testing many metrics, and confusing "not significant" with "no effect".

## Prerequisites

- Understanding of means, proportions, variance, and the normal distribution
- Familiarity with the idea of hypothesis testing (null vs alternative)
- Comfort with a stats library (SciPy, statsmodels) and a dataframe tool
- Sample experiment data with a group label and an outcome per unit

## Learning Objectives

By the end, you should be able to:

- Calculate required sample size from a baseline rate, minimum detectable effect, power, and alpha
- Pick the correct test for the metric (two-proportion z-test, Welch's t-test, chi-square)
- Report a confidence interval and effect size, not just a p-value
- Apply a multiple-comparison correction when several metrics are evaluated
- Explain why peeking at results early inflates the false-positive rate

## Functional Requirements

1. The tool must compute required sample size given baseline, MDE, power, and significance level.
2. It must select and run the appropriate test based on whether the metric is a rate or a mean.
3. It must output a point estimate, confidence interval, effect size, and p-value together.
4. It must apply a correction (Bonferroni or Benjamini-Hochberg) when more than one metric is tested.
5. It must flag when the observed sample is below the required size and warn about underpowering.
6. It must include at least one guardrail metric check alongside the primary metric.
7. It must produce a plain-language verdict ("significant lift of X% [CI]" or "inconclusive").

## Suggested Milestones

1. **Milestone 1 — Power & sizing:** Implement sample-size and power calculations.
2. **Milestone 2 — Testing:** Run the correct significance test with CI and effect size.
3. **Milestone 3 — Rigour:** Add multiple-comparison correction and guardrail/peeking checks.

## Data & Interface Sketch

```text
Experiment record (one per unit)
  unit_id : string
  group   : "control" | "variant"
  metric  : float | 0/1 (conversion)

Analysis output
  n_per_group, observed rates/means
  test_used     : "two-prop z" | "welch t" | "chi2"
  estimate      : diff of rates/means
  ci_95         : [low, high]
  effect_size   : Cohen's h / d
  p_value, corrected_p
  verdict       : "significant" | "inconclusive"
  power_warning : bool

Steps
  1. required_n = f(baseline, mde, power=0.8, alpha=0.05)
  2. choose test by metric type
  3. compute estimate, CI, effect size, p
  4. correct p across metrics
  5. render verdict + power warning
```

## Stretch Goals

- Add a sequential/Bayesian analysis so early looks are valid by design.
- Run an A/A test on real data to confirm the false-positive rate matches alpha.
- Support ratio metrics (e.g. revenue per user) with the delta method for variance.
- Add a simulation mode that generates data at a known effect to validate the tool.

## Definition of Done

- [ ] Sample size is computed from stated inputs and shown before any conclusion.
- [ ] The test chosen matches the metric type and its assumption is checked.
- [ ] Every result carries a confidence interval and effect size, not just a p-value.
- [ ] Multiple metrics trigger a correction and the corrected p-values are reported.
- [ ] The verdict is stated in plain language with the caveat when underpowered.

## Common Pitfalls

- Reporting a p-value with no effect size, so a trivial difference looks important.
- Testing ten metrics at alpha 0.05 and celebrating the one that "won" by chance.
- Treating p > 0.05 as proof of no difference rather than insufficient evidence.
- Using a t-test on a binary conversion metric where a proportion test belongs.

## Resources

- [Kohavi et al.: Trustworthy Online Controlled Experiments](https://experimentguide.com/) — the definitive practitioner reference.
- [statsmodels: Power and sample size](https://www.statsmodels.org/stable/stats.html#power-and-sample-size-calculations) — implementation reference.
- [Wikipedia: Multiple comparisons problem](https://en.wikipedia.org/wiki/Multiple_comparisons_problem) — why corrections matter.
- [Evan Miller: How Not to Run an A/B Test](https://www.evanmiller.org/how-not-to-run-an-ab-test.html) — the peeking problem explained.
