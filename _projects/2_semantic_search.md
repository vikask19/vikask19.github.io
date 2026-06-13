---
layout: page
title: Semantic Search at Scale
description: Real-time document similarity search across 20M+ documents in sub-600ms
img: assets/img/projects/semantic_search.jpg
importance: 2
category: work
---

Production semantic search system enabling real-time document similarity and retrieval across a corpus of 20M+ enterprise documents.

**What it does:**

- Dense embedding-based search with sub-600ms end-to-end latency
- Supports fuzzy matching, semantic similarity, and hybrid retrieval
- Scales to millions of queries per day in a multi-tenant environment

**Tech stack:**

- Qdrant vector database for dense embedding storage and ANN retrieval
- HuggingFace sentence transformers for document encoding
- FastAPI serving layer with async processing

**Key challenge:** Keeping latency under 600ms at the 99th percentile while new documents are continuously indexed. Required careful index configuration and embedding dimensionality tradeoffs.
