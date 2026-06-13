---
layout: post
title: "Using Quantized Llama 3 for Document Annotation at Scale"
date: 2025-04-01 10:00:00 +0530
description: How we replaced expensive manual annotation and GPT-4 API calls with a local quantized Llama 3 deployment via Ollama — and what it took to make the outputs reliable enough for production.
tags: [llm, llama, annotation, production-ml, nlp]
categories: [engineering]
---

Manual data labeling is slow and expensive. GPT-4 API labeling is faster but costs add up at scale and has data privacy issues for enterprise clients. We ended up deploying quantized Llama 3 locally via Ollama, and it worked better than expected — with some important caveats.

## Why Local LLM Labeling

Our enterprise clients had strict data residency requirements — documents couldn't leave their on-prem environment. That ruled out any cloud API approach (GPT-4, Claude, Gemini). We needed to run inference locally.

The task: given a cluster of documents, extract representative keywords and assign topic labels. This fed downstream classification models that needed training data.

## The Setup

**Ollama** is the simplest way to run local LLMs. It handles model download, quantization, and a REST API that mimics the OpenAI interface.

```bash
# Install and start
curl -fsSL https://ollama.com/install.sh | sh
ollama serve &

# Pull the model
ollama pull llama3:8b-instruct-q4_K_M  # 4.7GB
```

`q4_K_M` is the 4-bit quantized variant. On a machine with 16GB RAM and no GPU, it runs a 512-token prompt in ~8 seconds. With a GPU (even a mid-range one), this drops to ~1-2 seconds.

## The Prompt

Getting consistent structured outputs was the hardest part. The key insight: **don't ask for freeform JSON, use a strict schema with examples**.

```python
LABEL_PROMPT = """You are a document classification assistant. Given a list of document excerpts from the same cluster, return ONLY a valid JSON object with this exact structure:

{{
  "topic_label": "<3-5 word topic label>",
  "keywords": ["keyword1", "keyword2", "keyword3", "keyword4", "keyword5"],
  "confidence": <0.0 to 1.0>
}}

Rules:
- topic_label must be 3-5 words, title case
- keywords must be exactly 5 single words, lowercase
- confidence reflects how coherent this cluster is
- Return ONLY the JSON, no explanation

Documents:
{documents}

JSON:"""
```

The `JSON:` at the end is a crucial nudge — it steers the model to start its response with `{`.

## Handling Unreliable Outputs

Even with strict prompting, LLMs produce invalid JSON ~5-15% of the time. We built a retry + fallback pipeline:

```python
import json
import re
from ollama import Client

client = Client()

def label_cluster(documents: list[str], max_retries: int = 3) -> dict:
    doc_text = "\n---\n".join(documents[:10])  # cap at 10 docs per call
    
    for attempt in range(max_retries):
        response = client.generate(
            model="llama3:8b-instruct-q4_K_M",
            prompt=LABEL_PROMPT.format(documents=doc_text),
            options={"temperature": 0.1},  # low temp for consistency
        )
        
        raw = response["response"].strip()
        
        # Extract JSON even if wrapped in markdown code blocks
        json_match = re.search(r'\{.*\}', raw, re.DOTALL)
        if not json_match:
            continue
            
        try:
            result = json.loads(json_match.group())
            if validate_output(result):
                return result
        except json.JSONDecodeError:
            continue
    
    # Fallback: return a low-confidence placeholder
    return {"topic_label": "Uncategorized", "keywords": [], "confidence": 0.0}


def validate_output(result: dict) -> bool:
    return (
        isinstance(result.get("topic_label"), str)
        and 2 <= len(result["topic_label"].split()) <= 6
        and isinstance(result.get("keywords"), list)
        and len(result["keywords"]) >= 3
        and isinstance(result.get("confidence"), (int, float))
    )
```

`temperature=0.1` is important — higher temperature makes JSON errors much more frequent.

## Throughput

We processed ~50k document clusters using a Celery queue with 4 workers, each running one Ollama instance. On CPU-only machines:

- ~8s per cluster (512-token prompt)
- ~4 workers × 3600s/hr = ~1,800 clusters/hr
- 50k clusters ≈ 28 hours

Not fast, but it ran overnight and the cost was effectively zero (no API fees, existing hardware).

## Quality Comparison

We ran a manual evaluation on 500 clusters (ground truth labeled by human annotators):

| Method | Label Accuracy | Keyword Relevance | Cost per 1k clusters |
|--------|--------------|-------------------|---------------------|
| Manual annotation | 92% | 94% | ~$150 |
| GPT-4 API | 88% | 91% | ~$12 |
| Llama 3 8B (q4) | 81% | 84% | ~$0.10 (electricity) |
| Llama 3 70B (q4) | 86% | 89% | ~$0.80 (electricity) |

The 8B model is good enough for bootstrapping training data; the 70B model is better for cases where label quality directly affects downstream model accuracy.

## What We'd Do Differently

**Use structured outputs / grammar-constrained decoding.** Ollama recently added support for `format: "json"` which forces JSON output at the sampling level. This would have eliminated most of our retry logic.

**Cache embeddings for similar clusters.** If two clusters have very similar average embeddings, they likely get the same label. Caching saves redundant LLM calls.

**Human review loop.** We used low-confidence outputs (< 0.5) as a signal for human review rather than discarding them. This turned out to be a good quality filter.

---

*The privacy + cost case for local LLMs is stronger than most teams realize. If your data can't leave the building, quantized local models are now genuinely good enough for many annotation tasks.*
