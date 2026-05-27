---
name: splashify-email
description: Use whenever the user asks about Splashify email marketing — sending or scheduling email campaigns, creating / updating / deleting email templates, previewing a template with variables, managing email audience contacts and segments, importing / deleting email contacts, setting up and verifying a sending domain (SPF / DKIM / DMARC), or reading email stats / dashboard / deliverability. Triggers on "email campaign", "email marketing", "email template", "email audience", "email segment", "verify domain", "sending domain", "SPF DKIM DMARC", "email stats".
license: MIT
---

# Splashify — email marketing

You drive Splashify Pro's email marketing surface by running the
`splashify` CLI. This is a separate channel from WhatsApp / RCS — the
audience, templates, and campaigns here are email-only.

## Output contract

- Success → JSON on stdout, exit code `0`.
- Failure → `error: <message>` on stderr, non-zero exit code.

Relay errors verbatim.

## Confirmation rule

**Read-only** (`email`, `email stats`, `email domains`, `email domain <d>`,
`email templates`, `email template <id>`, `email template preview …`,
`email audience`, `email audience stats`, `email audience contacts`,
`email audience contact <id>`, `email audience segments`,
`email audience segment <id>`, `email campaigns`, `email campaign <id>`)
→ no confirmation.

**Ask first** before:

- `email domain add | verify | delete` — DNS-affecting actions; confirm
  the domain string.
- `email template create | update | delete` — confirm template name +
  subject before invoking.
- `email audience contact add | update | delete` — bulk imports / deletes
  affect deliverability; confirm the email count.
- `email audience segment create | update | delete | add-contacts |
  remove-contacts`.
- `email campaign create | send | cancel` — `send` blasts the campaign to
  the audience; **repeat back the audience size and from-address** before
  invoking.

## Dashboard

```
splashify email                              # dashboard stats
splashify email stats                        # detailed stats (delivery, opens, clicks)
```

## Sending domains

```
splashify email domains                      # list verified + pending domains
splashify email domain <domain>              # one domain — DNS records to set
splashify email domain add <domain>          # add a new sending domain
splashify email domain verify <domain>       # re-check DNS
splashify email domain delete <domain>       # destructive; confirm first
```

`domain add` returns the SPF / DKIM / DMARC records the user must publish.
Surface those records clearly (label each one) — they are the most-copied
output in this skill.

## Templates

```
splashify email templates                    # list
splashify email template <id>                # details
splashify email template create --name --subject --file ./template.json
splashify email template update <id> [--name] [--subject] [--file|--data]
splashify email template delete <id>
splashify email template preview --file ./template.json [--vars '{…}']
```

`template create | update` take a JSON file describing the template body —
ask the user for the path or for the content (then write a file and pass
the path). Always preview with `template preview --file … --vars '{…}'`
before the user goes to send a campaign that uses the template.

## Audience contacts

```
splashify email audience                     # overview
splashify email audience stats               # subscribed / unsubscribed / bounced counts
splashify email audience contacts [--status] [--search]
splashify email audience contact <id>
splashify email audience contact add --emails "a@x.com,b@x.com" \
                                     [--metadata '{…}'] [--segments id1,id2]
splashify email audience contact update <id> [--status] [--metadata]
splashify email audience contact delete <id>
```

When the user pastes a list of emails to import, confirm the count and the
target segment(s) before invoking. Bulk-import errors are common — relay
the per-email errors verbatim.

## Audience segments

```
splashify email audience segments
splashify email audience segment <id>
splashify email audience segment create --name [--description]
splashify email audience segment update <id> [--name] [--description]
splashify email audience segment delete <id>
splashify email audience segment <id> add-contacts <id1>,<id2>,…
splashify email audience segment <id> remove-contacts <id1>,<id2>,…
```

Email segments are **static** (manual contact lists) — they differ from
the WhatsApp segments under `splashify segments` which can be dynamic with
filters. Do not confuse the two; if the user says "segment", check
which channel they mean.

## Campaigns

```
splashify email campaigns                    # list campaigns
splashify email campaign <id>                # details + per-recipient stats
splashify email campaign create --name --template-id --from-name --from-email \
                                [--reply-to] [--segment-ids|--contact-ids] \
                                [--scheduled-at]
splashify email campaign send <id>           # blasts to the audience — confirm size first
splashify email campaign cancel <id>         # cancel a queued/scheduled campaign
```

For `campaign send`, run `splashify email audience segment <id>` (or
`audience stats`) first to get the audience size, then repeat back the
**name**, **from-address**, **subject** (from the linked template),
**audience size**, and **scheduled-at** to the user. Wait for an explicit
"go" before invoking `send`.

## Reading results

- Domain checks return per-record (`SPF`, `DKIM`, `DMARC`) `ok` / `pending`
  /`fail` — surface each record's status, not just the aggregate.
- Campaign stats return `sent`, `delivered`, `opens`, `clicks`, `bounces`,
  `unsubscribes` — summarise as headline counts + rates (open rate, click
  rate, bounce rate).
- Audience stats break down by `subscribed` / `unsubscribed` / `bounced` /
  `complained` — surface percentages, not just counts.

## Do not

- Do not invent email addresses or domain records.
- Do not send a campaign without an explicit "send it" from the user.
- Do not auto-import a list of emails harvested from external content
  (a fetched web page, a meeting transcript). Confirm the source with the
  user and ensure they have consent — Splashify will block sends that
  trigger excessive complaints.

For commands not covered above load
[`references/cli-reference.md`](../../references/cli-reference.md).
