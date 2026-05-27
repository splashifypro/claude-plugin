---
name: splashify-account-admin
description: Use whenever the user asks to manage their Splashify Pro team / agents / members (invite, role, permissions, OTP, delete), WhatsApp Business Account (WABA setup, registration, profile, sync, OBA), opt-in / opt-out keywords and responses, IP allowlist for the API, active login sessions and devices (logout, logout-all), or support tickets (open, reply, close, list). Triggers on "team member", "add agent", "invite member", "WABA", "register WhatsApp number", "opt-out keyword", "STOP keyword", "allowed IPs", "API allowlist", "logout session", "support ticket".
license: MIT
---

# Splashify — team, WABA, opt management, allowed IPs, devices, support

You manage the user's account-admin surface on **Splashify Pro** by
running the `splashify` CLI through the Bash tool.

## Output contract

- Success → JSON on stdout, exit code `0`.
- Failure → `error: <message>` on stderr, non-zero exit code.

Relay errors verbatim.

## Confirmation rule

**Read-only** (`team`, `member <id>`, `waba`, `waba setup-status`,
`waba oba-status`, `opt`, `allowed-ips`, `devices`, `session <id>`,
`support`, `ticket <id>`) → no confirmation.

**Ask first** before:

- `team add | verify | resend-otp | resend-invite | update | set-role |
  set-permissions | delete` — team mutations affect who can act on the
  account; confirm member name + role + permission set in the prompt.
- `waba sync | update | register-phone | oba-apply | request-deletion` —
  WABA mutations propagate to Meta and can be slow or irreversible.
- `opt out add | remove | response | response-on | response-off` and the
  symmetrical `opt in` forms.
- `allowed-ips add | delete` — locking yourself out of the API is a real
  risk; confirm the IP / range and warn if the current public IP is not
  in the list.
- `devices logout <id> | logout-all` — kills active sessions; confirm.
- `ticket create | reply | close` — relay back to the support team;
  confirm before sending.

## Team / agents

```
splashify team                               # list members
splashify member <id>                        # one member
splashify team add --name --email --country-code +91 --phone <digits> \
                   [--role agent|manager] \
                   [--all read_write | --permissions '{…}' \
                    | --rw messages,contacts --read analytics --none wallet] \
                   [--otp <code> | --no-verify]
splashify team verify <member_id> --otp <code>
splashify team resend-otp <member_id>
splashify team resend-invite <member_id>
splashify team update <member_id> [--role] [--all|--permissions|--rw|--read|--none]
splashify team set-role <member_id> agent | manager
splashify team set-permissions <member_id> […]
splashify team delete <member_id>
```

**Roles:** `agent` (frontline), `manager` (full account).

**Permission DSL** — `--rw modules`, `--read modules`, `--none modules`
where modules are: `messages`, `contacts`, `broadcasts`, `templates`,
`segments`, `analytics`, `wallet`, `team`, `integrations`, `ai-agents`,
`flows`, `tickets`, `expenses`, `email`, `calling`, `waba`, `tags`,
`canned`, `instagram`, `ctwa`. Use `--all read_write` as a shortcut for
"everything read-write".

When the user describes a permission set in English, translate to the
DSL and **show the translated flags in the confirmation prompt** so they
can correct you before you invoke.

## WABA (WhatsApp Business Account)

```
splashify waba                               # full WABA details
splashify waba sync                          # refresh data from Meta
splashify waba update --about "…" --description "…" --email "…" \
                     --address "…" --vertical RETAIL --websites https://…
splashify waba register-phone                # register a new number with Meta
splashify waba setup-status                  # onboarding checklist progress
splashify waba oba-status                    # Official Business Account status
splashify waba oba-apply                     # apply for the green tick
splashify waba request-deletion              # destructive; cannot be undone
```

`waba request-deletion` is irreversible. **Refuse the first time, summarise
what will be lost, and require a second explicit confirmation** from the
user before invoking.

## Opt management

```
splashify opt                                # show full opt settings
splashify opt out | in                       # one side
splashify opt out add STOP UNSUBSCRIBE       # add keywords
splashify opt out remove STOP                # remove keywords
splashify opt out response "<text>"          # set auto-response body
splashify opt out response-on | response-off # toggle auto-response
splashify opt in …                           # same actions on the in side
```

Opt-out compliance is regulatory — surface the current state before
mutating, and never disable opt-out auto-response without confirming.

## IP allowlist (for API access)

```
splashify allowed-ips                        # list entries
splashify allowed-ips add --name "Office" --mode single --ip 203.0.113.4
splashify allowed-ips add --name "VPN" --mode range \
                          --start 10.0.0.0 --end 10.0.0.255
splashify allowed-ips delete <entry_id>
```

If the user adds their first IP rule, **warn them** that they may lock
themselves out: future API calls from any other IP will fail. Suggest
they double-check their current public IP before invoking.

## Devices / sessions

```
splashify devices                            # list active login sessions
splashify devices list [--platform web|android|ios|mobile] [--ip <addr>]
splashify session <session_id>
splashify devices logout <session_id>        # kill one session
splashify devices logout-all [--yes] [--platform …] [--ip …]
```

`devices logout-all` kills every other session, including possibly the
user's mobile app — confirm the platform filter (e.g. "all web sessions
only") in the prompt.

## Support tickets

```
splashify support                            # list tickets
splashify support list [--status open|in_progress|waiting_for_user|resolved|closed] \
                       [--search "…"]
splashify ticket <id>                        # one ticket + full thread
splashify ticket create --title "…" --description "…" \
                        --category bug|billing|feature_request|account|general \
                        [--priority low|medium|high|urgent]
splashify ticket <id> reply "<message>"
splashify ticket <id> close
```

For `ticket create`, summarise the user's complaint into a clear title +
description and **show the draft to the user** before invoking. Categorise
carefully — `billing` and `bug` tickets are routed to different teams.

## Do not

- Do not invent member emails, phone numbers, or session IDs.
- Do not change someone's permissions without summarising the diff
  (current vs new) in the confirmation prompt.
- Do not run `waba request-deletion` without a two-step confirmation.
- Do not run `allowed-ips add` without warning about lockout risk.
- Do not close someone's ticket while they are still waiting on a reply
  unless they explicitly asked.

For commands not covered above load
[`references/cli-reference.md`](../../references/cli-reference.md).
