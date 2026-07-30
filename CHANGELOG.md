# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Consumers should pin to a rolling major tag (`uses: pphatdev/telegrambot-alert-action@v1`) to automatically receive backwards-compatible fixes.

## [Unreleased]

## [0.1.0] — 2026-07-30

### Added

- Composite GitHub Action that sends build, deploy, release, and CI status alerts to Telegram (`action.yml`).
- Inputs: `telegram-bot-token`, `telegram-chat-id`, `status`, `stage`, `environment`, `extra`, `disable-notification`.
- Output: `message-id` — the Telegram `message_id` of the sent alert.
- Friendly HTML message format with per-field emoji labels, celebratory trailer emoji on success/failure, and clickable commit / run / actor links.
- Actor name links to the GitHub profile, with a `pphatdev` fallback when `github.actor` is empty.
- Example workflows:
  - `ci.yml` — build/test on push/PR with runtime selector (`node` | `bun` | `deno`) and configurable version.
  - `deploy.yml` — Cloudflare Workers deploy with the same runtime selector.
  - `release.yml` — notify on GitHub release published.
  - `test-action.yml` — smoke test for `action.yml`.
- Dependabot config for weekly action pin updates.
- MIT license and `.gitignore`.

### Security

- Secrets re-registered with `::add-mask::` at the start of the composite step so they stay redacted across the action boundary.
- All caller-supplied and `github.*` context values are passed through `env:` and only concatenated inside double-quoted bash strings — never spliced into a `run:` script via `${{ ... }}`.
- All third-party actions pinned to commit SHAs with a trailing version comment.

### Fixed

- Newline handling in the message body — real newlines are now used instead of literal `%0A`, which `curl --data-urlencode` was double-encoding as `%250A` so Telegram rendered them as text. The `extra` input accepts `\n` for line breaks.
- Node setup no longer fails when the consuming repo has no lockfile — `cache: npm` is only enabled when a lockfile is present, and install steps skip cleanly when no `package.json` / `deno.json` exists.

[Unreleased]: https://github.com/pphatdev/telegrambot-alert-action/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/pphatdev/telegrambot-alert-action/releases/tag/v0.1.0
