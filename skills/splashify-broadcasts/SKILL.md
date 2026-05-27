---
name: splashify-broadcasts
description: Use whenever the user asks to start a marketing broadcast, schedule a campaign, check broadcast progress / status / delivery / cohorts, cancel or restart a broadcast, rebroadcast to a cohort (failed / sent / delivered / read), list or sync WhatsApp templates, create or delete a template, or manage audience segments and custom contact attributes. Triggers on "broadcast", "campaign", "send template to segment", "broadcast status", "broadcast progress", "rebroadcast", "WhatsApp template", "template list", "create template", "segment", "attribute".
license: MIT
---

# Splashify — broadcasts, templates, segments

You run and monitor broadcasts on the user's **Splashify Pro** account by
driving the `splashify` CLI through the Bash tool. The CLI carries the
user's credentials. You carry **no credentials of your own**.

## Output contract

- **Success** → JSON on stdout, exit code `0`.
- **Failure** → `error: <message>` on stderr, **non-zero exit code**.

Relay errors verbatim. Never invent template names, segment ids, or
broadcast ids.

## Confirmation rule

**Read-only** (`broadcasts`, `broadcast <id>`, `broadcast stats`,
`broadcast <id> progress`, `broadcast <id> messages`,
`broadcast <id> cohorts`, `broadcast <id> export`, `templates`,
`template <id>`, `segments`, `segments stats`, `segment <id>`,
`segment <id> contacts`, `segment <id> count`, `attributes`,
`attribute <id>`) → no confirmation.

**Ask first** for anything that mutates:

- `broadcast create` — repeat back the **name**, **template**, **audience
  size** (segment name + count, or contact-id count), and **send time**
  before invoking.
- `broadcast <id> cancel | restart | send-now | rebroadcast …` — destructive
  or large-fan-out actions; confirm the broadcast id and current status.
- `templates create | delete | sync <id>` — template approvals are a slow,
  costly cycle; confirm exact template names before mutating.
- `segment create | update | delete | refresh` — segments drive broadcast
  audience; confirm before mutating.
- `attribute create | update | delete | toggle-visibility | reorder` —
  schema changes; confirm before mutating.

When in doubt, pause and ask.

## Broadcasts

### Listing & inspection

```
splashify broadcasts [--status …] [--search "…"] [--page N] [--limit N]
splashify broadcast stats [--period 7d|30d|all]
splashify broadcast <id> [--recompute]                   # one broadcast; --recompute refreshes counters
splashify broadcast <id> messages [--status …] [--channel whatsapp|rcs] \
                                  [--page] [--limit] [--search] [--cumulative true|false]
splashify broadcast <id> cohorts                          # delivery cohorts (sent/delivered/read/failed)
splashify broadcast <id> export [--status ALL] [--csv]    # export per-recipient
```

### Live progress

```
splashify broadcast <id> progress              # follow SSE until terminal status
splashify broadcast <id> progress --once       # one snapshot only
splashify broadcast <id> progress --max N      # cap iterations
```

Prefer `--once` if you only need a snapshot — streaming output is harder to
summarise. Reach for the streaming form only if the user explicitly asked
to "watch" or "monitor" the broadcast.

### Creating a broadcast

```
splashify broadcast create --name "May Sale" \
  --template may_offer --category MARKETING --language en \
  (--segment-ids id1,id2 | --contact-ids id1,id2) \
  [--template-id <id>] [--params '{…}'] [--media-url https://…] \
  [--send-type now|scheduled] [--scheduled-at 2026-06-01T10:00:00Z] \
  [--rate-limit N] [--batch-size N] \
  [--rcs | --rcs-fallback-template-id … --rcs-fallback-template-name …] \
  [--yes]
```

Required: `--name`, `--template`, `--category`, `--language`, and **either**
`--segment-ids` or `--contact-ids` (not both).

`--scheduled-at` must be RFC 3339 UTC (`2026-06-01T10:00:00Z`). Convert
local times the user gives you into UTC before invoking; show your
conversion in the confirmation prompt so the user can correct it.

Do **not** pass `--yes` automatically — it bypasses the CLI's own
confirmation prompt. Reserve `--yes` for follow-ups where the user has
already confirmed.

### Cancel / restart / send-now / rebroadcast

```
splashify broadcast <id> cancel             # stop a queued or sending broadcast
splashify broadcast <id> restart            # restart a stalled broadcast
splashify broadcast <id> send-now           # promote a scheduled broadcast to immediate
splashify broadcast <id> rebroadcast --cohort failed|sent|delivered|read|all \
                                     [--name] [--template] [--template-id] [--params] \
                                     [--media-url] [--send-at] [--yes]
```

`rebroadcast --cohort all` is a very large action — confirm the cohort
size with the user (use `splashify broadcast <id> cohorts` first to get
counts) before invoking.

## WhatsApp templates

Templates must be approved by Meta before they can be sent in a broadcast.

```
splashify templates                          # list every template
splashify template <id>                      # details
splashify templates sync                     # sync ALL templates from Meta
splashify templates sync <id>                # sync one
splashify templates upload-media <file>      # upload media header asset
splashify templates create --name "…" --language en --category MARKETING --text "…"
splashify templates create --name "…" --language en --category MARKETING \
                            --components '[{"type":"BODY","text":"…"}]'
splashify templates create --file ./template.json
splashify templates delete <id> [--name <template_name>]
```

Template names are case-sensitive and usually snake_case. When the user
asks for a template by display name, look it up via `splashify templates`
first — do not guess.

## Segments (audiences)

```
splashify segments [--search] [--page] [--limit]
splashify segments stats
splashify segment <id>
splashify segment <id> contacts [--page] [--limit]
splashify segment <id> count
splashify segment <id> refresh                # recompute dynamic membership
splashify segment create --name "VIP" --filters '{…}' [--description "…"] \
                         [--dynamic true|false] [--active true|false]
splashify segment update <id> [--name] [--description] [--filters] [--dynamic] [--active]
splashify segment delete <id>
```

`--filters` is a JSON string — single-quote it on the command line so your
shell does not eat the braces. If the user does not give you a precise
filter, ask them to describe the audience and translate that to the filter
JSON yourself; then **show the translated JSON in the confirmation prompt**
so they can correct it.

## Custom contact attributes

```
splashify attributes [--search "…"]
splashify attribute <id>
splashify attribute create --label "Company" --type TEXT \
                           [--options "a,b,c"] [--visible true|false] \
                           [--required true|false] [--default "…"] [--help "…"]
splashify attribute update <id> [--label] [--type] [--visible] [--required] \
                                 [--options] [--default] [--help]
splashify attribute delete <id>
splashify attribute toggle-visibility <id>
splashify attribute reorder <id> up | down
```

Attribute types: `TEXT`, `NUMBER`, `BOOLEAN`, `DATE`, `SELECT`, `MULTI_SELECT`.
`--options` is required for `SELECT` / `MULTI_SELECT`.

## Reading results

Broadcast JSON is large. Summarise the headline counters for the user:

- Status (`queued`, `sending`, `delivered`, `failed`, `cancelled`,
  `scheduled`).
- Audience size and per-cohort counts (sent / delivered / read / failed).
- For a creation result, the new broadcast `id` and the time the first
  message will go out.

Never dump the full JSON unless the user asks.

## Do not

- Do not pass `--yes` unless the user has explicitly confirmed in this turn.
- Do not invent template names, segment ids, or broadcast ids.
- Do not run `templates sync` without asking — it pulls every template from
  Meta and can be slow for large accounts.
- Do not retry a failed broadcast create more than once; surface the error.

For commands not covered above load
[`references/cli-reference.md`](../../references/cli-reference.md).
