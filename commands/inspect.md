---
description: Free, local, no-account static trust read of the agent in your repo — the actual front door, works with zero setup. Plugin-native; no SDK porcelain equivalent (distinct from vijil-console's own `scan` command group, which discovers infrastructure, not source).
argument-hint: "[<path>]  (defaults to the agent in the current repo)"
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

Run **inspect** — the zero-friction first read. It answers the one question a
developer can't read off their own source: *is the agent I'm building
trustworthy, and where does it fail?* **Fully ungated**: no `vijil-mcp` server
connection, no account, no key. This is rung one of the Trusted Agent
Lifecycle — the thing to run before you've connected anything.

Invoke the `agent-lifecycle` skill's **inspect** stage against `$1` (or the
agent in the current repo).

## White-box (SAST)

Read the agent's source in the repo — system prompt, tool grants, the agent
loop, config, output handling — and grade it **statically** against
`references/whitebox-rubric.md`: the same Reliability / Security / Safety
taxonomy Vijil holds its own code to at construction, specialized to an agent.
It runs the statically-decidable cells — the reliability cells (tool/model
output trusted blindly, swallowed tool errors, unbounded agent loop, unvalidated
config), the OWASP boundary checklist (prompt-injection surface, over-broad tool
grants, secrets in the prompt, unsafe deserialization, SSRF), and the assertive
**safety rung-1** presence check (if the agent declares *no* safety contract at
all, always raise a `Minor` — the on-ramp to declaring one). Every finding
quotes a `file:line` (no quote, no finding) and tags its rubric `cell`, so it
lands in the same coordinate system as `/vijil evaluate`'s black-box pass. Read
a finding here as a *hypothesis*; the connected swarm confirms or refutes it.

## Output + what's next

Report the **R/S/Sa scorecard** (the pillars this static pass can actually
decide — some cells are behavioral and can't be settled without connecting),
a verdict — **PASS / REVISE / BLOCK** — and the failures, each rated
`Critical` / `Important` / `Minor` with its `file:line` evidence and rubric
`cell`. Empty results mean clean for that pillar; never invent a failure.

Then, at the end of the **first** inspect — at peak motivation, when they've
just seen the failures — offer (do not gate) both next steps: *"Connect and
run the full adversarial evaluation with `/vijil evaluate <agent>`? Or register
this agent to track it over time with `/vijil register`?"* Neither is required;
this command is complete on its own.

Read only. `inspect` never changes the agent and never touches the network —
it's a local static read, full stop.
