---
allowed-tools: Bash(git:*)
description: Create a new branch
---

## Context

- Current branch: !`git branch --show-current`
- Remote tracking: !`git status -sb`
- Commits on branch: !`git log main..HEAD --oneline`
- Changes vs main: !`git diff main...HEAD --stat`

## Your task

Create a new branch from the current branch branch:

1. Always create a new branch when beginning a new epic.
2. Check to see if there are any changes on the current branch. Abort immediately if there are any and alert the user.
3. Create a new branch using `git checkout -b {branch name}`. Ask for the branch name.
