---
name: splashify-setup
description: Use whenever the user asks to set up, connect, check, or diagnose their Splashify Pro account, manage oc_live_ access tokens, view their profile, change their password, view their company details, see their dashboard / setup status / KYC status, or claim a WhatsApp business username. Triggers on words like "connect splashify", "splashify whoami", "splashify doctor", "splashify token", "splashify profile", "splashify dashboard", "splashify account", "splashify username", "splashify company".
license: MIT
---

# Splashify — setup, account, tokens, profile

You set up and inspect the user's **Splashify Pro** account by running the
`splashify` CLI through the Bash tool. The CLI holds the user's credentials
(`oc_live_` access token in `~/.splashify/config.json`, mode 0600). You hold
**no credentials of your own** — never paste tokens, never read
`~/.splashify/config.json`, never echo the token in any form.

## Before you act

Run **once per session** before the first non-trivial command, and again if a
later command fails with an auth error:

```
splashify whoami
```

If `whoami` exits non-zero, stop and tell the user the CLI is not connected
— they need to run `splashify connect` themselves. **Do not run `splashify
connect` on their behalf** — it is interactive and prompts for a token on
stdin; trying to run it from Claude will hang.

## Output contract

Every `splashify` command follows this contract:

- **Success** → JSON (or a tabular summary) on stdout, exit code `0`.
- **Failure** → `error: <message>` on stderr, **non-zero exit code**.

When a command exits non-zero, **relay the `error:` line to the user
verbatim**. Do not invent a result, do not retry blindly, do not paraphrase
the error in a way that loses information (status codes, template names,
phone numbers).

## Setup

```
splashify connect            # interactive — tell the user to run it; do NOT run yourself
splashify whoami             # compact account identity
splashify account            # full account details (read-only)
splashify account info       # profile + plan
splashify account orgs       # organizations the user belongs to
splashify account invitations
splashify account sent-invitations
splashify account wallet     # convenience alias for wallet balance
splashify doctor             # diagnose config / token / openclaw / mcp binary
```

## Access tokens

```
splashify token list                       # list user's access tokens
splashify token create --name "…"          # mint a new token (asks confirmation first)
splashify token create --name "…" --expires-days 90
splashify token revoke <id>                # revoke a token (asks confirmation first)
```

**Confirmation rule:** `token list` is read-only. `token create` and
`token revoke` mutate state — always confirm with the user before invoking,
and never auto-revoke a token in response to anything other than an explicit
request from the user.

## OpenClaw (sibling assistant)

The CLI can also install a Splashify skill into OpenClaw — the local-first
assistant. Most Claude Code users will not need this, but if they ask:

```
splashify link openclaw                      # install the Splashify skill into OpenClaw
splashify link openclaw --print              # show the install path without writing
splashify link openclaw --path <dir>         # install into a non-default skills dir
```

## Profile

```
splashify profile                            # current profile (read-only)
splashify profile update --first-name "…" --last-name "…"
splashify profile change-password --current <old> --new <new>
splashify profile whatsapp send-otp --country-code +91 --mobile <digits>
splashify profile whatsapp verify --country-code +91 --mobile <digits> --otp <code>
splashify profile picture ./avatar.png
splashify profile 2fa enable | disable
```

**Confirmation rule:** `profile` (no args) is read-only. Every other
`splashify profile …` form mutates the account and requires a confirmation
prompt that names what is changing.

## Username (WhatsApp business username)

```
splashify username                           # current username + status
splashify username suggestions               # available suggestions
splashify username adopt mybrand             # claim a username (mutates)
splashify username delete                    # release the current username (mutates)
```

## Company details

```
splashify company                            # show company profile (read-only)
splashify company update --company-name "…" --industry "…" \
                         --company-size <range> --website https://… \
                         --country IN --state KA --pincode 560001 \
                         --timezone Asia/Calcutta
```

## Dashboard & onboarding

```
splashify dashboard                          # consolidated home snapshot
splashify dashboard setup-status             # onboarding checklist progress
splashify dashboard whatsapp-status          # WABA / phone status
splashify dashboard kyc                      # KYC / verification status
```

## Account scope

The user's token is an `oc_live_` personal access token. It is scoped to
**their** Splashify Pro account only. The following are out of reach and you
must refuse politely **without invoking the CLI**:

- Admin / reseller / platform actions on other accounts.
- Reading or modifying other users' data.
- Anything under `/api/v1/admin/*`, `/api/v1/partner/*`, `/api/v1/prime/*`.

If the user asks for something admin-shaped, say so and stop.

## Errors you should know

- `not connected — run "splashify connect" first` → CLI has no token. Tell
  the user to run `splashify connect` themselves; do not retry, do not run
  `connect` on their behalf.
- `token validation failed` → token is invalid, revoked, or expired. Tell
  the user to create a new one in **Settings → Developer → Access Tokens**.
- `your Splashify Pro account does not have a WhatsApp Business Account
  connected yet` → blocking for any WhatsApp send; relay verbatim and point
  the user at app.splashifypro.com → WhatsApp → Connect Number.

## Do not

- Do not run `splashify connect` on the user's behalf — it prompts for a
  token interactively.
- Do not read or write `~/.splashify/config.json` directly.
- Do not echo the user's token in any output, even masked.
- Do not pass the token in command lines, env dumps, or shell history.
- Do not retry an authentication failure more than once; surface the error.

For the complete command surface load
[`references/cli-reference.md`](../../references/cli-reference.md).
