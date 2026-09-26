---
description: The full trust evaluation — re-runs the free `/vijil inspect` white-box pass, then adds black-box adversarial evaluation with Vijil Diamond, unified into one R/S/Sa scorecard. Requires the `vijil-mcp` server connected. The plugin's name for the SDK's `vijil evaluate` porcelain verb.
argument-hint: "[<agent-id|name|path>]  (defaults to the agent in the current repo)"
disallowed-tools:
  - mcp__plugin_vijil_vijil-mcp__evolution_run
  - mcp__plugin_vijil_vijil-mcp__dome_config_apply
  - mcp__plugin_vijil_vijil-mcp__dome_config_create
  - mcp__plugin_vijil_vijil-mcp__dome_config_update
  - mcp__plugin_vijil_vijil-mcp__agent_lifecycle
  - mcp__plugin_vijil_vijil-mcp__proposal_approve
  - mcp__plugin_vijil_vijil-mcp__proposal_reject
  - mcp__plugin_vijil_vijil-mcp__eval_delete
  - mcp__plugin_vijil_vijil-mcp__eval_cancel
  - mcp__plugin_vijil_vijil-mcp__agent_archive
---

<!-- Copyright 2026 Vijil, Inc. -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

Run **evaluate** — the connected half of the Trusted Agent Lifecycle. Where
`/vijil inspect` is the free, local, no-account first read, `evaluate` is
what you run once you've connected: it re-runs inspect's white-box pass fresh
(source may have changed) and adds Vijil Diamond's black-box adversarial pass,
reporting both as one unified scorecard.

Invoke the `agent-lifecycle` skill's **evaluate** stage against `$1` (or the
agent in the current repo). It runs two complementary passes:

## White-box (SAST) — same pass as `/vijil inspect`

Read the agent's source and grade it statically against
`references/whitebox-rubric.md` — see `/vijil inspect` for the full method.
Re-run here (not just reused from the last `inspect`) because source can have
changed since. If the `vijil-mcp` server is unreachable, this command still runs this
half and reports the black-box half skipped — it degrades to `inspect`, never
to nothing.

## Black-box (adversarial) — Vijil Diamond, on connect

With the `vijil-mcp` server connected (`VIJIL_CLIENT_ID` / `VIJIL_CLIENT_SECRET`), evaluate the agent's *behavior*
with Diamond, deepening in three tiers:

- **baseline** — the default trust harness (R/S/Sa),
- **bespoke** — a policy-tailored harness for this agent,
- **adaptive** — the multi-turn adversarial swarm.

The free tier is an **account-scoped weekly allowance**, not a lifetime count
of runs. It refills every week and is sized to cover a full turn of the loop
with room to re-test after a fix. The server is the authority on the balance:
it reports what remains and refuses a run that would exceed the ceiling,
returning `429` with `Retry-After`. Surface that number before each run and
relay a refusal plainly — never estimate the balance yourself, and never
retry past a refusal.

## Scope

If the agent is **registered**, scope to its registry record (white-box from
the deposited source; black-box against the registered endpoint). If not,
white-box reads the local repo and black-box runs against a provided endpoint —
registration is never a prerequisite for an evaluation.

## Output + the register moment

Report the **R/S/Sa scorecard**, a verdict — **PASS / REVISE / BLOCK** — and the
failures, white-box and black-box together, each rated `Critical` / `Important` /
`Minor` with its evidence (`file:line`, or `eval_id` / `probe_id` / judge
verdict) and rubric `cell`. One severity ladder and one verdict across both
passes, so a developer reads one language. Empty results mean clean for that
pillar; never invent a failure.

Then offer the artifact the scorecard is evidence for: `eval_report` generates
the evaluation report and `eval_results_report` downloads it as a PDF — an
Executive Summary, the agent's specification, per-harness breakdown, and a
detailed section per trust dimension. This is what a developer hands to
whoever asked for the number. Both are reads; neither mutates the agent.

If this is the **first time** this agent has been evaluated (no prior
`inspect` or `evaluate`) — at peak motivation, when they've just seen the
failures — offer (do not gate): *"Register this agent and these results to
track evaluations, observations, mutations, and redeployments over time?"* →
`/vijil register`. Forward to `/vijil analyze` to root-cause the failures.

Read and measure only. `evaluate` never changes the agent.
