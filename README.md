# claude-review

Central, independent AI code review for the repositories of **AppySro** (appysk, signaldesk, appybuddy, MASPO-OS, maspo-web, and future Appy projects).

Livappy and Jobappy are separate businesses and keep their own copy of this review; they do not call this repository (owner decision, 27. 11. 2026).

**Rule (owner decision, 19. 11. 2026):** a PR written with AI is reviewed by a *different* AI session. The reviewer sees only the diff and the repository, never the author's reasoning (no PR description, comments or commit messages). It posts inline comments and one `## Claude review` summary. It cannot approve, merge or push. The human answers the business questions in the PR and merges.

The review itself lives in [`.github/workflows/claude-review.yml`](.github/workflows/claude-review.yml). Change it here once and every repository uses the new version on its next PR.

This repository is **public** for now: the Appy repositories still live on a personal account, and GitHub lets another owner call a reusable workflow only from a public repository. It contains no secrets. Once all Appy repositories are in AppySro, it can be made private and shared with the organization (Settings → Actions → Access), if the plan allows it.

## Add the review to a repository

1. Add **one** credential as a secret (organization secret, or repository secret on a personal account or for private repositories on GitHub Free):
   - `ANTHROPIC_API_KEY`: an API key from Claude Console. Billed per token. Use one Console workspace per company, with a spend limit.
   - `CLAUDE_CODE_OAUTH_TOKEN`: a Claude Pro/Max subscription token, created locally with `claude setup-token`. No per-token bill, but reviews share the subscription's usage limits with your own Claude work, and the token is tied to one person's account.

   Set only one of them per repository.
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
    # Skip drafts, PRs from forks, and Dependabot (it gets no Actions secrets).
    if: >-
      github.event.pull_request.draft == false &&
      github.event.pull_request.head.repo.full_name == github.repository &&
      github.actor != 'dependabot[bot]'
    permissions:
      contents: read
      pull-requests: write
    uses: AppySro/claude-review/.github/workflows/claude-review.yml@main
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

3. Optional (paid plan): require the **review / Claude review** check (caller job / called job) in branch protection.

## Changing the review

Every change here reaches all repositories, so it goes through a PR like any other change, and the review of that PR runs too.
