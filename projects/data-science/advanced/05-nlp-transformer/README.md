# NLP Transformer System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a production-grade NLP service around a pre-trained transformer, fine-tuned for one concrete task (classification, extraction, or summarization) and served under a real latency and memory budget. The interesting engineering lives at the edges of the model: efficient tokenization and truncation for long inputs, parameter-efficient fine-tuning so you can train on modest hardware, quantized inference so it fits in memory, and honest evaluation that includes a look at bias and failure modes. You will fine-tune, serve, measure, and interrogate the model — not just call an API.

## Prerequisites

- Understanding of the transformer architecture (attention, tokenization, embeddings)
- Experience with the Hugging Face `transformers` library
- Familiarity with a labeled text dataset for your chosen task
- Comfort with GPU memory constraints and mixed-precision training

## Learning Objectives

By the end, you should be able to:

- Fine-tune a pre-trained transformer for a specific task, including full and parameter-efficient (LoRA) approaches
- Handle tokenization, truncation, and long inputs correctly
- Optimize inference via quantization, batching, and appropriate decoding settings
- Serve the model behind an API within a latency and memory budget
- Evaluate task quality alongside bias, calibration, and failure modes

## Functional Requirements

1. The system must fine-tune a pre-trained transformer on a labeled dataset for one task.
2. Tokenization must handle truncation/padding and inputs exceeding the model's context window.
3. Fine-tuning must support a parameter-efficient method (e.g. LoRA) as an alternative to full fine-tuning.
4. Inference must be quantized and batched, with configurable decoding parameters where relevant.
5. The model must be served over an API with a documented request/response schema.
6. Evaluation must report task metrics plus at least one bias/fairness probe.
7. The optimized (quantized) model's quality must be compared against the full-precision version.

## Non-Functional Requirements

- **Latency:** serving p95 within a stated budget at the target batch size.
- **Memory:** the served model must fit within a declared GPU/CPU memory ceiling.
- **Reproducibility:** fine-tuning with a fixed seed and config reproduces reported metrics.
- **Robustness:** malformed or over-length inputs must be handled without crashing.

## Suggested Milestones

1. **Milestone 1 — Data & tokenization:** Prepare the dataset and a tokenization pipeline handling long inputs.
2. **Milestone 2 — Fine-tune:** Fine-tune the model (full and LoRA); track and compare runs.
3. **Milestone 3 — Optimize & serve:** Quantize, batch, and expose an API within budget.
4. **Milestone 4 — Evaluate & probe:** Report task metrics and run a bias/failure-mode analysis.

## Data & Interface Sketch

```text
 labeled text
     |
     v
 +--------------------+   truncate/pad, chunk long docs
 | Tokenizer          |
 +---------+----------+
           v
 +--------------------+   full FT  OR  LoRA adapters
 | Pretrained model   |   mixed precision, seed, checkpoint(best)
 | (BERT/T5/LLM)      |
 +---------+----------+
           v
 +--------------------+   int8/4-bit quantization, dynamic batching
 | Optimized inference|
 +---------+----------+
           v
   POST /infer { text | texts[] } -> { label|spans|summary, score, model_version }

 Evaluation: task metric (F1/ROUGE/acc)
           + bias probe (per-group performance gap)
           + quality delta: fp16 vs quantized
```

## Stretch Goals

- Add retrieval augmentation so the model grounds answers in a document store.
- Support multi-task inference behind one endpoint via task-specific adapters.
- Add streaming generation for summarization/generation tasks.
- Add a calibration analysis (reliability diagram) for classification confidence.

## Definition of Done

- [ ] The model is fine-tuned and beats a zero-shot or baseline on the task metric.
- [ ] Tokenization handles over-length inputs without silent truncation errors.
- [ ] A LoRA run is compared against full fine-tuning on quality and cost.
- [ ] The quantized, batched model is served within the stated latency/memory budget.
- [ ] Evaluation includes task metrics and at least one bias/fairness probe.

## Common Pitfalls

- Silently truncating long inputs and losing the part of the text that carried the signal.
- Full fine-tuning on hardware too small for it instead of reaching for LoRA/quantization.
- Comparing to no baseline, so you cannot tell if fine-tuning actually helped.
- Reporting aggregate accuracy while a demographic subgroup performs far worse.

## Resources

- [Hugging Face Transformers Documentation](https://huggingface.co/docs/transformers/index) — fine-tuning, tokenization, and inference.
- [Hugging Face PEFT (LoRA) Documentation](https://huggingface.co/docs/peft/index) — parameter-efficient fine-tuning.
- [Attention Is All You Need (Vaswani et al., 2017)](https://arxiv.org/abs/1706.03762) — the transformer architecture.
- [LoRA: Low-Rank Adaptation of Large Language Models (Hu et al., 2021)](https://arxiv.org/abs/2106.09685) — the LoRA method.
