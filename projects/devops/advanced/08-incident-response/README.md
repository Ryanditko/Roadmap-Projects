# Incident Response Automation

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build the machinery that turns a raw alert into a coordinated response: detection routes to the right on-call, an incident channel and timeline are created automatically, safe first-line remediations run without waiting for a human, and everything is recorded for a blameless post-mortem. The goal is to shrink the two numbers that define incident pain — time to detect and time to resolve — while keeping a human firmly in control of anything risky. The hard, interesting parts are the judgment calls encoded in software: which remediations are safe to automate, when to escalate versus wait, how to avoid alert fatigue that trains people to ignore the pager, and how to capture a timeline good enough that the post-mortem produces real fixes rather than blame.

## Prerequisites

- An observability stack that produces alerts on SLO or health signals
- A notification/chat platform to integrate (paging tool, chat channels)
- Understanding of on-call, escalation, and severity concepts
- Familiarity with runbooks and the idea of safe, reversible actions

## Learning Objectives

By the end, you should be able to:

- Route alerts to the right responder based on service and severity
- Automate incident creation: channel, timeline, and roles
- Run safe, reversible first-line remediations automatically with guardrails
- Design escalation that reaches a human when automation isn't enough
- Capture a timeline and drive a blameless post-mortem with tracked action items

## Functional Requirements

1. An alert must be classified by severity and routed to the correct on-call responder.
2. Declaring an incident must automatically create a channel, a timeline, and assign roles.
3. Defined safe remediations must run automatically, with a guardrail and an audit record.
4. If automation does not resolve within a threshold, the system must escalate.
5. Every human and automated action must be appended to an incident timeline.
6. Resolution must trigger a post-mortem template pre-filled from the timeline.
7. Post-mortem action items must be tracked to completion.

## Suggested Milestones

1. **Milestone 1 — Detect & route:** Classify alerts by severity and page the correct on-call.
2. **Milestone 2 — Incident orchestration:** Auto-create the channel, timeline, and role assignments on declare.
3. **Milestone 3 — Safe auto-remediation:** Run one reversible remediation with a guardrail and escalation fallback.
4. **Milestone 4 — Learn:** Generate a post-mortem from the timeline and track action items to closure.

## Data & Interface Sketch

```text
   alert fires
      │  classify severity (SEV1..SEV3), map service -> on-call
      ▼
   ┌────────────────┐   page      ┌──────────────┐
   │ Incident        │────────────▶│  On-call     │
   │ orchestrator    │             └──────────────┘
   └───┬───────┬─────┘
       │       │ create channel + timeline + roles (IC, comms, ops)
       │       ▼
       │  ┌──────────────┐
       │  │ Incident      │  append every action, timestamped
       │  │ timeline      │
       │  └──────┬────────┘
       │ safe remediation? (guardrail: reversible, scoped)
       ▼   yes -> run + record ; no/timeout -> escalate
   ┌────────────────┐   resolve   ┌──────────────┐
   │ Auto-remediate │────────────▶│ Post-mortem  │ (blameless, action items)
   └────────────────┘             └──────────────┘

Notification recipients use placeholder identities, e.g. oncall@example.com

Non-functional targets:
  MTTD   time-to-detect measured and trending down
  MTTR   time-to-resolve measured per severity
  alert precision  actionable-alert ratio tracked (fight fatigue)
  automation safety only reversible, scoped actions run unattended
```

## Stretch Goals

- Add auto-detected root-cause hints by correlating the alert with recent deploys and traces.
- Introduce severity-based comms automation (status page updates, stakeholder notifications).
- Add a "practice incident" mode to rehearse the flow without a real outage.
- Track error-budget burn and auto-declare incidents when burn rate is critical.

## Definition of Done

- [ ] Alerts are classified and routed to the correct responder by service and severity.
- [ ] Declaring an incident auto-creates a channel, timeline, and role assignments.
- [ ] At least one safe, reversible remediation runs automatically with a guardrail.
- [ ] Escalation reaches a human when automation fails or times out.
- [ ] A post-mortem is generated from the timeline and its action items are tracked.

## Common Pitfalls

- Automating a remediation that is not reversible, turning a small incident into a big one.
- Alert fatigue: paging on everything until responders mute the channel that matters.
- No timeline discipline, so the post-mortem is guesswork and produces no real fixes.
- Escalation with no timeout, so an unresolved incident sits with automation forever.
- Blameful post-mortems that make people hide detail, defeating the entire point.

## Resources

- [Google SRE Book: Managing Incidents](https://sre.google/sre-book/managing-incidents/) — incident command and coordination.
- [Google SRE Book: Postmortem Culture](https://sre.google/sre-book/postmortem-culture/) — blameless learning done right.
- [PagerDuty Incident Response](https://response.pagerduty.com/) — a practical, open incident response guide.
- [Atlassian: Incident management](https://www.atlassian.com/incident-management) — severity, roles, and process fundamentals.
