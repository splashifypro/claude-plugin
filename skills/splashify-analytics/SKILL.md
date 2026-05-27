---
name: splashify-analytics
description: Use whenever the user asks for messaging analytics, wallet balance, wallet transactions, billing details, invoices, GST profile, subscription / plan / add-ons / eligibility, expenses summary or trends or category breakdown, AI credits balance or transactions, voice AI rate / minutes / agents, account activity logs, or read-only dashboard / setup / KYC status. Triggers on "analytics", "trends", "wallet", "balance", "billing", "invoice", "subscription", "plan", "addons", "expenses", "AI credits", "voice credits", "activity log", "audit log".
license: MIT
---

# Splashify — analytics, wallet, billing, subscription, expenses, credits, activity

You read the user's reporting and billing surface on **Splashify Pro** by
running the `splashify` CLI through the Bash tool. Nearly every command in
this skill is read-only.

## Output contract

- Success → JSON on stdout, exit code `0`.
- Failure → `error: <message>` on stderr, non-zero exit code.

Relay errors verbatim.

## Confirmation rule

Everything in this skill is **read-only** — run without confirmation. The
only mutating action reachable here is

```
splashify activity (write actions are not available — read only)
```

…which is nominal. If the user asks for something that would mutate state
(top-up wallet, change plan, etc.), redirect them to the relevant skill or
to the Splashify app — those operations require interactive payment flows
not reachable through the CLI.

## Analytics

```
splashify analytics                          # messaging summary
splashify analytics trends                   # daily series
```

The summary returns counts per channel (WhatsApp, RCS, Instagram) and per
status (sent, delivered, read, failed). Summarise the headline counters for
the user; do not dump the full JSON unless asked.

## Wallet

```
splashify wallet                             # current balance
splashify wallet transactions                # ledger
```

The ledger is paginated server-side. If the user asks for "all" wallet
transactions, surface the latest page and tell them how many more pages
exist — do not auto-paginate to the end.

## Billing

```
splashify billing                            # consolidated billing
splashify billing profile                    # GST profile + billing address
splashify billing invoices                   # invoice list
splashify billing logs [--period all]
```

Invoices are PDFs hosted by the backend. The CLI returns the invoice
metadata + a `download_url`; surface the URL to the user, do not try to
fetch the PDF inline.

## Subscription

```
splashify subscription                       # plan + add-ons + eligibility + available plans
splashify subscription status                # only the active plan
splashify subscription plans                 # available plans for upgrade
splashify subscription addons                # active add-ons
```

If the user asks to "upgrade to Pro", point them at the app — payment is
not reachable through the CLI. Surface the plans / pricing and the upgrade
URL.

## Expenses

```
splashify expenses                           # summary (default 30d)
splashify expenses summary    [--period 7d|30d|3m|6m|all]
splashify expenses categories [--period …]
splashify expenses countries  [--period …]
splashify expenses trends     [--period …]
splashify expenses logs       [--period …] [--limit N] [--category …] \
                              [--country IN] [--free-trial true|false]
splashify expenses export     [--period …] [--limit N] [--out file.csv]
```

`expenses export` writes a CSV to disk — confirm the output path with the
user before invoking (so they know where to look afterward).

## AI credits

```
splashify credits                            # AI credits + voice AI rate + agents
splashify credits ai                         # AI credit balance
splashify credits transactions               # AI credit ledger
splashify credits voice                      # voice AI rate, balance, trial, minutes
splashify credits agents                     # voice AI agents
```

`credits voice` returns the per-minute voice rate, the trial-minutes
remaining, and the paid-minute balance. When the user asks "how many calls
can I make", compute trial + paid minutes ÷ average call length and surface
the assumption.

## Activity logs (audit trail)

```
splashify activity                           # latest 100 logs
splashify activity --limit 200
splashify activity --action login
splashify activity --entity contact [--entity-id <id>]
splashify activity --actor <user_id>
splashify activity --search "…"
```

Useful for "who deleted this contact" / "what did Alice do last week" /
"did a token leak get used". When the user asks an investigation question,
combine `--actor`, `--entity`, and `--action` filters to narrow first;
the unfiltered query returns 100 rows max.

## Reading results

For every analytics/billing read, surface:

1. The **headline number** (balance, total sent, total spent, plan name).
2. The **trend or context** (vs last period, % change, what's expiring).
3. The **next action** if the user mentioned one (e.g. "upgrade", "top up").

Never dump the raw JSON unless the user asks.

## Do not

- Do not try to mutate billing / plans / payments through the CLI — those
  flows are interactive in the app.
- Do not invent expense categories or AI-credit prices — if the user asks
  "what's a WhatsApp message cost", surface the data the CLI returned and
  caveat anything you computed.
- Do not auto-export every period when the user asks for a summary — start
  with the default period and ask before expanding.

For commands not covered above load
[`references/cli-reference.md`](../../references/cli-reference.md).
