---
layout: page
title: mle path
permalink: /mle-path/
description: My journey from IIT Kharagpur to Uber — what I built, learned, and wish I'd known earlier.
nav: true
nav_order: 3
---

This page is for engineers on a similar path — ML beginners, early-career data scientists, or anyone trying to understand what a career in production ML actually looks like. Not a polished LinkedIn narrative; more of an honest map.

---

## The Arc

### 🎓 IIT Kharagpur (2012–2017)
Dual Degree in Agricultural & Food Engineering — not ML. I got into ML through Kaggle during my final year, which taught me more about practical modeling than any course.

**What clicked:** Kaggle forced me to think about what actually works in practice, not just in theory. Leaderboard feedback is brutally honest.

---

### 📊 EXL Analytics — Data Scientist (2017–2019)
First real job. Document extraction, speech recognition (DeepSpeech / WaveNet), and one of the early ULMFiT-based email classifiers in production.

**What I built:**
- Email intent classifier using ULMFiT transfer learning — 89% accuracy, outperformed XGBoost by 8%
- Computer vision pipeline using Mask R-CNN for bill of lading extraction
- Speech recognition platform comparing DeepSpeech vs WaveNet (21% vs 34% WER)

**What I learned:** Deep learning in production is 80% data quality and pipeline reliability, 20% model architecture. The research paper's numbers never transfer directly.

---

### 🏢 Deloitte Consulting USI — Applied AI Consultant (2019–2020)
Fortune 500 clients across insurance, healthcare, and media. Learned how to translate messy business problems into ML solutions and how to communicate tradeoffs to non-technical stakeholders.

**What I built:**
- Email response classification (XGBoost + BERT, 81% F1 across 20+ categories, automating 60% of responses)
- Market intelligence pipeline with XGBoost for biologic market sizing

**What I learned:** $2M+ in measurable ROI teaches you to care about business metrics, not just model metrics. The best model is the one that ships and gets used.

---

### 🚀 Lightbeam.ai — Founding ML Engineer → ML Lead (2020–2025)
Joined as the first ML hire at a document intelligence startup. Built the core AI from scratch.

**What I built:**
- Multi-stage extraction pipeline: 2–3 TB/day, 5M+ documents, 93% F1 across text/PDFs/scanned images (Celery + Kafka)
- Semantic search: 50M+ documents, sub-600ms latency, Qdrant vector DB
- Document classification: 16 categories, 89% accuracy, BERT + ONNX (20% faster inference)
- LLM data labeling: quantized Llama 3 via Ollama for annotation at scale

**What I learned:** Scaling from prototype to production is a different skill than building the first version. Distributed systems, observability, and incremental model updates matter more than model accuracy at scale.

Grew from individual contributor to leading a team of 4+ engineers. The hardest part of ML lead isn't the ML — it's designing experiments others can run, and making model decisions defensible to product and customers.

---

### ⚡ Uber — Senior ML Engineer (2025–present)
Designing and scaling high-performance ML systems for marketplace mechanics.

*Still early — will update as I go.*

---

## Resources That Actually Helped

These aren't generic "learn ML" recommendations — these are things that made a real difference at specific points in my journey.

| Resource | When It Helped | Why |
|----------|---------------|-----|
| [Designing ML Systems](https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/) — Chip Huyen | Before Lightbeam | The only book that covers the full lifecycle honestly |
| [The Morning Paper](https://blog.acolyer.org) | Throughout | Keeps you connected to research without reading every arxiv paper |
| Kaggle competitions | 2016–2021 | Fastest way to build intuition for what works |
| [Eugene Yan's Blog](https://eugeneyan.com) | At EXL/Deloitte | ML in production, not just ML in notebooks |
| FastAI (ULMFiT paper + course) | 2018 | Transfer learning before it was mainstream |

---

## What I'd Do Differently

1. **Learn systems earlier.** Kafka, distributed queues, observability — I learned these reactively when things broke at scale. Better to understand them before you need them.
2. **Write more.** I built a lot but wrote little about it. Writing forces clarity and compounds over time (this site is my attempt to fix that).
3. **Kaggle earlier.** If you're starting out, spend 6 months doing Kaggle competitions seriously. Nothing else teaches modeling intuition as efficiently.
4. **Don't wait to be "ready" for senior roles.** I underestimated myself in early career. The gap between senior and non-senior is mostly about owning problems end-to-end, not raw technical knowledge.

---

*This page is a living document — I'll update it as I learn more.*
