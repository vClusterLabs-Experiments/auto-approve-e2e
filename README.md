# auto-approve-e2e

End-to-end scenario coverage for `loft-sh/github-actions/.github/workflows/auto-approve-bot-prs.yaml@main`.

## Invariants

- The reusable workflow **must never hard-fail** a caller. For every scenario this
  harness asserts `conclusion ∈ {success, skipped, neutral}` and fails the e2e
  job otherwise.
- The eligibility decision table (title / branch / author) matches the spec.

## Scenarios (matrix)

| name | title | branch | expected eligible |
|---|---|---|---|
| chore-deps-title | `chore(deps): bump bar` | `test/chore-deps-*` | true |
| fix-deps-title | `fix(deps): patch cve` | `test/fix-deps-*` | true |
| backport-branch | any | `backport/e2e-*` | true |
| renovate-branch | any | `renovate/e2e-*` | true |
| update-platform-version-branch | any | `update-platform-version-*` | true |
| ineligible-feat | `feat: new thing` | `test/feat-*` | false |
| ineligible-random-title | `random commit message` | `test/random-*` | false |

All scenarios use `github-actions[bot]` as author and token — exercising the
self-approve graceful-skip path (the production regression that motivated this
harness).

## Running

```
gh workflow run e2e.yaml --repo vClusterLabs-Experiments/auto-approve-e2e
```

Scheduled to run weekly. Each scenario creates a PR, waits for
`caller-auto-approve-bot-prs.yaml` to complete, asserts invariants, then closes
the PR and deletes the branch.
