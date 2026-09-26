---
description: Root-cause an agent's failures — cluster them into lineages and name the fix surface (prompt, guardrail, or code). The conversational counterpart to the SDK's `vijil analyze` porcelain verb, over the same `rca-runs` plumbing group.
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

Run **analyze**, the root-cause stage of the Trusted Agent Lifecycle.
Requires the `vijil-mcp` server connected. The SDK ships the same verb:
`vijil analyze <agent> --eval <eval-id>` is the porcelain over its `rca-runs`
plumbing group, and this command is its conversational counterpart. Both say
the same word for the same job.

Invoke the `agent-lifecycle` skill's **analyze** stage against agent `$1`:

1. Take the failing findings from the most recent `/vijil inspect` or
   `/vijil evaluate` (or read them via `eval_results_detail`).
2. Run Vijil's analyzer over the agent (`vijil_analyzer_run_start`, poll
   `vijil_analyzer_run_status`, read `vijil_ticket_list` / `vijil_ticket_get`).
   It clusters findings that share a cause into **lineages** and names each
   lineage's fix surface — the `Category`: `prompt_instructions`, `guardrail`,
   or `code_change`.
3. Report each lineage: the root cause, how many findings it explains, its fix
   surface, and its evidence (the `eval_id` / trace it rests on). Cite the
   evidence; never assert a cause you cannot ground.
4. Point forward by fix surface:
   - `prompt_instructions` / `code_change` → `/vijil adapt <agent>` (a Darwin
     mutation / code fix).
   - `guardrail` → `/vijil protect <agent>` (a Dome control now).

Read only. Analysis measures and explains; it changes nothing. If the MCP is
unreachable, say so and stop.
