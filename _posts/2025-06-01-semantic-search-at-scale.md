---
layout: post
title: "Building Semantic Search Over 50M Documents at Sub-600ms Latency"
date: 2025-06-01 10:00:00 +0530
description: Lessons from designing and shipping a production embedding-based retrieval system at Lightbeam.ai — what worked, what didn't, and what I'd do differently.
tags: [nlp, search, production-ml, qdrant, embeddings]
categories: [engineering]
---

At Lightbeam.ai, we built a semantic search system that lets enterprise customers search across their entire document corpus — contracts, invoices, policies, emails — in under 600ms. Here's what building that actually looked like.

## The Problem

Enterprise customers had tens of millions of documents and needed to find relevant ones based on semantic meaning, not just keyword overlap. A contract clause about "force majeure" should surface when someone searches for "acts of God" — the exact words don't appear, but the meaning is the same.

Keyword search (Elasticsearch/BM25) handles volume well but falls apart on semantic relevance. We needed dense retrieval.

## The Stack

After evaluating several options, we landed on:
- **Qdrant** as the vector database (self-hosted, on-prem requirement from enterprise clients)
- **Sentence-transformers** (`all-mpnet-base-v2` initially, later domain-fine-tuned)
- **FastAPI** serving layer with async batching
- **Celery** for async indexing of new documents

## The Latency Challenge

Getting to sub-600ms end-to-end (query → encode → ANN search → rerank → return) required several iterations:

**Query encoding:** The biggest initial bottleneck was encoding the query at runtime. Even a small transformer adds ~80-100ms per query. We moved to ONNX-optimized inference, which brought this down to ~30ms.

**Index tuning:** Qdrant's HNSW index has two key parameters — `m` (connections per node) and `ef_construction` (build accuracy). We had to tune these against a recall vs latency tradeoff. Higher `m` → better recall, slower search. We settled on `m=16, ef=100` which gave us 95%+ recall at acceptable latency.

**Dimensionality:** Going from 768-dim to 256-dim (via fine-tuned projection) cut search latency by ~40% with minimal recall loss on our domain.

**Reranking:** Cross-encoder reranking significantly improves precision but adds ~200-300ms. We limited reranking to the top-20 ANN results and made it optional (a query parameter) for latency-sensitive use cases.

## What Didn't Work

- **Scalar quantization** helped theoretically but caused precision issues on domain-specific financial text — the quantization artifacts hurt recall on rare terms
- **Approximate hybrid search** (dense + sparse) sounded great but was complex to tune; the latency gains from pure dense were sufficient for our recall requirements
- **Single large index** didn't scale well for multi-tenant isolation; we moved to per-customer collections

## Results

- 50M+ documents indexed
- p99 latency: 580ms (end-to-end, including query encoding)
- Recall@20: 94.3% vs ground truth
- Index update lag: <2 minutes for new documents

## What I'd Do Differently

**Start with domain fine-tuning earlier.** We spent months tuning the index before fine-tuning the encoder. A domain-adapted encoder with default index settings outperformed a default encoder with heavily tuned index.

**Build query analytics from day one.** Understanding what users are actually searching for is invaluable for both encoder fine-tuning and recall evaluation. We added this too late.

**Separate the SLA for rare vs common queries.** Common queries hit a cache; rare queries hit the full ANN path. We didn't do this initially and burned latency budget on cache misses for popular searches.

---

*This is part of a series on ML systems we built at Lightbeam. Next up: how we got BERT inference 20% faster with ONNX.*
