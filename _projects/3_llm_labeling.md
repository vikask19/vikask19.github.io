---
layout: page
title: LLM-Powered Data Labeling
description: Quantized Llama 3 deployed via Ollama for large-scale document annotation
img: assets/img/projects/llm_labeling.jpg
importance: 3
category: work
---

Internal system using quantized Llama 3 for automated data labeling, keyword extraction, and topic classification across large document clusters — replacing expensive manual annotation.

**What it does:**

- Automated keyword-based template creation from document clusters
- Topic classification and taxonomy labeling at scale
- Structured output extraction to feed downstream ML pipelines

**Tech stack:**

- Quantized Llama 3 deployed locally via Ollama
- Prompt engineering for consistent structured outputs
- Post-processing pipeline to validate and normalize LLM outputs

**Key challenge:** Getting consistent structured outputs from a local LLM without the cost of cloud API calls. Quantization introduced quality/speed tradeoffs that required careful prompt tuning to manage.
