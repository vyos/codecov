# vyos/codecov

Codecov Global YAML for the `vyos` GitHub organization.

This repo is the canonical source for the file rendered at
[`https://app.codecov.io/account/gh/vyos/yaml`](https://app.codecov.io/account/gh/vyos/yaml).
The dashboard is a cache; this repo's `production` branch is the source of truth.

Sibling repo: [VyOS-Networks/codecov](https://github.com/VyOS-Networks/codecov) for the
VyOS-Networks org. Both repos hold byte-identical `codecov.yml` at design time; any future
delta is enumerated in the per-org delta table below — never as comments in `codecov.yml`
(Codecov strips them on dashboard save).

## Change protocol

1. PRs only. No direct dashboard edits.
2. PRs run the `validate` workflow which POSTs `codecov.yml` to
   `https://codecov.io/validate`. The check must pass.
3. After merge to `production`:
   ```bash
   gh api 'repos/vyos/codecov/contents/codecov.yml?ref=production' \
     -H "Accept: application/vnd.github.raw"
   ```
   To paste, the operator follows the paste protocol below, which fetches this same file
   pinned to one commit SHA as `payload.yml`; that file is the only content to paste
   into the dashboard editor (linked above) and verify.

### Branch governance

This repo carries the enterprise governance tier `central-config`. On `production`:

- Merges are PR-only and need **2 approving reviews**. Stale approvals are dismissed
  when new commits are pushed, and the most recent push must be approved by someone
  other than the person who pushed it.
- The only bypass on this merge gate is Mergify's PR merge path. No human role,
  including org admins, can bypass it.
- The per-repo ruleset `production-required-checks` makes the `validate` check
  required, with no bypass actors.

Plan changes so that two reviewers are available when the PR needs to merge.

## Paste protocol (operator)

The Codecov UI exposes no audit history for the account YAML, so the paste is verified
by comparing content hashes.

1. Resolve the `production` commit SHA once, then fetch the payload and its blob SHA
   from that same commit (so both describe the same content even if `production` moves):
   ```bash
   SRC=$(gh api 'repos/vyos/codecov/commits/production' --jq .sha)
   gh api "repos/vyos/codecov/contents/codecov.yml?ref=$SRC" \
     -H "Accept: application/vnd.github.raw" > payload.yml
   gh api "repos/vyos/codecov/contents/codecov.yml?ref=$SRC" --jq .sha
   ```
   `$SRC` is the source commit SHA; the last command prints the `codecov.yml` blob SHA.
2. Open https://app.codecov.io/account/gh/vyos/yaml in browser.
3. Before pasting, capture the current editor content. It is expected to be empty on
   the first paste; otherwise keep it as the rollback pre-image. Blanking the editor
   and saving is an exact rollback only when the pre-state was empty.
4. Paste `payload.yml` via the clipboard — do not type it (editor auto-indent can
   corrupt YAML). Save.
5. Reload the page, copy the editor content back into a file (for example `saved.yml`),
   and compare it with the payload: `git hash-object saved.yml` must equal the blob SHA
   from step 1. **Codecov strips comments** on save; the hashes match because the file
   is comment-free.
6. Record the source commit SHA, blob SHA, paste timestamp, pre-state, and the
   comparison result in the change ticket.

## Per-org delta (vyos vs VyOS-Networks)

(none at the time of last update — both orgs paste byte-identical `codecov.yml`)

## How this interacts with per-repo codecov.yml

Codecov merges this account-level YAML with each repository's own `codecov.yml`: the
Global YAML is not replaced but updated with the repository YAML, so every key the
repository sets wins and every nested key it omits is inherited from here (see the
[Codecov YAML documentation](https://docs.codecov.com/docs/codecov-yaml)).

This Global YAML sets `informational: true` on the default `project` and `patch`
statuses. A repository that wants a blocking (enforced) Codecov status must set
`informational: false` explicitly in its own `codecov.yml`; omitting the key silently
inherits `informational: true`.

## Language-applicability caveat

The numeric thresholds (`project.threshold: 1%`, `patch.target: 70%`, `patch.threshold: 5%`)
and the `ignore` patterns (`**/*.config.{ts,js,mjs,cjs}`, `**/*.d.ts`, `.next/`, `public/`)
are Next.js-derived from the canary in [VyOS-Networks/next-js-vyos](https://github.com/VyOS-Networks/next-js-vyos).
They are reasonable starting points for JS/TS repos and harmless for non-JS repos (the
`ignore` patterns simply don't match Python/C++/Ansible paths and the thresholds apply to
whatever does upload coverage). The root `tests/` and `scripts/` patterns are also globally
ignored: coverage *of* test code and utility scripts is intentionally excluded from
reporting. A repo whose `scripts/` (or `tests/`) tree holds coverage-bearing product code
should override the `ignore` list in its per-repo `.codecov.yml`. Repos in other languages
that opt into Codecov should likewise override numerics in their per-repo `.codecov.yml`.

## Onboarding a new repo to Codecov coverage

(deferred — see spec §9 follow-up; the playbook lives in `docs/per-repo-onboarding.md` once
the first non-canary opt-in lands)
