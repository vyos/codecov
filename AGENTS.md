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

Companion repo: [VyOS-Networks/codecov](https://github.com/VyOS-Networks/codecov) holds the Global YAML for
the `VyOS-Networks` org; both repos carry a byte-identical `codecov.yml`.

## Change protocol
1. PRs only. The only allowed dashboard change is pasting the merged, pinned `production`
   payload (step 3); no ad-hoc or manual edits in the dashboard editor. This repo carries
   the enterprise governance tier `central-config`: merges to `production` need 2
   approving reviews (stale approvals are dismissed on push), the only bypass on that
   merge gate is Mergify's PR merge path (no human bypass), and the per-repo ruleset
   `production-required-checks` makes `validate` required. Plan changes so two reviewers
   are available.
2. Keep `codecov.yml` comment-free — Codecov strips comments on dashboard save; any
   explanatory content lives here in AGENTS.md or in README.md.
3. After merge to `production`, resolve the commit SHA once and fetch the pinned payload:
   `SRC=$(gh api 'repos/vyos/codecov/commits/production' --jq .sha)`, then
   `gh api "repos/vyos/codecov/contents/codecov.yml?ref=$SRC" -H "Accept: application/vnd.github.raw" > payload.yml`.
   The operator pastes `payload.yml` into the dashboard. Repo is the source of truth;
   dashboard is a cache.
4. Verify the paste by content hash — the Codecov UI exposes no audit history for the
   account YAML. Capture the pre-paste editor content (rollback pre-image), paste via
   the clipboard (never type it), reload, copy the editor content back, and check that
   `git hash-object` of the copy equals the `codecov.yml` blob SHA. Read the blob SHA at the
   same `$SRC` as the payload (step 3), not from the moving `production` ref.
   Record source commit SHA, blob SHA, paste timestamp, pre-state, and the result in the
   change ticket. Full steps: README.md "Paste protocol (operator)".
5. Codecov merges this Global YAML with each repository's own `codecov.yml` (repository
   values win; nested keys the repository omits are inherited from here). Statuses here
   are `informational: true`, so a repository that wants a blocking status must set
   `informational: false` explicitly. See README.md "How this interacts with per-repo
   codecov.yml".
