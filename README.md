# build-alert

A **GitHub Action** that sends build, deploy, release, and CI status alerts to **Telegram**.
Ships with ready-to-use CI/CD workflows for **Cloudflare Workers** as examples.

## Use it in your workflow

```yaml
- uses: sophat/build-alert@v1
  with:
    telegram-bot-token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
    telegram-chat-id:   ${{ secrets.TELEGRAM_CHAT_ID }}
    status: success            # success | failure | cancelled | started
    stage:  Deploy             # any label — Build, Deploy, Release, Migration, ...
    environment: production    # optional
    extra: "<b>URL:</b> https://example.workers.dev"   # optional HTML, \n for newlines
```

Inside this repo, workflows reference it with `uses: ./` (local path). External repos use `uses: sophat/build-alert@v1` once you tag a release.

## Inputs

| Input | Required | Description |
|---|---|---|
| `telegram-bot-token` | ✅ | Bot token from @BotFather |
| `telegram-chat-id` | ✅ | Chat ID (user or group, e.g. `-1001234567890`) |
| `status` | ✅ | `success` \| `failure` \| `cancelled` \| `started` |
| `stage` | ✅ | Free-form label — `Build`, `Deploy`, `Release`, `Migration`, ... |
| `environment` | ❌ | `production`, `staging`, `preview`, ... |
| `extra` | ❌ | Extra HTML line(s) appended. Use `\n` for newlines. |
| `disable-notification` | ❌ | `true` for silent messages |

## Outputs

| Output | Description |
|---|---|
| `message-id` | Telegram `message_id` of the sent alert |

## Sample message

```
✅ Deploy Succeeded 🎉

📦 Repo: sophat/my-worker
🌿 Branch: main
🌐 Env: production
👤 Actor: turbotech
⚡ Event: push
🔖 Commit: 3f2a1b0
🏃 Run: #12345
URL: https://my-worker.workers.dev
```

## Included example workflows

| File | Trigger | Alerts |
|---|---|---|
| `.github/workflows/ci.yml` | push / PR on `main`, `develop` | Build success or failure |
| `.github/workflows/deploy.yml` | push to `main`, manual dispatch | Deploy started, success (with URL), or failure |
| `.github/workflows/release.yml` | GitHub release published | Release announcement with tag + URL |
| `.github/workflows/test-action.yml` | manual / change to `action.yml` | Sends a smoke-test message |

## Setup

### 1. Create a Telegram bot

1. Message **@BotFather** → `/newbot` → follow prompts. Copy the token.
2. Message your bot (or add it to a group and send any message).
3. Get your **chat ID**:
   - Personal chat: message @userinfobot, or open `https://api.telegram.org/bot<TOKEN>/getUpdates` and read `"chat":{"id":...}`.
   - Group: add the bot to the group, send a message, then hit `getUpdates`. Group IDs are negative (e.g. `-1001234567890`).

### 2. Add repository secrets

`Settings → Secrets and variables → Actions → New repository secret`:

| Secret | Value |
|---|---|
| `TELEGRAM_BOT_TOKEN` | Bot token from BotFather |
| `TELEGRAM_CHAT_ID` | Chat ID (user or group) |
| `CLOUDFLARE_API_TOKEN` | Cloudflare API token with **Workers Scripts: Edit** (only needed for `deploy.yml`) |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account ID (only needed for `deploy.yml`) |

Create the CF token at `dash.cloudflare.com → My Profile → API Tokens → Create Token → Edit Cloudflare Workers`.

### 3. Publish this action (so other repos can consume it)

```bash
git init && git add . && git commit -m "chore: initial build-alert action"
git tag v1
git push origin main --tags
```

Then in any other repo: `uses: <your-gh-org>/build-alert@v1`.

## Advanced

**Different chat per environment.** Create GitHub Environments (`Settings → Environments`) and set a per-environment `TELEGRAM_CHAT_ID` secret. Jobs that bind to that environment automatically pick it up.

**Notify on any event.** The action is stage-agnostic — call it from cron workflows, migration jobs, security scans, etc. Just set `stage` to whatever label you want to see in the alert.

**Silent alerts.** Set `disable-notification: "true"` for low-priority updates.

## Security

- **Secret masking.** The action re-registers `telegram-bot-token` and `telegram-chat-id` with `::add-mask::` at the start of its script so they are redacted from logs even across the composite-action boundary.
- **Injection-safe interpolation.** All caller-supplied and `github.*` context values (commit messages, branch names, etc.) flow through `env:` and are only concatenated inside double-quoted bash strings — never spliced into a `run:` script via `${{ ... }}`. A commit message like `$(rm -rf /)` becomes a literal string sent to Telegram, not executed.
- **No secrets on forked PRs.** `ci.yml` uses `pull_request` (not `pull_request_target`), so fork PRs get an empty secret and the notify step will fail loudly rather than leak.
- **Least-privilege token.** Every workflow declares `permissions: contents: read`. The Cloudflare deploy uses the CF API token, not `GITHUB_TOKEN`.
- **SHA-pinned actions.** Third-party actions are pinned to commit SHAs with the version in a trailing comment. Dependabot (`.github/dependabot.yml`) opens weekly PRs to update them.

## Troubleshooting

- **`chat not found`** — you haven't messaged the bot yet, or the chat ID is wrong. Group IDs must include the `-100` prefix.
- **`Unauthorized`** — bot token is wrong or was regenerated.
- **Deploy fails with 10000/authentication error** — the Cloudflare API token is missing the Workers Scripts scope, or the Account ID is wrong.
