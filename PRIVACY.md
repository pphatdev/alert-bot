# Privacy Notice

This document explains what data the **telegrambot-alert-action** GitHub Action handles, where it goes, and what is retained. It applies to the composite action defined in `action.yml`, not to any consuming workflow's own logic.

## TL;DR

- The action makes exactly **one outbound HTTP call**: `POST https://api.telegram.org/bot<TOKEN>/sendMessage`.
- No data is sent anywhere else. There is no telemetry, no analytics, no third-party endpoint.
- Secrets are re-masked so they cannot appear in job logs.
- Retention of the sent message is governed by Telegram, not by this action.

## Data the action reads

| Category | Source | Purpose |
|---|---|---|
| Telegram bot token | `inputs.telegram-bot-token` | Authenticate the API call |
| Telegram chat ID | `inputs.telegram-chat-id` | Route the message |
| Status / stage / environment | `inputs.*` | Message content |
| Extra HTML line | `inputs.extra` | Message content |
| Repository name | `github.repository` | Message content |
| Branch name | `github.ref_name` | Message content |
| Actor username | `github.actor` | Message content, profile link |
| Commit SHA | `github.sha` | Message content, commit link |
| Run ID | `github.run_id` | Message content, run link |
| Event name | `github.event_name` | Message content |
| Server URL | `github.server_url` | Building profile / commit / run URLs (GitHub.com or Enterprise host) |

None of this is sent anywhere except to the Telegram Bot API endpoint described above.

## Data the action writes

| Destination | Data | Notes |
|---|---|---|
| Telegram chat | Rendered HTML message | Retention governed by Telegram and your chat policy. |
| Job logs | Raw Telegram API response (`echo "$RESP"`) | Bot token and chat ID are re-masked with `::add-mask::` before this line runs, so they appear as `***`. |
| Action output | `message-id` | The numeric Telegram `message_id`. No user data. |

The action never writes to disk, uploads artifacts, or calls back to any endpoint owned by the maintainers.

## Secret handling

- `telegram-bot-token` and `telegram-chat-id` are re-registered with `::add-mask::` at the start of the composite step. This is defence-in-depth on top of GitHub's normal secret masking, so the values stay redacted even across the composite-action boundary.
- The token is passed to `curl` via a `-d` argument inside the runner process. It never appears in the workflow YAML or in the run log.
- No credentials are persisted anywhere by this action.

## Third parties

- **Telegram** — receives the rendered message. See [Telegram's Privacy Policy](https://telegram.org/privacy).
- **GitHub** — hosts the runner and the job logs. See [GitHub's Privacy Statement](https://docs.github.com/site-policy/privacy-policies/github-general-privacy-statement).

The maintainers of this action receive **no data** from your use of it.

## Your responsibilities as a consumer

- Store `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` as GitHub Actions secrets — never in the workflow YAML, never in `env:` defaults.
- Do not pass user-controlled input (e.g. from `pull_request_target` events) into `extra` without sanitising the HTML.
- If your chat is shared, remember that everyone in it sees the alert content — including branch names and commit messages.

## Changes to this notice

Material changes will be recorded in `CHANGELOG.md` and, if impactful, called out in the release notes.

## Contact

Questions about data handling: `hi@pphat.me`.
