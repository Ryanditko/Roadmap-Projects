# Deep Learning Image Classifier

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Train a convolutional image classifier to production quality on a real dataset, then optimize it for deployment. The modeling is only half the project: the advanced part is doing transfer learning correctly (freeze, then fine-tune with discriminative learning rates), building an augmentation pipeline that helps rather than hurts, and then shrinking the trained network so it can serve within a memory and latency budget. You will end with a model, a reproducible training pipeline, and an optimized artifact whose accuracy loss you can quantify.

## Prerequisites

- Working knowledge of neural networks and backpropagation
- Experience with PyTorch or TensorFlow/Keras and GPU training
- Familiarity with a labeled image dataset (CIFAR-100, Food-101, or your own)
- Understanding of overfitting, regularization, and learning-rate schedules

## Learning Objectives

By the end, you should be able to:

- Apply transfer learning: freeze a backbone, then fine-tune with layer-wise learning rates
- Build an augmentation pipeline and measure its effect on validation accuracy
- Use learning-rate scheduling, early stopping, and checkpointing in a real training loop
- Optimize a trained model via quantization, pruning, or export (ONNX/TFLite)
- Quantify the accuracy-vs-size-vs-latency tradeoff of the optimized model

## Functional Requirements

1. The pipeline must load a labeled image dataset with reproducible train/val/test splits.
2. Training must start from a pre-trained backbone (ResNet, EfficientNet, or MobileNet) and fine-tune it.
3. An augmentation pipeline must be configurable and applied only to the training split.
4. Training must checkpoint the best model by validation metric and support resuming.
5. The system must report per-class metrics and a confusion matrix on the test split.
6. The trained model must be exported and optimized for inference (quantized and/or ONNX/TFLite).
7. The optimized model's accuracy must be measured against the full-precision baseline.

## Non-Functional Requirements

- **Reproducibility:** seeds, splits, and augmentation config must reproduce reported metrics.
- **Deployment budget:** optimized model must meet a stated size (e.g. ≤ 25 MB) and CPU latency target.
- **Accuracy tolerance:** optimization-induced accuracy drop must stay within a documented bound.
- **Throughput:** batched inference throughput on target hardware must be reported.

## Suggested Milestones

1. **Milestone 1 — Data & baseline:** Reproducible splits, a data loader, and a from-scratch or frozen-backbone baseline.
2. **Milestone 2 — Transfer learning:** Fine-tune with discriminative learning rates and augmentation; beat the baseline.
3. **Milestone 3 — Evaluation:** Per-class metrics, confusion matrix, and error analysis on the test split.
4. **Milestone 4 — Optimize & export:** Quantize/prune, export, and quantify the accuracy/size/latency tradeoff.

## Data & Interface Sketch

```text
 dataset/
   train/ val/ test/   (stratified, seeded split)
        |
        v
 +---------------------+   train-only: flips, crop, color jitter, mixup
 | Augmentation        |
 +----------+----------+
            v
 +---------------------+   backbone frozen -> then fine-tune
 | Pretrained CNN      |   ResNet/EfficientNet/MobileNet
 |  + new head         |   LR schedule, early stop, checkpoint(best)
 +----------+----------+
            v
 +---------------------+
 | Evaluation          |   per-class P/R/F1, confusion matrix
 +----------+----------+
            v
 +---------------------+   quantize / prune / export
 | Optimized artifact  |   ONNX / TFLite
 +---------------------+
   report: {acc_fp32, acc_int8, size_mb, cpu_latency_ms}
```

## Stretch Goals

- Add Grad-CAM visualizations to inspect what the model attends to.
- Handle class imbalance with weighted loss or focal loss and measure the effect.
- Add test-time augmentation and compare accuracy vs latency cost.
- Distill the fine-tuned model into a smaller student network.

## Definition of Done

- [ ] Splits, seeds, and augmentation config reproduce the reported metrics.
- [ ] The fine-tuned transfer-learning model beats the baseline on the test split.
- [ ] Per-class metrics and a confusion matrix are produced and briefly analyzed.
- [ ] An optimized artifact is exported and meets the stated size/latency budget.
- [ ] The accuracy delta between full-precision and optimized models is documented.

## Common Pitfalls

- Applying augmentation to the validation/test splits and inflating apparent robustness.
- Fine-tuning the whole network at a high learning rate and destroying the pre-trained weights.
- Reporting only top-1 accuracy while a minority class quietly performs terribly.
- Forgetting to re-measure accuracy after quantization and shipping a degraded model.

## Resources

- [PyTorch Transfer Learning Tutorial](https://pytorch.org/tutorials/beginner/transfer_learning_tutorial.html) — freezing and fine-tuning done right.
- [TensorFlow: Data augmentation](https://www.tensorflow.org/tutorials/images/data_augmentation) — building an augmentation pipeline.
- [PyTorch Quantization Documentation](https://pytorch.org/docs/stable/quantization.html) — post-training and quantization-aware approaches.
- [Deep Residual Learning for Image Recognition (He et al., 2015)](https://arxiv.org/abs/1512.03385) — the ResNet paper behind most modern backbones.
