# NLP Pipeline (Tokenization + Embeddings)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Raw text is not something a model can consume — it has to become numbers first. This project builds the pipeline that does that conversion end to end: clean and tokenize documents, turn tokens into vectors (from a sparse TF-IDF matrix up to dense embeddings), and package the result so a downstream classifier or search index can use it. The point is not to train the fanciest model but to build the reusable, well-tested transformation layer that every text project needs, and to prove your representations actually capture meaning by measuring them on a small downstream task.

## Prerequisites

- Comfort with Python string handling and a dataframe library
- Understanding of what a vector is and cosine similarity
- Familiarity with train/test splitting for evaluation
- A text dataset with labels (e.g. sentiment reviews or topic-tagged news)

## Learning Objectives

By the end, you should be able to:

- Build a configurable preprocessing stage (lowercasing, tokenization, stopwords, lemmatization)
- Produce both sparse (TF-IDF) and dense (Word2Vec/GloVe or transformer) document vectors
- Fit the vectorizer on training text only and transform validation/test with it
- Measure embedding quality on a downstream classification task, not just by eyeballing
- Handle out-of-vocabulary tokens and document the vocabulary you keep

## Functional Requirements

1. The pipeline must accept raw documents and emit a documented, reusable feature matrix.
2. Preprocessing steps must be individually toggle-able and their effect observable.
3. The vectorizer must be fit on the training split only, then applied to held-out data.
4. It must offer at least two representations (TF-IDF and one embedding-based) for comparison.
5. It must evaluate each representation on the same downstream classifier and report metrics.
6. Out-of-vocabulary and empty-document cases must be handled without crashing.
7. Vocabulary size and coverage must be reported.

## Suggested Milestones

1. **Milestone 1 — Preprocess:** Tokenize, normalize, and build a clean vocabulary.
2. **Milestone 2 — Vectorize:** Produce TF-IDF and embedding representations.
3. **Milestone 3 — Evaluate:** Train a simple classifier on each and compare metrics.

## Data & Interface Sketch

```text
Document record
  doc_id : string
  text   : raw string
  label  : category   (for downstream eval)

Pipeline steps
  1. split docs -> train / valid / test
  2. preprocess: normalize -> tokenize -> stopwords -> lemmatize
  3. fit vectorizer on TRAIN tokens
       tfidf  -> sparse matrix (n_docs x vocab)
       embed  -> mean/pooled word vectors -> dense (n_docs x dim)
  4. transform valid/test with the fitted vectorizer
  5. eval: LogisticRegression on each -> accuracy / macro-F1
  6. report vocab size, OOV rate, coverage
```

## Stretch Goals

- Add subword tokenization (BPE/WordPiece) and measure its effect on OOV rate.
- Swap in contextual embeddings from a pretrained transformer and compare cost vs gain.
- Visualize embeddings in 2D (UMAP/t-SNE) coloured by label to inspect separability.
- Cache preprocessed tokens so re-running vectorization does not re-tokenize.

## Definition of Done

- [ ] The pipeline turns raw text into a feature matrix in one documented call.
- [ ] The vectorizer is fit on training data only — no vocabulary leaks from test.
- [ ] Two representations are compared on the same held-out classification task.
- [ ] OOV and empty documents are handled gracefully with a defined policy.
- [ ] Vocabulary size and OOV rate are reported.

## Common Pitfalls

- Fitting TF-IDF on the whole corpus, leaking test vocabulary and inflating scores.
- Over-cleaning text (stripping negations, emojis) and destroying signal the label depends on.
- Averaging word vectors without handling documents where every token is OOV.
- Comparing representations with different classifiers, so you cannot isolate the cause.

## Resources

- [spaCy 101](https://spacy.io/usage/spacy-101) — tokenization, lemmatization, and pipelines.
- [scikit-learn: Text feature extraction](https://scikit-learn.org/stable/modules/feature_extraction.html#text-feature-extraction) — TF-IDF in depth.
- [Jay Alammar: The Illustrated Word2Vec](https://jalammar.github.io/illustrated-word2vec/) — how word embeddings work.
- [Sentence Transformers docs](https://www.sbert.net/) — modern dense document embeddings.
