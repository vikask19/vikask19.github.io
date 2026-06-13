---
layout: page
title: Document Intelligence Pipeline
description: Multi-stage extraction system processing 2–3 TB of documents daily at 93% F1
img: assets/img/projects/doc_pipeline.jpg
importance: 1
category: work
---

Production-scale document intelligence system at Lightbeam.ai that handles unstructured text, PDFs, and scanned images at 2–3 TB per day.

**What it does:**

- Named entity recognition and information extraction across 16+ document categories
- Handles diverse inputs: PDFs, scanned images, Word/Excel/PowerPoint files
- Achieves 93% F1 score on extraction tasks

**Tech stack:**

- Deep learning models (BERT, layout-aware transformers) optimized with ONNX for 20% faster inference
- Distributed processing with Celery and Kafka for throughput at scale
- Document classification across 16 categories at 89% accuracy

**Key challenge:** Making the pipeline reliable at scale — balancing model quality, inference speed, and queue throughput when document sizes and formats vary wildly.
