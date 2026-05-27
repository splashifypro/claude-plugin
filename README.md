# Splashify Pro for Claude Code

Drive a **Splashify Pro** messaging account from Claude Code — send WhatsApp,
RCS, and Instagram messages, run broadcasts, manage contacts and templates,
configure AI agents, run email campaigns, and read analytics.

The plugin wraps the [`splashify` CLI](https://github.com/splashifypro/cli),
which holds your `oc_live_` access token in `~/.splashify/config.json`. **The
plugin itself stores no credentials and Claude never sees your token.** See
[SECURITY.md](SECURITY.md) for the full security model.

---

## Install

### 1. Install the splashify CLI (one-time)

The plugin needs the `splashify` binary on your `PATH`.

**macOS / Linux:**

```bash
curl -fsSL https://raw.githubusercontent.com/splashifypro/cli/main/install.sh | bash
```

**Windows (PowerShell):**

```powershell
iwr -useb https://raw.githubusercontent.com/splashifypro/cli/main/install.ps1 | iex
```

Then connect your account once:

```bash
splashify connect
# Paste an oc_live_ token from app.splashifypro.com → Settings → Developer → Access Tokens
```

Verify it worked:

```bash
splashify whoami
splashify doctor
```

### 2. Install the plugin

From inside Claude Code:

```
/plugin marketplace add splashifypro/claude-plugin
/plugin install splashifypro@splashifypro
```

Or from your shell:

```bash
claude plugin marketplace add splashifypro/claude-plugin
claude plugin install splashifypro@splashifypro
```

---

## What you can ask Claude

Once installed, just talk to Claude in plain English. The plugin's seven skills
auto-load based on what you ask:

### Messaging

> *"Send a WhatsApp message to +919876543210 saying 'Your order has shipped'"*
> *"Reply to the conversation with John with the canned 'business-hours' message"*
> *"List open WhatsApp conversations from this week"*
> *"Send the order_update template to +14155551234 with vars [Alice, ORD-2024]"*

### Broadcasts

> *"Create a broadcast called 'May Sale' using the may_offer template to my 'VIP customers' segment"*
> *"What's the delivery status of broadcast bcst_…?"*
> *"Cancel the scheduled broadcast for tomorrow"*
> *"Rebroadcast to the contacts that failed"*

### Contacts & segments

> *"List my VIP contacts"*
> *"Tag this list of numbers as 'newsletter-signup'"*
> *"Create a dynamic segment for contacts in Mumbai who opted in this month"*
> *"Block +919999999999 from messaging me"*

### AI agents & automation

> *"Create a support AI agent for WhatsApp called 'Riya'"*
> *"Upload this PDF as knowledge for my support agent"*
> *"Show me all my Click-to-WhatsApp ads campaigns"*

### Email marketing

> *"Set up the domain transactional.acme.com and verify DNS"*
> *"Create an email campaign 'May Newsletter' to my 'subscribers' segment"*

### Analytics & billing

> *"How many messages did I send last week?"*
> *"What's my wallet balance and recent transactions?"*
> *"Show me my expense trend over 3 months"*

### Account & ops

> *"Add Alice as a manager with read-write on contacts and analytics"*
> *"What's my WABA setup status?"*
> *"Open a support ticket about template approval delay"*

---

## How it works

```
You ── Claude Code ── splashify CLI ── HTTPS, Bearer oc_live_ ── api.splashifypro.com
                                          │
                                          └── reads ~/.splashify/config.json (mode 0600)
```

- The plugin contributes **skills** — markdown instructions that teach Claude
  *when* and *how* to invoke `splashify` subcommands.
- Claude uses its **Bash tool** to run `splashify` with the right flags.
- The CLI talks to the Splashify Pro backend with your token; Claude reads the
  JSON the CLI prints and summarises it for you.

This is the same architecture the
[OpenClaw skill](https://github.com/splashifypro/cli/tree/main/openclaw-skill)
uses — production-tested, with the token kept out of every Claude-visible
surface.

---

## Skills

| Skill | When it loads |
|-------|---------------|
| `splashify-setup` | account connection, `whoami`, `doctor`, profile, dashboard, tokens |
| `splashify-messaging` | sending WhatsApp/RCS/Instagram messages, conversations, contacts, tags, media |
| `splashify-broadcasts` | broadcasts, WhatsApp templates, segments, attributes |
| `splashify-automation` | AI agents, flows, integrations, CTWA, canned messages, calling |
| `splashify-analytics` | analytics, wallet, billing, subscription, expenses, AI credits, activity |
| `splashify-email` | email marketing — domains, templates, audiences, campaigns |
| `splashify-account-admin` | team, WABA, opt management, allowed IPs, devices, support tickets |

The complete CLI command reference is in
[`references/cli-reference.md`](references/cli-reference.md).

---

## Confirmation rule

The plugin enforces a confirmation rule before any state-mutating action — you
will always see a prompt like *"Send to **+91…7890**: 'Your order has shipped'
— go?"* before a real message goes out. See
[SECURITY.md → Confirmation rule](SECURITY.md#3-confirmation-rule-for-state-mutating-actions).

Read-only commands (`whoami`, `conversations`, `contacts`, `templates`,
`analytics`, `wallet`, `splashify api GET …`) run without prompting.

---

## Troubleshooting

### `not connected — run 'splashify connect' first`

The CLI does not have a token. Run `splashify connect` and paste a fresh
`oc_live_` token from **app.splashifypro.com → Settings → Developer → Access
Tokens**.

### `token validation failed`

Your token is invalid, revoked, or expired. Create a new one in the app and
re-run `splashify connect`.

### `your Splashify Pro account does not have a WhatsApp Business Account connected yet`

Finish the Meta Embedded Signup at app.splashifypro.com → WhatsApp → Connect
Number, then retry. WhatsApp sends will fail until a WABA is connected; other
features (email, contacts, analytics, billing) still work.

### `splashify: command not found`

Re-run the install script and make sure `splashify` is on your `PATH`. The
installer prints the install directory at the end — add it to `PATH` and open
a new shell.

### Claude is sending without asking

Open an issue with the prompt you used. Skills are written to require
confirmation before any mutating action; if Claude skipped the prompt, that is
a bug we want to fix.

---

## Contributing

The plugin lives at
[github.com/splashifypro/claude-plugin](https://github.com/splashifypro/claude-plugin).
The CLI it drives lives at
[github.com/splashifypro/cli](https://github.com/splashifypro/cli).

Issues and pull requests welcome. For security issues, please email
**security@splashifypro.com** rather than filing a public issue (see
[SECURITY.md](SECURITY.md)).

---

## License

MIT — see [LICENSE](LICENSE).
