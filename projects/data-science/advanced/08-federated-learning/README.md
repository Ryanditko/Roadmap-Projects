# Federated Learning Simulation

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Simulate a system where many clients train a shared model together without ever sending their raw data to a central server. Each client trains locally on its own data, sends only model updates, and a server aggregates them into a new global model. It sounds simple until the realities hit: clients hold non-IID data, some are slow or drop out, communication is expensive, and the updates themselves can leak information. This project has you implement the FedAvg loop, then confront heterogeneity, stragglers, communication cost, and privacy — the four things that separate a toy from a real federated system.

## Prerequisites

- Solid understanding of gradient-based training and model averaging
- Experience with PyTorch or TensorFlow for the local training step
- Familiarity with the concept of differential privacy
- Comfort simulating distributed processes (threads, processes, or a framework)

## Learning Objectives

By the end, you should be able to:

- Implement the federated averaging (FedAvg) training loop across simulated clients
- Handle non-IID data partitions and measure their effect on convergence
- Deal with stragglers and client dropout without stalling a round
- Reduce communication cost via update compression or fewer rounds
- Add a privacy mechanism (differential privacy or secure aggregation) and quantify its cost

## Functional Requirements

1. The simulation must run multiple clients, each training locally on a private data partition.
2. A server must aggregate client updates into a global model each communication round.
3. Data partitions must support both IID and non-IID distributions across clients.
4. The system must tolerate stragglers and dropped clients within a round.
5. It must implement a communication-reduction technique (compression or local epochs).
6. It must offer a privacy mechanism (e.g. DP-SGD) that can be toggled and measured.
7. It must track and report global-model convergence across rounds.

## Non-Functional Requirements

- **Convergence:** the global model must converge under non-IID data within a documented round budget.
- **Communication efficiency:** report bytes-per-round and total communication versus a centralized baseline.
- **Privacy:** when DP is enabled, report the privacy budget (epsilon) and its accuracy cost.
- **Robustness:** a round must complete even if a configurable fraction of clients drop out.

## Suggested Milestones

1. **Milestone 1 — FedAvg core:** Simulate clients, local training, and server averaging on IID data.
2. **Milestone 2 — Heterogeneity:** Introduce non-IID partitions and measure the convergence hit.
3. **Milestone 3 — Robustness & communication:** Handle stragglers/dropout and add update compression.
4. **Milestone 4 — Privacy:** Add DP-SGD, sweep epsilon, and quantify the privacy–accuracy tradeoff.

## Data & Interface Sketch

```text
              +-------------------- Server --------------------+
              |  global model w_t                              |
              |    broadcast w_t                               |
              +-----+-----------+-----------+------------------+
                    |           |           |
             +------v--+  +-----v---+  +----v----+  ... (K clients)
             | Client1 |  | Client2 |  | ClientK |
             | local   |  | local   |  | local   |   private data (IID or non-IID)
             | train   |  | train   |  | train   |
             | -> dw_1 |  | -> dw_2 |  | -> dw_K |   (+ DP noise, compression)
             +----+----+  +----+----+  +----+----+
                  |            |            |
                  +------ aggregate (weighted avg) ------+
                                |
                          w_{t+1} = sum_k (n_k/n) * (w_t + dw_k)

 Round drops clients past a deadline (straggler handling)
 Track: global acc per round, bytes/round, epsilon (if DP on)
```

## Stretch Goals

- Add secure aggregation so the server never sees individual updates in the clear.
- Add personalization: a shared base with per-client fine-tuned heads.
- Simulate an adversarial client (model poisoning) and add a robust aggregator (e.g. trimmed mean).
- Compare FedAvg against FedProx under strong heterogeneity.

## Definition of Done

- [ ] FedAvg converges on IID data across simulated clients.
- [ ] Non-IID partitions are supported and their convergence impact is measured.
- [ ] Rounds complete despite a configurable fraction of straggling/dropped clients.
- [ ] A communication-reduction technique is implemented and its savings reported.
- [ ] DP can be enabled, with epsilon and the accuracy cost reported.

## Common Pitfalls

- Testing only on IID data and never seeing the convergence problems that define federated learning.
- Waiting synchronously for every client, so one straggler stalls the whole round.
- Adding differential privacy without tracking the actual epsilon, so "private" is meaningless.
- Averaging updates unweighted when clients hold very different amounts of data.

## Resources

- [Communication-Efficient Learning of Deep Networks from Decentralized Data (McMahan et al., 2017)](https://arxiv.org/abs/1602.05629) — the FedAvg paper.
- [Advances and Open Problems in Federated Learning (Kairouz et al., 2019)](https://arxiv.org/abs/1912.04977) — the comprehensive survey.
- [TensorFlow Federated Documentation](https://www.tensorflow.org/federated) — a framework for federated computation.
- [Deep Learning with Differential Privacy (Abadi et al., 2016)](https://arxiv.org/abs/1607.00133) — DP-SGD and the privacy budget.
