---
description: Propose a Vijil Darwin change that durably fixes an agent's failures — a pull request you review and merge, never applied automatically. The plugin's name for the SDK's `vijil adapt` porcelain verb — not `vijil evolve`, which creates a new agent from scratch, a different feature this plugin doesn't expose.
argument-hint: <agent-id|name>
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

Run **adapt**, the Darwin stage of the Trusted Agent Lifecycle. Requires the
`vijil-mcp` server connected.

Adapt is a **proposal**, never an executed evolution. This command produces a
recommended mutation as a pull request for you to review; it never runs an
evolution against a live agent.

Invoke the `agent-lifecycle` skill's **propose** stage for the
`prompt_instructions` / `code_change` lineages of agent `$1`:

1. Take those lineages from `/vijil analyze` and their root-cause evidence.
2. Compose a **Darwin PR proposal**: the recommended system-prompt (or config /
   code) mutation, as text, with the evidence and the expected trust-score
   effect.
3. Emit it as a PR proposal for the agent's repo. Do **NOT** call
   `evolution_run` — describe the mutation, never apply it.
4. Report the proposal(s), with links, and the proposed `trust_stage`
   transition (e.g. `hardened → trusted` once re-evaluated) — proposed, never
   executed.

Reversible by default: the pull request is the gate; you merge. If the MCP is
unreachable, say so and stop — never claim a proposal you did not make.
