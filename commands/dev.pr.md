---
allowed-tools: Bash(git:*), Bash(gh:*)
description: Create a pull request
---

## Context

- Current branch: !`git branch --show-current`
- Remote tracking: !`git status -sb`
- Commits on branch: !`git log main..HEAD --oneline`
- Changes vs main: !`git diff main...HEAD --stat`

## Your task

Create a pull request for the current branch:

1. Push the branch if not already pushed
2. Create a PR using `gh pr create` with:
   - Clear title summarizing the changes
   - Body with summary bullets and test plan
3. Return the PR URL
