---
layout: post
title: "20% Faster BERT Inference with ONNX: A Production Walkthrough"
date: 2025-05-01 10:00:00 +0530
description: How we exported a fine-tuned BERT document classifier to ONNX and cut inference latency by 20% without touching the model architecture.
tags: [nlp, bert, onnx, production-ml, inference]
categories: [engineering]
---

When you're running BERT inference on millions of documents a day, a 20% latency reduction compounds into real infrastructure savings. Here's exactly how we did it.

## The Context

At Lightbeam.ai, we had a document classification model (BERT fine-tuned on 16 document categories) running in production. It was accurate (89% on our test set) but slow — average inference of ~120ms per document, which made it the bottleneck in our extraction pipeline.

We weren't willing to sacrifice accuracy by moving to a smaller model. ONNX Runtime turned out to be the right answer.

## What ONNX Actually Does

ONNX (Open Neural Network Exchange) is a format for representing models, and ONNX Runtime is an optimized inference engine. When you export a PyTorch model to ONNX, the runtime can:

1. **Fuse operations** — combine sequential ops like LayerNorm + GELU into a single optimized kernel
2. **Use hardware-specific backends** — CPU optimizations (oneDNN/MKL) or GPU (TensorRT/CUDA)
3. **Apply graph optimizations** — constant folding, dead code elimination

## The Export

```python
import torch
from transformers import AutoModelForSequenceClassification, AutoTokenizer

model = AutoModelForSequenceClassification.from_pretrained("./bert-doc-classifier")
tokenizer = AutoTokenizer.from_pretrained("./bert-doc-classifier")
model.eval()

# Create dummy input for tracing
dummy_input = tokenizer(
    "Sample document text for tracing",
    return_tensors="pt",
    max_length=512,
    padding="max_length",
    truncation=True
)

# Export
torch.onnx.export(
    model,
    (dummy_input["input_ids"], dummy_input["attention_mask"]),
    "bert_classifier.onnx",
    input_names=["input_ids", "attention_mask"],
    output_names=["logits"],
    dynamic_axes={
        "input_ids": {0: "batch_size", 1: "sequence"},
        "attention_mask": {0: "batch_size", 1: "sequence"},
    },
    opset_version=14,
)
```

**Important:** `dynamic_axes` is critical for production — without it, the model is fixed to the dummy input's batch size and sequence length.

## Optimization Pass

After export, run ONNX Runtime's optimization:

```python
from onnxruntime.transformers import optimizer

optimized_model = optimizer.optimize_model(
    "bert_classifier.onnx",
    model_type="bert",
    num_heads=12,
    hidden_size=768,
    optimization_level=99,  # all optimizations
    use_gpu=False,
)
optimized_model.save_model_to_file("bert_classifier_optimized.onnx")
```

The `model_type="bert"` hint lets the optimizer apply BERT-specific fusions (attention + LayerNorm fusion, etc.).

## Inference

```python
import onnxruntime as ort
import numpy as np

session = ort.InferenceSession(
    "bert_classifier_optimized.onnx",
    providers=["CPUExecutionProvider"],
)

def predict(texts: list[str]) -> np.ndarray:
    inputs = tokenizer(
        texts,
        return_tensors="np",
        max_length=512,
        padding=True,
        truncation=True,
    )
    outputs = session.run(
        ["logits"],
        {"input_ids": inputs["input_ids"], "attention_mask": inputs["attention_mask"]},
    )
    return outputs[0]
```

## Results

| | PyTorch | ONNX (optimized) |
|--|---------|-----------------|
| Single doc (ms) | 120 | 95 |
| Batch=8 (ms) | 380 | 290 |
| Batch=32 (ms) | 1200 | 960 |
| Accuracy | 89.1% | 89.1% |

**~21% speedup** on single doc inference, ~20% on batches. No accuracy regression — ONNX export is deterministic for inference.

## What to Watch Out For

**Dynamic shapes:** If you forget `dynamic_axes`, inference will silently fail or produce wrong results on inputs with different shapes than the export dummy.

**Tokenizer is NOT exported:** The tokenizer stays in Python/HuggingFace. ONNX only covers the model forward pass.

**Validate outputs:** Always compare logits between PyTorch and ONNX on 100+ samples before deploying. Numerical precision can differ slightly (usually within 1e-5).

**Provider order matters:** `["CUDAExecutionProvider", "CPUExecutionProvider"]` — ORT will fall back to CPU if CUDA is unavailable, which is what you want for resilient production deploys.

---

*Running this in a Celery worker? Add `num_threads` configuration to your `SessionOptions` to prevent ORT from spawning too many threads when you have many concurrent workers.*
