# Repo context for AI assistants

This repo holds the **Codecov Global YAML** for the `vyos` GitHub organization.

## What lives here
- `codecov.yml` — the canonical, comment-free dashboard-paste payload. Mirror of
  `https://app.codecov.io/account/gh/vyos/yaml`.
- `.github/workflows/validate.yml` — required CI check; POSTs the file to
  `https://codecov.io/validate` on every PR and on pushes to `production`.
- `.mergify.yml` — extends central [vyos/mergify](https://github.com/vyos/mergify).
- `.coderabbit.yaml` — inherits from [vyos/coderabbit](https://github.com/vyos/coderabbit)
  (`inheritance: true` is mandatory).
- `README.md` — change protocol + dashboard-paste discipline.

## Change protocol
1. PRs only. No direct dashboard edits.
2. Keep `codecov.yml` comment-free — Codecov strips comments on dashboard save; any
   explanatory content lives here in AGENTS.md or in README.md.
3. After merge to `production`, operator pastes from
   `gh api 'repos/vyos/codecov/contents/codecov.yml?ref=production' -H "Accept: application/vnd.github.raw"`
   into the dashboard. Repo is the source of truth; dashboard is a cache.
