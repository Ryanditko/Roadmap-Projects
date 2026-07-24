# Text Sentiment Analysis

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Sentiment analysis turns free text — a review, a tweet, a support ticket — into a label like positive or negative. It is a gentle first step into Natural Language Processing because the pipeline is concrete: clean the text, turn words into numbers, train a classifier, and check where it goes wrong. In this project you build that end-to-end on a labeled dataset of reviews, and you spend real time on the failure cases, because the interesting lessons in NLP live in the misclassifications — negation, sarcasm, and domain-specific slang that a bag-of-words model simply cannot see.

## Prerequisites

- Basic Python and pandas
- scikit-learn installed
- Understanding of what a classifier does (maps features to a label)
- A labeled text dataset — the [IMDb movie reviews](https://ai.stanford.edu/~amaas/data/sentiment/) or the [UCI Sentiment Labelled Sentences dataset](https://archive.ics.uci.edu/dataset/331/sentiment+labelled+sentences) are good choices

## Learning Objectives

By the end, you should be able to:

- Preprocess raw text (lowercasing, tokenization, stopword handling)
- Convert text into numeric features with a bag-of-words or TF-IDF representation
- Train and evaluate a text classifier (Naive Bayes or Logistic Regression)
- Inspect misclassifications to understand model limitations
- Explain why a linear bag-of-words model struggles with negation and sarcasm

## Functional Requirements

1. The workflow must load a labeled text dataset and report class balance.
2. It must apply a documented preprocessing pipeline to the raw text.
3. It must vectorize the text into numeric features (bag-of-words or TF-IDF).
4. It must train a classifier and evaluate it on a held-out test set.
5. It must report accuracy, precision, recall, and F1, plus a confusion matrix.
6. It must surface and discuss at least three misclassified examples.
7. It must predict the sentiment of a new, hand-written sentence.

## Suggested Milestones

1. **Milestone 1 — Preprocess & vectorize:** Clean the text and turn it into a TF-IDF feature matrix.
2. **Milestone 2 — Train & evaluate:** Fit a classifier, report metrics, and build the confusion matrix.
3. **Milestone 3 — Error analysis:** Examine misclassifications, identify patterns, and test on your own sentences.

## Data & Interface Sketch

```text
Model pipeline (text -> label)
  raw text        "This movie was NOT good at all."
    -> preprocess  lowercase, strip punctuation, tokenize, (optional) remove stopwords
    -> vectorize   TF-IDF -> sparse vector [n_features]
    -> classify    -> label in {positive, negative}  (+ probability)

Data shape
  input:   { text: string, label: "positive" | "negative" }
  vector:  each token -> a weighted column; document -> a row of weights

Error-analysis table
  text                        | true      | predicted | likely cause
  "not good at all"           | negative  | positive  | negation lost in bag-of-words
  "yeah, great, another bug"  | negative  | positive  | sarcasm
```

## Stretch Goals

- Add bigrams so "not good" becomes a single feature and compare the metrics.
- Compare TF-IDF against simple word counts on the same classifier.
- Add a probability threshold so low-confidence predictions are marked "uncertain".
- Try a pretrained sentiment model (e.g. a Hugging Face pipeline) and compare it to your own.

## Definition of Done

- [ ] The preprocessing steps are documented and applied consistently to train and test.
- [ ] Text is vectorized and a classifier is trained on the training split only.
- [ ] Accuracy, precision, recall, F1, and a confusion matrix are all reported.
- [ ] At least three misclassifications are shown with a plausible explanation.
- [ ] The model classifies a new hand-written sentence end to end.

## Common Pitfalls

- Fitting the vectorizer on the full dataset, leaking test vocabulary into training.
- Removing stopwords blindly — "not" and "no" carry the sentiment you care about.
- Reporting only accuracy on an imbalanced dataset that a constant guess would beat.
- Expecting a bag-of-words model to catch sarcasm or word order it fundamentally ignores.

## Resources

- [scikit-learn: Working with text data](https://scikit-learn.org/stable/tutorial/text_analytics/working_with_text_data.html) — the canonical text-classification tutorial.
- [scikit-learn: TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html) — turning text into features.
- [NLTK Book, Chapter 6: Text Classification](https://www.nltk.org/book/ch06.html) — preprocessing and classification fundamentals.
- [Hugging Face: Sentiment analysis pipeline](https://huggingface.co/docs/transformers/main/en/quicktour) — a pretrained baseline for the stretch goal.
