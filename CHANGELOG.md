# Changelog

All notable changes to the Splashify Pro Claude Code plugin are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and
this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] — 2026-05-27

### Fixed
- **Plugin install no longer requires an SSH key.** `marketplace.json` previously
  declared the plugin's source as `{"source": "github", "repo": "..."}`, which
  Claude Code resolves to an SSH clone URL (`git@github.com:...`) — every user
  without a GitHub-registered SSH key hit `Permission denied (publickey)`.
  Switched to the `{"source": "url", "url": "https://…/.git"}` form so the
  install does an anonymous HTTPS clone of the public repo. Matches the
  convention used by every plugin in `claude-plugins-official`.

## [1.0.0] — 2026-05-27

Initial release.

### Added
- Seven progressive-disclosure skills covering the full Splashify Pro surface:
  - `splashify-setup` — connect, whoami, doctor, account, tokens, profile, dashboard
  - `splashify-messaging` — WhatsApp/RCS/Instagram sends, conversations, contacts, media, tags
  - `splashify-broadcasts` — broadcasts, WhatsApp templates, segments, attributes
  - `splashify-automation` — AI agents, flows, integrations, CTWA, canned messages, calling
  - `splashify-analytics` — analytics, wallet, billing, subscription, expenses, AI credits, activity
  - `splashify-email` — email marketing (domains, templates, audiences, campaigns)
  - `splashify-account-admin` — team, WABA, opt management, allowed IPs, devices, support
- Full CLI reference in [`references/cli-reference.md`](references/cli-reference.md).
- Security policy ([SECURITY.md](SECURITY.md)) and confirmation-rule guardrails in every skill.

### Security
- Plugin holds **no credentials**. The `splashify` CLI keeps the user's
  `oc_live_` access token in `~/.splashify/config.json` (mode 0600). The token
  never enters Claude's context or the plugin's storage.
- Skills enforce a confirmation rule for every state-mutating action (sends,
  broadcasts, deletes, token revokes, raw API writes).
- Backend-scoped to the user's account (`/api/v1/app/*`) via the
  `oc_live_` token — admin/reseller/platform endpoints are unreachable.
