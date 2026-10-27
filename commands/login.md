---
description: Sign in to Vijil from your browser — OAuth2 PKCE against the Vijil Console. Writes a session the plugin reads, so you never copy an API key by hand. An alternative rather than a prerequisite, since VIJIL_CLIENT_ID and VIJIL_CLIENT_SECRET also work and are the better choice for CI.
argument-hint: "(no arguments)"
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

Run **login** — an interactive browser sign-in that connects this plugin to
Vijil. One sign-in, no key copied by hand.

It is not the only way to connect, and not required for `/vijil inspect`,
which is free, local, and needs no account at all. For CI or a headless
box, `VIJIL_CLIENT_ID` + `VIJIL_CLIENT_SECRET` remain the better path — a
revocable key pair beats an interactive flow no one is present to complete.

**Ran `auth init --url` or `login-browser` and nothing changed?** Those
write to disk, and every `vijil-mcp` tool call shells out fresh and rereads
`~/.vijil/config.yaml` / `credentials.json` — the next call already sees
it, no reconnect needed.

**Exported `VIJIL_CONSOLE_URL`, `VIJIL_CLIENT_ID`, or `VIJIL_CLIENT_SECRET`
in your shell and nothing changed?** That's different: the MCP server's
process inherited the environment only at spawn, so a later `export` never
reaches it — editing the variable again will not help. Run `/mcp`, pick
`vijil-mcp`, and reconnect it, then retry. `uvx vijil-console auth status`
reports the credential source, the URL in use, and where that URL came
from — run it first if you're not sure what's actually configured.

## Steps

1. **Run the sign-in, in the foreground.**

   ```bash
   uvx vijil-console auth login-browser
   ```

   Nothing needs installing first — `uvx` fetches the package on demand, the
   same way the plugin's own MCP server starts. **Do not background this
   call.** It blocks while it waits for the browser round-trip, and that wait
   is the command working, not hanging.

   What it does: opens the default browser to the Vijil Console itself, where
   the user signs in with their email and password and approves a consent
   screen naming this tool and machine. The command catches the redirect on
   a localhost server, exchanges the auth code for a Vijil JWT, and writes it
   to `~/.vijil/credentials.json` and `~/.vijil/config.yaml`. No third-party
   identity provider is involved anywhere in the flow. The plugin's MCP
   server reads that session automatically — `VIJIL_API_KEY` takes precedence
   when set, and a browser session is used when it is not.

2. **If the browser cannot open**, the command prints a URL instead. Surface
   that URL to the user verbatim. A headless terminal is not a failure.

3. **Report what actually happened.** On success, say they are signed in and
   can run `/vijil evaluate`. On timeout or failure, show the error the
   command printed. Never report a sign-in you did not see succeed — an
   agent that claims a session it does not have sends the user into a 401
   one step later, and even now that it names the URL and credential
   source it is still a wasted, avoidable step.

## Boundary

This command's only writes are local: `~/.vijil/credentials.json` and
`~/.vijil/config.yaml` on the caller's own machine. It touches no agent, no
platform configuration, and no record — nothing changes on the Vijil Console
side. If the sign-in fails, stop and say so — do not fall back to asking for
an API key unless the user asks for that path.
