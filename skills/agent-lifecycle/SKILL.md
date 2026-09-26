---
name: agent-lifecycle
description: The Trusted Agent Lifecycle — inspect the agent a developer is building (free, local, no-account white-box static pass), evaluate it fully once connected (inspect re-run + black-box adversarial with Diamond), root-cause failures, and propose fixes via Dome (protect) or Darwin (adapt). Registration is offered after the first inspect or evaluation, not required before it; the registry is the spine that tracks every eval, observation, mutation, and redeploy. Reversible — reads/measures + first-party record writes only — never mutates a live agent. Invoked by the /vijil commands (inspect, evaluate, analyze, protect, adapt, register) — inspect is plugin-native (no SDK equivalent), the rest named to match the SDK's porcelain CLI vocabulary.
---

<!-- Copyright 2026 Vijil, Inc. -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# The Trusted Agent Lifecycle

This skill is the engine behind the `/vijil` commands. Its job is the developer's
question — *is the agent I'm building trustworthy, and where does it fail?* — not
the risk owner's (finding shadow AI across an enterprise; that's Vijil Discover).

`inspect` is the actual ungated front door — free, local, no MCP, no account.
It's a standalone read, not a step of the control loop below; it exists so a
developer gets value before connecting anything. The loop itself, with
`evaluate` as its first connected step and the **registry** as the spine
everything reads and writes:

```
grade last run  →  evaluate (white+black)  →  analyze  →  propose(protect|adapt)  →  close  →  learn
  (Evaluate′)          (Perceive)              (Reason)            (Act)              (Evaluate) (Learn)
                                       ⇅  the registry record  ⇕
```

Two words are doing double duty here, in different registers. `evaluate` and
`adapt` are this plugin's **command** names (matching the SDK's porcelain
verbs). `Evaluate` and `Adapt` — capitalized, in parens — are two of the six
generic **beat** names below (`references/loop.md`), a control-loop vocabulary
this skill shares with Vijil's own internal platform and unrelated to the SDK
CLI. A step header names the command it runs; its parenthetical names the beat
it fulfills — they are not always the same word for the same reason.

`register` is the on-ramp, offered after the first `inspect` or `evaluate` —
it deposits the agent into the registry so its trust history persists. See
`references/loop.md` for the six beats and seven invariants.

## The registry — the spine and the one schema

The registry record is the canonical **governed agent**. Every door writes the
same shape: a developer's **manual register**, a repo scan, and shadow's
**discovery** all conform to **one registry schema** — so a built agent and a
discovered one are the same kind of record. Once an agent is registered, all
six verbs scope to its record, and it becomes visible to the governance side.
(Schema contract: the unified registry schema; do not invent per-command shapes.)

## Reversible boundary — refined for a builder's own agent

- **Reads / measurements** (inspect, evaluate, analyze): always allowed.
- **First-party record writes** (register): allowed — the developer is creating
  or updating the record for *their own* agent (`agent_create` / `agent_update`).
  This writes the registry, not the agent's runtime; it is reversible (a record).
- **Live-agent mutations** (apply a Dome config, run an evolution, transition a
  trust stage on an agent you're auditing): **never executed.** Emitted as
  **proposals** into a human gate.

## HARD-GATE — refuse and stop if:

1. **The `vijil-mcp` server is unreachable** for a step that needs it. For
   `evaluate`, run the white-box pass anyway and report the black-box pass
   skipped — it degrades to `inspect`, never to nothing. `inspect` itself
   never needs the MCP server in the first place; never fabricate a score.
2. **A live-agent mutation is implied.** `protect` / `adapt` / stage transitions
   are proposals, never applied. `register` is the *only* write, and only to the
   caller's own agent record.
3. **A finding cannot be verified** against its source (`file:line`, `eval_id`,
   `probe_id`, `trace_id`). Unverifiable → dropped. Empty results mean clean,
   never "invent a finding."

### Vijil MCP tools

- **Read / measure (reversible):** `agent_list`, `agent_get`, `agent_eval_config`,
  `agent_dome_configs`, `dome_detect_status`, `dome_config_list`, `dome_config_get`,
  `telemetry_*`, `proposal_list`, `proposal_get`; **Diamond** `eval_run`
  (within the free/paid budget), `eval_status`, `eval_results_detail`,
  `eval_summary_by_agent`, `eval_report` (generate the evaluation report),
  `eval_results_report` (download it as PDF), `eval_redteam_report` /
  `_md` / `_pdf` for adaptive runs; **analyzer** `vijil_analyzer_run_*`,
  `vijil_ticket_*`.
- **First-party record write (register only):** `agent_create`, `agent_update`
  — for the caller's own agent, to create/attach its registry record + results.
- **Forbidden (the human gate):** `dome_config_apply` / `create` / `update`,
  `evolution_run`, `agent_lifecycle` (stage transition on a third-party agent),
  `proposal_approve` / `reject`, `eval_delete` / `cancel`, `agent_archive`. If a
  finding implies one, ROUTE it as a proposal.

When polling `eval_status`, its terminal states are **`saved`** (success — scores
and `eval_results_detail` are ready) and `failed`; it never reports `completed`.
Poll for `saved` / `failed`, not `completed` / `failed`, or the loop hangs.

## The steps

### 0 — INSPECT (standalone, not a loop beat)
The white-box half of step 2 below, run alone: read the agent's source and
grade it against `references/whitebox-rubric.md`. Same method, same finding
schema, same `file:line`-or-drop rule — the only difference is there's no
loop around it (no grade-before-act, no registry record required, no MCP).
`evaluate` re-runs this exact pass as its own white-box half, fresh, every
time — never reuses a stale `inspect` result.

### 1 — GRADE last run (Evaluate′)
If the agent is registered, read its record: prior evals, open proposals,
`action_rate = (approved + applied) / proposed`, and any lineage open ≥2 runs.
First run of an unregistered agent: nothing to grade yet. Grade-before-act.

### 2 — EVALUATE (Perceive) — white-box + black-box
- **White-box (SAST, free/local):** read the agent's source (registry record if
  registered, else the local repo) and grade it against
  `references/whitebox-rubric.md` — the R/S/Sa construction rubric specialized to
  an agent. Run the statically-decidable cells only: reliability (tool/model
  output trusted blindly, swallowed tool errors, unbounded loop, unvalidated
  config), security (the OWASP boundary checklist — injection surface, tool
  grants, secrets, deserialization, SSRF), and the assertive **safety rung-1**
  presence check (no declared contract → always a `Minor`). Each finding quotes
  a `file:line` and tags its `cell`.
- **Black-box (Diamond, on connect):** evaluate behavior — `baseline` → `bespoke`
  → `adaptive` (the multi-turn swarm). This owns the behavioral cells the
  white-box pass cannot decide (correctness, consistency, safety correctness).
  Free tier = an **account-scoped weekly allowance**, not a lifetime count. The
  server is the authority on the balance: surface what it reports, relay a `429`
  plainly, never estimate it yourself.
- **Reaching the endpoint (AgentCore).** Which protocol the runtime was created
  with decides how Vijil connects. An **A2A** runtime publishes an agent card
  Vijil discovers. An **HTTP** runtime does not: AgentCore answers a card
  request with `GetAgentCard is only supported for A2A agents`. That refusal is
  the correct answer, not a transient error — do not retry it, and do not treat
  it as the agent being unreachable. Point Vijil at whatever already fronts the
  runtime (the API Gateway stage or Lambda URL exposing it as an
  OpenAI-compatible endpoint) and pass `protocol: "chat_completions"` to
  `agent_create`. Pass `"a2a"` only for a runtime that really publishes a card.
  Never guess between the two — ask, because the wrong one registers an agent
  that no later evaluation can reach.
One taxonomy, two lenses: a white-box finding is a hypothesis the black-box pass
confirms. Emit `Finding{dimension, severity: Critical|Important|Minor, cell,
location, evidence, predicts_cell}`; the pass returns a **PASS / REVISE / BLOCK**
verdict.

### 3 — ANALYZE (Reason)
Hand findings to the analyzer (`vijil_analyzer_run_*`). Cluster into root-cause
**lineages**; name each lineage's fix surface (`prompt_instructions` /
`guardrail` / `code_change`). Deduplicate against the registry record's open
lineages. Keep only NEW ones at/above severity.

### 4 — PROPOSE (Act — proposals only) / and the register moment
For each lineage emit the gate proposal — Darwin PR (`prompt_instructions`),
Console Dome-config (`guardrail`), or tracker issue (`code_change`) — described,
never applied. **After the first `inspect` or `evaluate`**, offer register
(never gate): "Register this agent and these results to track them over
time?" → `register` deposits the record via `agent_create`/`agent_update`.

### 5 — CLOSE (Evaluate)
Report the scorecard, the failures, the proposals created (IDs/links), and — if
registered — the record id + `trust_stage`. Name any unreachable system; never
claim a write or score you did not make.

### 6 — LEARN
Append routed lineages to the registry record keyed by `signature`. A weakness
recurring across the agent's runs (or, on the governance side, across agents) is
systemic — encode a durable change. Learn emits; it does not decide.

## Cross-links

- `references/loop.md` — the six beats and seven invariants.
- `references/whitebox-rubric.md` — the R/S/Sa static rubric (`inspect`, and `evaluate`'s white-box half).
- `sweeps/trust.md` — the trust sweep (white-box static + Diamond/Dome black-box).
- `cycles/trust.example.yaml` — a scheduled-run config.
