# Classification Model (Iris dataset)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

The Iris dataset — 150 flowers, four measurements, three species — is the classic first classification problem, and it is small enough that you can focus entirely on the workflow rather than fighting the data. In this project you train a model to predict a flower's species from its petal and sepal measurements, then evaluate it honestly with a confusion matrix and per-class metrics. Because two of the three species overlap, you also learn that accuracy alone can hide real weaknesses, and that precision and recall tell the fuller story.

## Prerequisites

- Basic Python and pandas
- scikit-learn installed
- Understanding of the difference between classification (categories) and regression (numbers)
- The dataset is built in: `sklearn.datasets.load_iris` — see the [scikit-learn Iris reference](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_iris.html)

## Learning Objectives

By the end, you should be able to:

- Frame a multi-class classification problem and prepare labeled data
- Train a classifier (Decision Tree, k-NN, or Random Forest) and predict classes
- Read a confusion matrix to see which classes get confused
- Compute and interpret accuracy, precision, recall, and F1 per class
- Use cross-validation and a train/test split to estimate real-world performance

## Functional Requirements

1. The workflow must load the Iris data and inspect class balance and feature ranges.
2. It must split the data into stratified training and test sets.
3. It must train at least one classifier on the training set.
4. It must report accuracy plus per-class precision, recall, and F1 on the test set.
5. It must produce and interpret a confusion matrix.
6. It must use cross-validation to confirm the result is stable.
7. It must compare at least two different classifiers on the same split.

## Suggested Milestones

1. **Milestone 1 — Explore & split:** Load Iris, check class balance, do a stratified train/test split.
2. **Milestone 2 — Train & evaluate:** Fit a classifier, build the confusion matrix, report the classification metrics.
3. **Milestone 3 — Compare:** Train a second model, cross-validate both, and explain which you would pick and why.

## Data & Interface Sketch

```text
Model I/O
  features (X):  [sepal_length, sepal_width, petal_length, petal_width]  (cm, float)
  target (y):    species in {setosa, versicolor, virginica}
  prediction:    one of the three species labels

Confusion matrix (rows = actual, cols = predicted)
                 setosa  versicolor  virginica
  setosa            50        0          0
  versicolor         0       47          3
  virginica          0        2         48
  -> setosa is trivially separable; the confusion lives between versicolor & virginica

Per-class report
  precision = TP / (TP + FP)   recall = TP / (TP + FN)   F1 = harmonic mean
```

## Stretch Goals

- Tune hyperparameters with grid search and report whether it actually helped.
- Reduce to two features and plot the decision boundary to see how the classifier splits space.
- Add one-vs-rest ROC curves and compare AUC across classes.
- Build a tiny prediction interface that takes four measurements and returns a species with confidence.

## Definition of Done

- [ ] The train/test split is stratified so class balance is preserved.
- [ ] Accuracy and per-class precision/recall/F1 are all reported.
- [ ] The confusion matrix is shown and its off-diagonal cells are explained.
- [ ] At least two classifiers are compared on the same data.
- [ ] Cross-validation confirms the chosen model's score is stable.

## Common Pitfalls

- Judging the model on accuracy alone, missing that one class is systematically confused.
- Doing a non-stratified split so one species is underrepresented in the test set.
- Leaking information by scaling or tuning on the full dataset before the split.
- Overfitting a deep Decision Tree to 150 points and mistaking memorization for skill.

## Resources

- [scikit-learn: Iris dataset reference](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_iris.html) — the built-in loader and feature description.
- [scikit-learn: Classification metrics](https://scikit-learn.org/stable/modules/model_evaluation.html#classification-metrics) — precision, recall, F1, and the classification report.
- [scikit-learn: Confusion matrix](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.confusion_matrix.html) — how to compute and read it.
- [Google ML Crash Course: Classification](https://developers.google.com/machine-learning/crash-course/classification) — accessible grounding in the metrics.
