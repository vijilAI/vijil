<!-- Copyright 2026 Vijil, Inc. -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# The loop and the seven invariants

The Trusted Agent Lifecycle is one control loop, run over your agents. It is the
same primitive Vijil's platform runs internally — measure, reason, act, evaluate,
learn — pointed here at the agents you build.

## The six beats

| Beat | Here |
|---|---|
| **Perceive** | evaluate the agent — a white-box static pass over its source + black-box Diamond scores + Dome runtime signals |
| **Reason** | cluster failures into root-cause lineages; name the fix surface |
| **Act** | propose the fix (Dome control, Darwin PR, stage transition) into a human gate |
| **Evaluate** | grade the prior run (were proposals acted on?); report this run honestly |
| **Learn** | append routed lineages to the ledger, keyed by a recurrence signature |
| **Adapt** | when a signature recurs past threshold, encode a durable change (a default, a rubric) |

Evaluate runs twice per turn of the loop — at the front, grading the previous
run before perceiving anew (the "grade-before-act" phase offset), and at the
back, reporting this run's output.

## The seven invariants

Every stage is bound by all seven:

1. **Grade before you act.** Grade the previous run before starting the next.
2. **Evidence, or drop it.** Every finding, score, and proposal carries its
   source (`eval_id` / `probe_id` / `trace_id`). Unverifiable → dropped. Empty
   results mean the agent is clean, never "invent a finding."
3. **Decide with weight.** A signal that changes no decision is noise; set it aside.
4. **Reversible by default.** The loop takes only reads and measurements. Every
   irreversible act — mutate an agent, apply a config, run an evolution, approve
   a proposal — is emitted as a proposal into a human gate, never executed.
5. **Write it down.** Every run writes a dated proposal-list and a per-agent
   state file where the next run can read them.
6. **Learn, then encode.** Routed lineages accumulate in a ledger; a recurring
   signature becomes a durable change, not a repeated finding.
7. **Close honestly.** Report exactly what was done. Name every system that was
   unreachable. Never claim a write or a score you did not obtain.

Invariant 4 is the one a developer feels first: **Vijil reads and proposes; you
approve.** Nothing in this plugin changes a live agent.
