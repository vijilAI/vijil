---
description: Register the agent you're building into the Vijil registry — a governed record that tracks its evaluations, observations, mutations, and redeployments over time.
argument-hint: "[<agent-id|name>]  (defaults to the agent in the current repo)"
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

Run **register**, the on-ramp of the Trusted Agent Lifecycle — the same verb
name the SDK's porcelain CLI uses (`vijil register`). It gives the agent you're
building a **governed record** in the Vijil registry — the one place its whole
trust history lives.

Register is a **first-party** action — you're creating a record for *your own*
agent — so unlike `protect`/`adapt` it is an allowed write, not a proposal. It
creates a record; it does **not** mutate a live agent's behavior, config, or
deployment.

Invoke the `agent-lifecycle` skill's **register** stage against `$1` (or the
agent detected in the current repo):

1. **Read the white-box view** from the repo — system prompt, tools, config,
   and code paths — the same source `inspect` and `evaluate` use for their
   static pass.
2. **Create or update the registry record** via the backend registry API
   (`agent_create` for a new agent, `agent_update` to attach to an existing
   one). Deposit the white-box view and, if an `inspect` or `evaluate` just
   ran, its results. The record conforms to the **one canonical registry
   schema** shared across manual register, repo scan, and shadow discovery —
   so a developer's agent and a discovered one are the *same shape* of record.
   When the record carries an endpoint, set `protocol` alongside the URL —
   `chat_completions` for an OpenAI-compatible endpoint (including an AgentCore
   HTTP runtime behind API Gateway), `a2a` for a runtime that publishes an agent
   card. Ask rather than guess: the wrong value registers an agent no later
   evaluation can reach, and nothing downstream points back at the cause.
3. **Report** the registry id and the agent's `trust_stage` (now `registered`),
   and confirm what's now tracked: every future evaluation, observation,
   mutation, and redeployment against this agent.

If the `vijil-mcp` server is unreachable, say so and stop — never claim a
registration you did not make. Once registered, `inspect` / `evaluate` /
`analyze` / `protect` / `adapt` all scope to this record, and the agent is
visible to the governance side (the registry is where the two personas meet).
