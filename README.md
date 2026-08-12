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
   Operator pastes this content into the dashboard editor (linked above), saves,
   and re-opens the tab to confirm the saved content matches.

## Paste protocol (operator)

1. Open https://app.codecov.io/account/gh/vyos/yaml in browser.
2. Fetch the merged file via the quoted `gh api` raw-content command (above).
3. Paste into the dashboard editor.
4. Save.
5. Re-open the dashboard tab. **Codecov strips comments** — the saved content
   should match the paste content because the file is comment-free.
6. Capture the Codecov change-history audit-log row (timestamp, user, prior version)
   to the IS ticket as evidence.
7. Record merge commit SHA + paste timestamp + audit-log reference in the IS ticket.

## Per-org delta (vyos vs VyOS-Networks)

(none at the time of last update — both orgs paste byte-identical `codecov.yml`)

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

