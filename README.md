# telegrambot-alert-action

A **GitHub Action** that sends build, deploy, release, and CI status alerts to **Telegram**.
Ships with ready-to-use CI/CD workflows for **Cloudflare Workers** as examples.

## Use it in your workflow

```yaml
- uses: pphatdev/telegrambot-alert-action@v1.1
  with:
    telegram-bot-token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
    telegram-chat-id:   ${{ secrets.TELEGRAM_CHAT_ID }}
    status: success            # success | failure | cancelled | started
    stage:  Deploy             # any label — Build, Deploy, Release, Migration, ...
    environment: production    # optional
    extra: "<b>URL:</b> https://example.workers.dev"   # optional HTML, \n for newlines
```

Inside this repo, workflows reference it with `uses: ./` (local path). External repos use `uses: pphatdev/telegrambot-alert-action@v1.1` once you tag a release.

### PR-comment fallback (no Telegram creds required)

If you omit `telegram-bot-token` / `telegram-chat-id`, the action **skips the Telegram send** and, on `pull_request` / `pull_request_target` events, posts the same alert as a Markdown comment on the PR. On any other event — or when the comment can't be posted (missing `pull-requests: write` permission, fork PR with a read-only token, 4xx from the API) — it emits a warning, sets `channel=none`, and **exits successfully** with the rendered body available on the `message` output. Downstream steps keep running; you never lose the job over a missing notify channel.

> **Fork PRs.** `pull_request` triggers from forks get a read-only `GITHUB_TOKEN`, so the fallback comment will 403 and skip. Either gate the notify step (`if: github.event.pull_request.head.repo.fork == false`) or use `pull_request_target` (only if you've reviewed the security implications).

```yaml
name: PR checks
on: pull_request

permissions:
  contents: read
  pull-requests: write   # required for the fallback comment

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

      - if: always()
        uses: pphatdev/telegrambot-alert-action@v1.1
        with:
          # telegram-* intentionally omitted → fallback to PR comment
          status: ${{ job.status }}
          stage:  CI
          extra:  "<b>Run:</b> ${{ github.run_id }}"
```

Provide the creds later (repo/organization secrets) and the same call automatically switches back to Telegram — no workflow change needed.

### Reading the outputs

Give the notify step an `id` and downstream steps can branch on which channel was used or reuse the rendered message:

```yaml
      - id: notify
        if: always()
        uses: pphatdev/telegrambot-alert-action@v1.1
        with:
          telegram-bot-token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          telegram-chat-id:   ${{ secrets.TELEGRAM_CHAT_ID }}
          status: ${{ job.status }}
          stage:  Deploy

      - if: steps.notify.outputs.channel == 'none'
        run: echo "Alert not delivered; body was:" && echo "${{ steps.notify.outputs.message }}"
```

## Inputs

| Input | Required | Description |
|---|---|---|
| `telegram-bot-token` | ❌ | Bot token from @BotFather. Omit to skip Telegram and use the PR-comment fallback. |
| `telegram-chat-id` | ❌ | Chat ID (user or group, e.g. `-1001234567890`). Omit to skip Telegram and use the PR-comment fallback. |
| `status` | ✅ | `success` \| `failure` \| `cancelled` \| `started` |
| `stage` | ✅ | Free-form label — `Build`, `Deploy`, `Release`, `Migration`, ... |
| `environment` | ❌ | `production`, `staging`, `preview`, ... |
| `extra` | ❌ | Extra HTML line(s) appended. Use `\n` for newlines. |
| `disable-notification` | ❌ | `true` for silent messages |
| `github-token` | ❌ | Token used to post the PR-comment fallback. Defaults to `${{ github.token }}`; caller workflow must grant `pull-requests: write`. |

## Outputs

| Output | Description |
|---|---|
| `message-id` | Telegram `message_id` of the sent alert (empty when Telegram was skipped) |
| `comment-id` | GitHub PR comment id when the fallback comment was posted (empty otherwise) |
| `channel`    | Which channel received the alert: `telegram`, `pr-comment`, or `none` |
| `message`    | The rendered alert body (HTML for Telegram, Markdown for PR comments) |

## Sample message

```
✅ Deploy Succeeded 🎉

📦 Repo: pphatdev/my-worker
🌿 Branch: main
🌐 Env: production
👤 Actor: pphatdev
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
| `.github/workflows/test-action.yml` | manual / push / PR touching `action.yml` | `smoke` job sends a Telegram message; `pr-fallback` job runs on PRs without creds and asserts the PR-comment fallback path |

### Choosing runtime & version

`ci.yml` and `deploy.yml` accept a `runtime` (`node` \| `bun` \| `deno`) and a `runtime-version` when dispatched manually (`Actions → Run workflow`). Push/PR runs use the defaults (`node` 20).

| Runtime | Install | Scripts source |
|---|---|---|
| node | `npm ci` | `package.json` → `scripts` (uses `--if-present`) |
| bun | `bun install --frozen-lockfile` | `package.json` → `scripts` (skipped if missing) |
| deno | `deno install` | `deno.json` → `tasks` (skipped if missing) |

The workflows call `typecheck`, `lint`, `test`, and `build` in that order — define whichever your project has. The deploy workflow only runs `build`.

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
git init && git add . && git commit -m "chore: initial telegrambot-alert-action"
git tag v1.1.0
git tag -f v1.1        # rolling minor tag
git tag -f v1          # rolling major tag
git push origin main --tags --force
```

Then in any other repo: `uses: pphatdev/telegrambot-alert-action@v1.1`.

## Advanced

**Different chat per environment.** Create GitHub Environments (`Settings → Environments`) and set a per-environment `TELEGRAM_CHAT_ID` secret. Jobs that bind to that environment automatically pick it up.

**Notify on any event.** The action is stage-agnostic — call it from cron workflows, migration jobs, security scans, etc. Just set `stage` to whatever label you want to see in the alert.

**Silent alerts.** Set `disable-notification: "true"` for low-priority updates.

## Security

- **Secret masking.** The action re-registers `telegram-bot-token` and `telegram-chat-id` with `::add-mask::` at the start of its script so they are redacted from logs even across the composite-action boundary.
- **Injection-safe interpolation.** All caller-supplied and `github.*` context values (commit messages, branch names, etc.) flow through `env:` and are only concatenated inside double-quoted bash strings — never spliced into a `run:` script via `${{ ... }}`. A commit message like `$(rm -rf /)` becomes a literal string sent to Telegram, not executed.
- **No secrets on forked PRs.** `ci.yml` uses `pull_request` (not `pull_request_target`), so fork PRs get empty secrets. The notify step routes to the PR-comment fallback, which soft-skips with a `::warning::` (fork PRs also can't post comments with the read-only token) — the job stays green and no secret material can leak.
- **Least-privilege token.** Every workflow declares `permissions: contents: read`. The Cloudflare deploy uses the CF API token, not `GITHUB_TOKEN`.
- **SHA-pinned actions.** Third-party actions are pinned to commit SHAs with the version in a trailing comment. Dependabot (`.github/dependabot.yml`) opens weekly PRs to update them.

## Troubleshooting

- **`chat not found`** — you haven't messaged the bot yet, or the chat ID is wrong. Group IDs must include the `-100` prefix.
- **`Unauthorized`** — bot token is wrong or was regenerated.
- **Deploy fails with 10000/authentication error** — the Cloudflare API token is missing the Workers Scripts scope, or the Account ID is wrong.
