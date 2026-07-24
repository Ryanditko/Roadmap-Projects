# Design a Search Engine

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design the backend for a full-text search engine that crawls or ingests documents, builds an inverted index, and answers ranked keyword queries in milliseconds. Think of a site search or a smaller-scale web index. The heart of the system is the inverted index — how you build it, shard it, keep it fresh, and query it across many machines. This is a design exercise: you produce a design document, capacity numbers, and diagrams — not working code.

## Prerequisites

- Understanding of the inverted index and tokenization
- Familiarity with TF-IDF or BM25 relevance scoring at a conceptual level
- Awareness of sharding, replication, and scatter-gather queries
- Comfort estimating index size relative to corpus size

## Learning Objectives

By the end, you should be able to:

- Design an indexing pipeline: fetch → parse → tokenize → post to index
- Estimate corpus size, inverted index size, and query QPS
- Shard the index (by document vs. by term) and run scatter-gather queries
- Design a caching layer for popular queries and a plan for index freshness
- Justify trade-offs between document-partitioned and term-partitioned indexes

## Requirements & Constraints

- Assume 500M documents (avg 5 KB text), ~10k queries/s peak, index updated continuously.
- Query latency must stay under ~200 ms p99, including ranking.
- New/updated documents should become searchable within minutes.
- Results are ranked by relevance; support basic filters (date, type).
- Estimate raw corpus storage, inverted index size, and query throughput.

## Suggested Approach

1. Design the ingestion pipeline and how documents flow into segments.
2. Estimate index size (postings lists) relative to the 500M-document corpus.
3. Choose a partitioning scheme: document-partitioned shards with scatter-gather.
4. Add a query cache and result cache for head queries.
5. Design near-real-time indexing: small segments merged in the background.

## Architecture Sketch

```text
Crawler/Ingest -> Parser/Tokenizer -> [Indexer] -> Segment writer -> Shard 1..N (inverted index)
                                                       |-> background merge/compaction

Query -> [Query svc] -> query cache? -> scatter to all shards -> gather top-k -> rank (BM25) -> merge
                                                                    |-> result cache

GET /search?q=distributed+systems&page=1 -> 200 { results[], total, tookMs }

Document { docId, url, title, body, ts }        // partition by docId hash (doc-partitioned)
Postings { term -> [(docId, tf, positions), ...] }  // per-shard inverted index
```

## Deep-Dive Topics

- **Index construction:** tokenization, stemming, stop words; segment-based writes and merges.
- **Ranking:** BM25 scoring, combining term frequency with document frequency across shards.
- **Trade-off 1 — document vs. term partitioning:** document-partitioned scatters every query to all shards but writes are local and simple; term-partitioned routes a query to few shards but makes indexing a document touch many shards. Justify document-partitioning for balanced load.
- **Trade-off 2 — freshness vs. throughput:** frequent small segments keep results fresh but hurt query speed until merged; batch merges are efficient but delay visibility.

## Deliverables

- [ ] A design document (~3–5 pages) expanding the architecture above.
- [ ] Capacity estimates: corpus storage, inverted index size, query QPS, shard count.
- [ ] A partitioning plan (document- vs. term-partitioned) with rationale.
- [ ] A caching strategy for query and result caches, with invalidation on updates.
- [ ] At least two trade-offs, each with the option chosen and why.

## Common Pitfalls

- Rebuilding the whole index on every update instead of writing incremental segments.
- Ignoring the scatter-gather tail: the slowest shard sets query latency.
- Underestimating index size — positions and metadata can rival the raw text.
- Caching results without invalidating them when the underlying documents change.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — sharding and scatter-gather.
- [Elasticsearch: what is an inverted index](https://www.elastic.co/blog/found-elasticsearch-from-the-bottom-up) — index internals explained.
- [BM25 (Wikipedia)](https://en.wikipedia.org/wiki/Okapi_BM25) — the standard relevance scoring function.
- [Introduction to Information Retrieval (Manning et al.)](https://nlp.stanford.edu/IR-book/) — the reference text for indexing and ranking.
