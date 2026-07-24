# Real-Time Prediction System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a model-serving system that answers prediction requests at high volume with tight, predictable latency. Anyone can wrap a model in a Flask route; the hard part is holding p99 latency under budget while requests arrive in bursts, features must be fetched or computed on the fly, and the model itself may be large. This project pushes you into the real serving toolbox: request batching, model optimization (quantization, ONNX export), a warm feature cache, and graceful degradation when a dependency slows down. You will measure everything, because "fast enough" is only meaningful against numbers.

## Prerequisites

- A trained model you can export (scikit-learn, PyTorch, or TensorFlow)
- Solid grasp of HTTP services, concurrency, and async I/O
- Familiarity with a cache (Redis) and basic load testing (Locust, k6, or wrk)
- Comfort reading latency percentiles, not just averages

## Learning Objectives

By the end, you should be able to:

- Optimize a model for inference via quantization, pruning, or ONNX/TensorRT export
- Implement dynamic request batching to raise throughput without wrecking latency
- Design a feature cache and reason about staleness vs freshness
- Apply backpressure, timeouts, and circuit breakers so overload fails cleanly
- Measure and defend p50/p95/p99 latency and throughput under load

## Functional Requirements

1. The system must serve predictions over an API with a documented request/response schema.
2. Incoming requests must be dynamically batched up to a size/time window before inference.
3. Frequently used features must be served from a cache with an explicit TTL and miss path.
4. The system must enforce per-request timeouts and shed or queue load when saturated.
5. A fallback (cached prediction, default, or lighter model) must engage when the primary path fails.
6. The system must expose latency and throughput metrics per endpoint.
7. The optimized model's accuracy must be validated against the unoptimized baseline within a stated tolerance.

## Non-Functional Requirements

- **Latency:** p95 ≤ a stated budget (e.g. 50 ms) and p99 bounded, under the target request rate.
- **Throughput:** sustain a defined RPS (e.g. 1,000) on the target hardware.
- **Availability:** degrade rather than crash under overload; no unbounded queues.
- **Consistency:** feature cache staleness must be bounded and documented.

## Suggested Milestones

1. **Milestone 1 — Baseline serving:** Expose the model behind an API and measure baseline latency/throughput.
2. **Milestone 2 — Optimize the model:** Export to ONNX and/or quantize; verify accuracy delta and speedup.
3. **Milestone 3 — Batching & caching:** Add dynamic batching and a feature cache; re-measure.
4. **Milestone 4 — Resilience:** Add timeouts, circuit breakers, and a fallback; load-test to saturation.

## Data & Interface Sketch

```text
 client --> POST /predict {features|entity_id}
                 |
                 v
        +-----------------+   cache miss   +--------------+
        | Feature fetch   |--------------->| Feature store |
        | (Redis cache)   |<---------------|  / DB         |
        +--------+--------+                +--------------+
                 |
                 v
        +-----------------+   window: N reqs or T ms
        | Batching queue  |------------------------+
        +--------+--------+                        |
                 v                                 v
        +-----------------+                +----------------+
        | Optimized model | -- fail -----> | Fallback path  |
        | (ONNX/quantized)|                | cached/default |
        +--------+--------+                +----------------+
                 v
   response { prediction, model_version, latency_ms }

Metrics: p50/p95/p99 latency, RPS, batch size, cache hit ratio
```

## Stretch Goals

- Add a GPU path with TensorRT and compare cost/latency against CPU.
- Support A/B or shadow serving of two model versions with per-version metrics.
- Add adaptive batching that tunes the window based on live load.
- Precompute and warm the cache for the hottest entities on startup.

## Definition of Done

- [ ] p95 and p99 latency are measured under target load and meet the stated budget.
- [ ] The optimized model's accuracy delta versus baseline is documented and acceptable.
- [ ] Dynamic batching and the feature cache are in place with visible hit-ratio metrics.
- [ ] Overload triggers backpressure and the fallback, never an unbounded queue or crash.
- [ ] A load test report shows behavior from normal load through saturation.

## Common Pitfalls

- Optimizing latency on a single request and never testing under concurrent load.
- Batching so aggressively that tail latency balloons for the last request in each window.
- Caching features without a TTL, so the model quietly serves stale inputs.
- Reporting average latency and hiding a terrible p99 behind it.

## Resources

- [ONNX Runtime Documentation](https://onnxruntime.ai/docs/) — cross-framework inference optimization.
- [NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/index.html) — dynamic batching and multi-model serving.
- [TensorFlow Serving Architecture](https://www.tensorflow.org/tfx/serving/architecture) — production serving patterns.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — backpressure and graceful degradation.
