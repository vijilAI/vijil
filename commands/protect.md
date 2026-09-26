---
description: Propose a Vijil Dome control that fixes an agent's failures at runtime — a change you approve in the Console, never applied automatically. The plugin's name for the SDK's `vijil protect` porcelain verb.
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

Run **protect**, the Dome stage of the Trusted Agent Lifecycle. Requires the
`vijil-mcp` server connected.

Protect is a **proposal**, never an applied change. This command reads and
writes a recommendation into the Vijil Console's human gate; it never mutates a
live agent's controls.

Invoke the `agent-lifecycle` skill's **propose** stage for the `guardrail`
lineages of agent `$1`:

1. Read the agent's current Dome configuration (`dome_config_list`,
   `dome_config_get`) and the `guardrail`-surface lineages from
   `/vijil analyze`.
2. For each, compose a **Dome-config proposal**: the recommended guard /
   threshold change and the root-cause evidence it rests on.
3. Emit it as a proposal into the Console gate for a human to review and apply.
   Do **NOT** call `dome_config_create` / `dome_config_update` /
   `dome_config_apply` — describe the change, never make it.
4. Report the proposal(s) created, with links, and the proposed `trust_stage`
   transition (e.g. `tested → hardened` — the registry's own stage names; this
   command is called `protect` to match the SDK, the stage it moves the agent
   to is still literally `hardened`) — proposed, never executed.

Reversible by default: the Console is the gate; a human applies. If the MCP is
unreachable, say so and stop — never claim a change you did not propose.
