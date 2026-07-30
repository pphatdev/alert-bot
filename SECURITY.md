# Security Policy

## Supported versions

Only the latest major-version tag receives security fixes. Consumers should pin to a rolling major tag (`uses: pphatdev/build-alert@v1`) to receive them automatically.

| Version | Supported |
|---|---|
| `v1.x` | ✅ |
| `< v1` | ❌ |

## Reporting a vulnerability

**Please do not open a public GitHub issue for security reports.**

Use one of the following private channels:

1. **GitHub private vulnerability reporting** (preferred) — [open a private advisory](https://github.com/pphatdev/tg-alert-bot/security/advisories/new).
2. **Email** — `turbotech.kh@gmail.com` with the subject line `build-alert security`.

Include:

- A description of the issue and its impact.
- Steps to reproduce (a minimal workflow YAML is ideal).
- The commit SHA or version tag affected.
- Any suggested remediation.

You will receive an acknowledgement within **72 hours**. A fix and coordinated disclosure timeline will be proposed within **14 days** of confirmation.

## In scope

- Leakage of `telegram-bot-token` or `telegram-chat-id` in job logs, action outputs, or Telegram messages.
- Command injection via `stage`, `extra`, `environment`, or any `github.*` context value.
- Bypass of the composite action's secret masking.
- Vulnerabilities in the example workflows (`ci.yml`, `deploy.yml`, `release.yml`, `verify.yml`, `test-action.yml`) that could be exploited by a forked-PR contributor.

## Out of scope

- Telegram platform issues (report to Telegram).
- Cloudflare API / Wrangler issues (report to Cloudflare).
- Third-party actions we depend on — please report those upstream (`actions/checkout`, `actions/setup-node`, `oven-sh/setup-bun`, `denoland/setup-deno`, `cloudflare/wrangler-action`, `mpalmer/action-validator`). Our SHA pins protect against silent upstream changes.
- Rate limits, chat-not-found, or bot-token-revoked errors that surface as loud failures — these are working as intended.

## What we do

- All third-party actions are SHA-pinned with a trailing `# vX.Y.Z` comment. Dependabot opens weekly PRs to update them.
- Secrets are re-registered with `::add-mask::` inside the composite step so they stay redacted across the composite-action boundary.
- All caller-supplied and `github.*` values flow through `env:` and are only concatenated inside double-quoted bash strings, never spliced into a `run:` script via `${{ ... }}`.
- The included `verify.yml` workflow runs schema and marketplace-readiness checks on every push and PR touching `action.yml`.
