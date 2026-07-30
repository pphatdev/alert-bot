# Contributing

Thanks for your interest in telegrambot-alert-action. This is a small composite action — most contributions are welcome as PRs directly.

## Before you start

- Read the [Code of Conduct](CODE_OF_CONDUCT.md).
- For security issues, follow the [Security Policy](SECURITY.md) — do **not** open a public issue.
- Check open issues and PRs to avoid duplicating work.

## Reporting a bug

Open an issue with:

1. What you were trying to do.
2. The minimal workflow YAML that reproduces it (redact tokens).
3. Actual vs. expected behavior.
4. A link to the failing Actions run, if public.

## Suggesting a feature

Open an issue describing the use case first. Keep in mind the action's north star:

- Zero runtime dependencies (bash + curl only).
- Composite action, not a JS/Docker action.
- Stage-agnostic — no hard-coded assumptions about Node, Cloudflare, or any specific pipeline.
- Every third-party action is SHA-pinned.

Features that don't fit that shape are unlikely to land.

## Pull requests

### Setup

```bash
git clone https://github.com/pphatdev/telegrambot-alert-action.git
cd telegrambot-alert-action
git checkout -b your-branch
```

### Making changes

- **`action.yml`** is the source of truth. Keep it well-commented where behavior is non-obvious.
- If you touch the message format, update the sample in `README.md` too.
- If you add a new input/output, document it in the inputs/outputs tables in `README.md`.
- If you change marketplace-relevant copy, update `.github/marketplace-listing.md` as well.

### Testing locally

You cannot run a composite action outside of GitHub Actions, so testing means:

1. **Push your branch to a fork** (or a test branch on your own repo).
2. **Trigger `.github/workflows/test-action.yml`** manually (`Actions → Self-test (action.yml) → Run workflow`). This sends a real message to the chat behind `TELEGRAM_CHAT_ID`, so use a dev chat.
3. **Trigger `.github/workflows/verify.yml`** to validate the schema and marketplace requirements.
4. Confirm the message renders correctly in Telegram (newlines, links, formatting).

### Pinning third-party actions

Any new `uses:` line must reference a commit SHA with a trailing version comment:

```yaml
- uses: owner/repo@<40-char-sha>  # vX.Y.Z
```

Resolve tags to SHAs with:

```bash
gh api repos/OWNER/REPO/git/ref/tags/vX.Y.Z --jq '.object.sha'
```

Dependabot will pick it up from `.github/dependabot.yml` and open weekly update PRs.

### Commit messages

Follow the existing style:

- `feat: ...` — user-visible new capability
- `fix: ...` — bug fix
- `docs: ...` — README, marketplace copy, CHANGELOG
- `chore: ...` — deps, tooling, CI plumbing

Reference issues where relevant (`fix: newline handling in extra input (#12)`).

### Before opening the PR

- Update `CHANGELOG.md` under `[Unreleased]`.
- Confirm `verify.yml` passes on your branch.
- Rebase on `master` if there are conflicts.

## Release process (maintainers)

1. Move the `[Unreleased]` block in `CHANGELOG.md` to a new `[X.Y.Z]` section with today's date.
2. Push a `vX.Y.Z` tag: `git tag vX.Y.Z && git push origin vX.Y.Z`.
3. Create a GitHub release from that tag with auto-generated notes.
4. Move the rolling major tag: `git tag -f vX && git push -f origin refs/tags/vX`.
5. If publishing to the Marketplace, use the "Publish this Action to the GitHub Marketplace" button on the release page.

## Licensing

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).
