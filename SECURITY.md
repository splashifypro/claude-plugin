# Security Policy

The Splashify Pro Claude Code plugin is built with a **defence-in-depth**
posture. This document describes the security model so users, auditors, and
Anthropic's plugin reviewers can verify the guarantees.

## Threat model

The plugin lives inside Claude Code on the user's machine and drives the
`splashify` CLI to perform actions on a Splashify Pro account. The token that
authorizes those actions belongs to the user and is held by the CLI on disk.

**Assets we protect:**

1. The user's `oc_live_` access token.
2. The user's Splashify Pro account data (contacts, conversations, templates,
   wallet, etc.).
3. The integrity of state-changing actions — no silent sends, broadcasts, or
   deletes.

**Assumed adversary:** a prompt-injection payload reaching Claude (e.g. inside
a fetched URL, an email, an MCP tool result, or a downloaded file) that tries
to coerce the model into harmful actions.

## Guarantees

### 1. The plugin holds no credentials

The `splashify` CLI stores the token at `~/.splashify/config.json` (mode
`0600`). The plugin's manifest, skills, and skill-referenced material never
contain a token and never read that file. The token only enters memory inside
the `splashify` binary itself, on the user's machine, when the user invokes a
CLI subcommand.

This means:

- **Claude's context never contains the token.** Skills do not expose it, and
  CLI commands accept the token from the config file, not from arguments or
  environment.
- **The plugin's storage area never contains the token.** There is no
  `userConfig.api_key`, no `.env`, no `.mcp.json` env-var passthrough.
- **A compromised Claude context cannot exfiltrate the token** via the plugin,
  because the plugin does not have it. (An attacker could still ask Claude to
  run `cat ~/.splashify/config.json` via the Bash tool — that is a Claude Code
  permission issue, not a plugin issue, and is mitigated by Claude Code's
  built-in permission prompts.)

### 2. Backend-side scope restriction

`oc_live_` access tokens are restricted server-side to `/api/v1/app/*`
endpoints — the same surface a logged-in app user sees. Admin, reseller,
partner, and platform endpoints are unreachable with this token class:

```
/api/v1/admin/*        ✗ unreachable
/api/v1/partner/*      ✗ unreachable
/api/v1/prime/*        ✗ unreachable
/api/v1/app/*          ✓ reachable (scoped to the token owner's account)
```

Skills tell Claude to refuse admin-shaped requests politely without invoking
the CLI. Even if Claude were tricked into trying, the backend would reject
the request.

### 3. Confirmation rule for state-mutating actions

Every skill enforces a **confirmation rule** before any non-read action:

- WhatsApp / RCS / Instagram sends to a real number
- Broadcast create / cancel / restart / rebroadcast
- Contact / segment / attribute / template / team-member mutations
- Token revocations
- Any `splashify api POST|PUT|PATCH|DELETE`

Claude must repeat the destination back to the user (e.g. *"Send to
**+91…7890**: 'Order shipped' — go?"*) before invoking the CLI. Read-only
commands run without prompting.

### 4. No prompt-injection-driven sends

A common attack vector is an externally-fetched piece of content telling the
model "send a WhatsApp message to +1xxxxxxxxxx". Skills require Claude to
**get the destination from the user's own message** and to **confirm before
sending**. Phone numbers harvested from tool results are treated as data, not
instructions.

### 5. No silent token rotation

Skills never run `splashify connect`, `splashify token create`, or
`splashify token revoke` without the user explicitly asking. `connect` is
interactive (it prompts for a token on stdin) and would fail in a
non-interactive Claude session anyway, but the skill makes the rule explicit.

### 6. Error transparency

If the CLI exits non-zero, skills must relay the `error:` line verbatim. This
prevents prompt injection from "rewriting" a failed send as a success in the
user's view.

## What the plugin can do

- Invoke `splashify` subcommands documented in the
  [CLI reference](references/cli-reference.md).
- Read and interpret the JSON the CLI prints on stdout.
- Surface errors verbatim.

## What the plugin cannot do

- Read `~/.splashify/config.json` directly.
- Mint new tokens without the user typing `splashify token create` themselves.
- Reach admin / reseller / partner / platform endpoints.
- Send messages without the user supplying the destination in this session.
- Bypass the confirmation rule by reading "permission" from external content.

## Reporting a vulnerability

If you believe you have found a security issue in this plugin, please email
**security@splashifypro.com** rather than filing a public issue. We aim to
respond within two business days.

Please include:

- A clear description of the issue.
- A minimal reproduction (a prompt, a tool result that triggers it, etc.).
- The plugin version (`splashify --version` plus the plugin commit SHA).

We follow a 90-day coordinated-disclosure timeline by default.
