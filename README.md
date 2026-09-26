<!-- Copyright 2026 Vijil, Inc. -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# Vijil — the Trusted Agent Lifecycle for Claude Code

**Evaluate** the agent you're building. Make it trustworthy. **Track** it.

You have the agent — the code, the prompt, the tools. What you don't have is the
trust verdict: *is it reliable, secure, safe, and where does it fail?* Vijil
tells you, fixes it, and keeps watch across every change. It reads and measures
your agent and **proposes** fixes into a gate; it never changes a live agent.

## Quickstart — three rungs, each earned by value, none gated

**1. A trust read in your editor, free, no account:**

```
/vijil inspect
```

A white-box static pass over the agent in your repo — injection surface,
over-broad tools, missing output validation, secrets in the prompt. Runs
locally, no key, no MCP connection — nothing to set up.

**2. The full adversarial evaluation** (connect; your account is free, with a
weekly allowance sized to run a full turn of the loop):

```bash
export VIJIL_CLIENT_ID=vk_...        # from https://console.vijil.ai → Settings → API Keys
export VIJIL_CLIENT_SECRET=...       # shown once, at creation — copy it then
```
```
/vijil evaluate <agent>   # re-runs inspect + Diamond: baseline → bespoke → adaptive
/vijil analyze <agent>    # root-cause the failures → prompt / guardrail / code
/vijil protect <agent>    # propose a Dome control (you approve in the Console)
/vijil adapt <agent>      # propose a Darwin PR (you review and merge)
```

**3. Keep and track it:**

```
/vijil register           # the agent's governed record — every eval, observation, mutation, redeploy
```

Registration is offered right after your first `inspect` or `evaluate` —
when you've just seen the failures — never demanded before it.

## The verbs

`evaluate`/`protect`/`adapt`/`register` are named to match the SDK's own
porcelain CLI vocabulary, so the plugin and the CLI describe the same
lifecycle in the same words. `analyze` matches the SDK's root-cause
porcelain verb, which sits over the same `rca-runs` plumbing group.
`inspect` matches `vijil inspect`, the SDK's deterministic white-box pass.
(`inspect` is also deliberately not called "scan" —
`vijil-console` already has its own `scan` command group, for discovering
infrastructure, not reading source — reusing the word would have meant two
different things by the same name across the same product.)

| Verb | What it does | Powered by |
|---|---|---|
| **inspect** | White-box static pass (SAST-style), free, local, no account — the actual front door | — |
| **evaluate** | Re-runs `inspect` fresh **+** black-box adversarial (baseline → bespoke → adaptive) | **Diamond** |
| **analyze** | Clusters failures into root-cause lineages; names the fix surface | Vijil analyzer |
| **protect** | Proposes a runtime control for a `guardrail` failure | **Dome** |
| **adapt** | Proposes a durable mutation (prompt / code) as a pull request | **Darwin** |
| **register** | Creates the agent's governed record — tracks its trust across every change | the Vijil registry |

The weekly allowance is sized for one full lifecycle turn: **baseline,
bespoke, adaptive, post-protect, post-adapt** — a turn of the loop, and room
to re-test after you fix something.

## White-box **and** black-box

Most agent evaluation is black-box only, because most tools can only *call* an
agent, not *read* it. Vijil does both: a **static** pass over the source (the
injection surface, the tool grants, the unsafe patterns — `inspect`, and again
inside `evaluate`) **and** an **adversarial** pass against behavior (the swarm,
`evaluate` only). Registration deposits the source into the registry, so the
white-box view persists and every future evaluation is scoped to it.

## The safety boundary is structural

`inspect`, `evaluate`, and `analyze` are reads and measurements. `register`
writes a *record* (your own agent — a first-party action, reversible). The
fixes — `protect`, `adapt` — are **proposals into a human gate**: a
Dome-config change you approve in the Console, a Darwin pull request you
merge. The plugin is *unable* to mutate a live agent, apply a config, or run
an evolution.

## Install

```
/plugin marketplace add vijilAI/vijil
/plugin install vijil@vijil
```

## Discovery is a different job

Finding shadow AI you *didn't* build — across an enterprise, for auditors and
GRC — is **Vijil Discover**, a separate product for risk owners (dashboards,
alerts, fleet reports). This plugin is for the developer building and fixing a
specific agent. Both write the same registry record; the developer registers
theirs, Discover finds the unregistered ones.

## License

Apache-2.0.
