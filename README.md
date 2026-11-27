# claude-review

Central, independent AI code review for every product repository (Livappy, Jobappy, AppySro).

**Rule (owner decision, 19. 11. 2026):** a PR written with AI is reviewed by a *different* AI session. The reviewer sees only the diff and the repository, never the author's reasoning (no PR description, comments or commit messages). It posts inline comments and one `## Claude review` summary. It cannot approve, merge or push. The human answers the business questions in the PR and merges.

The review itself lives in [`.github/workflows/claude-review.yml`](.github/workflows/claude-review.yml). Change it here once and every repository uses the new version on its next PR.

This repository is **public** on purpose: GitHub lets repositories of other owners call a reusable workflow only from a public repository. It contains no secrets.

## Add the review to a repository

1. Add the secret `ANTHROPIC_API_KEY` (organization secret, or repository secret on a personal account).
2. Add `.github/workflows/claude-review.yml` to the repository:

```yaml
name: Claude review

on:
  # pull_request (not pull_request_target): fork PRs get no secrets.
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]

concurrency:
  group: claude-review-${{ github.event.pull_request.number }}
  cancel-in-progress: true

permissions: {}

jobs:
  review:
    # Skip drafts and PRs from forks.
    if: >-
      github.event.pull_request.draft == false &&
      github.event.pull_request.head.repo.full_name == github.repository
    permissions:
      contents: read
      pull-requests: write
    uses: AppySro/claude-review/.github/workflows/claude-review.yml@main
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

3. Optional (paid plan): require the **Claude review / Claude review** check in branch protection.

## Changing the review

Every change here reaches all repositories, so it goes through a PR like any other change, and the review of that PR runs too.
