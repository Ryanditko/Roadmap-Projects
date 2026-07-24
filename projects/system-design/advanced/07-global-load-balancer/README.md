# Design a Global Load Balancer

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design the traffic-steering layer that sits in front of a globally distributed service and decides, for every incoming request, which datacenter should handle it. A good global load balancer routes users to the nearest healthy region, detects failures within seconds and drains traffic away, respects each region's remaining capacity, and absorbs volumetric DDoS attacks — all before a request reaches an application server. The design revolves around two competing mechanisms — anycast and DNS-based steering — and their very different failover characteristics. This is a design exercise: your deliverable is a written design document with diagrams and trade-off analysis, not running code.

## Prerequisites

- Solid understanding of DNS, TCP/TLS handshakes, and BGP/anycast basics
- Familiarity with health checking and circuit-breaking concepts
- Understanding of latency measurement (RTT, geolocation) and its inaccuracy
- Exposure to DDoS attack classes (volumetric, protocol, application-layer)

## Learning Objectives

By the end, you should be able to:

- Compare anycast vs DNS-based steering and their failover speed trade-offs
- Design a health-check system that avoids both false positives and flapping
- Route on latency and capacity, not just geography, and explain the difference
- Reason about failover time budgets (DNS TTL vs BGP convergence)
- Layer DDoS mitigation ahead of origin capacity

## Requirements & Constraints

1. Route each client to the lowest-latency healthy region within its capacity.
2. Detect a region/PoP failure and drain it within seconds, not minutes.
3. Survive the loss of an entire region without a global outage.
4. Absorb volumetric DDoS at the edge before it reaches origin.
5. Avoid routing flaps: a marginally-unhealthy region must not oscillate in and out.
6. Support weighted/canary steering for gradual rollouts.
7. Keep the steering decision itself highly available and low-latency.

## Suggested Approach

Decide the primary steering mechanism first, because it caps your failover speed. **Anycast** (announce the same IP from many PoPs; BGP routes to the nearest) fails over in seconds but gives you coarse control and risks mid-connection resets on route changes. **DNS-based** steering (resolver returns the best region's IP) gives fine-grained, capacity-aware control but is bounded by DNS TTL and resolver caching — often tens of seconds to minutes. Most real systems combine both: anycast to the edge, DNS/edge logic to steer to origin. Design health checks as active probes plus passive signals, with hysteresis to prevent flapping. Feed capacity metrics into the decision so you shed load before a region saturates. Put DDoS scrubbing at the anycast edge where volumetric floods are absorbed close to the source.

## Architecture Sketch

```text
Client ──DNS resolve──> GeoDNS/Steering svc ──(health + capacity + latency)──> region IP
   │                          ^
   │                          └── Health-check controller (active probes + passive signals)
   ▼
Anycast edge PoP (DDoS scrubbing, TLS termination) ──> nearest healthy region ──> origin

Failover: region unhealthy -> controller marks DRAIN -> steering stops new traffic
          anycast: BGP withdraw (seconds) | DNS: shorten TTL, return alternate (TTL-bound)

Key APIs / control plane:
GET  /resolve?client_ip=...        -> { region, ip, ttl }
POST /health/report {region, rtt, errRate, capacityPct}
POST /steer/weight {region, weight} # canary / gradual shift

Decision inputs:
Region{ id, healthy, capacityPct, measuredRttByGeo }  # hysteresis on healthy flag
```

## Deep-Dive Topics

- **Anycast vs DNS steering:** failover speed, granularity, connection stability trade-offs.
- **Health checking:** active vs passive, hysteresis/flap damping, partial-failure detection.
- **Capacity-aware routing:** shedding load before saturation; avoiding cascading overload.
- **Failover time budget:** DNS TTL and resolver caching vs BGP convergence.
- **DDoS mitigation:** volumetric scrubbing at the edge, anycast dispersion, rate limiting.

## Deliverables

- [ ] A design document (~4–8 pages) with the steering and health-check control plane, refined.
- [ ] A reasoned choice of anycast, DNS, or hybrid, with the failover-time analysis.
- [ ] The health-check and flap-damping mechanism specified.
- [ ] A failure/DR analysis: single PoP loss, full region loss, steering-service outage.
- [ ] A DDoS mitigation section covering volumetric and application-layer attacks.

## Common Pitfalls

- Assuming DNS failover is instant — resolver caches ignore your TTL and serve stale IPs for minutes.
- Health checks with no hysteresis, causing a marginal region to flap in and out of rotation.
- Routing purely on geography, sending traffic to the nearest region even when it is overloaded.
- Terminating everything at one scrubbing center, making it the DDoS target itself.
- Forgetting that anycast route changes can reset in-flight TCP connections.

## Resources

- [Cloudflare: What is Anycast?](https://www.cloudflare.com/learning/cdn/glossary/anycast-network/) — anycast steering and DDoS dispersion explained.
- [Google Maglev paper](https://research.google/pubs/pub44824/) — a fast, reliable software network load balancer.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — load balancing and availability patterns.
- [AWS: Global Accelerator / Route 53 routing policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html) — real geo/latency/weighted routing options.
- [RFC 1035: Domain Names](https://datatracker.ietf.org/doc/html/rfc1035) — DNS, TTLs, and resolution behavior.
