---
allowed-tools: Read, Write, Glob, Grep, Task, AskUserQuestion, EnterPlanMode
description: Start a new epic
---

## Context

- Existing PRDs: !`ls docs/prds/`
- Current branch: !`git branch --show-current`

## Your task

Start a new epic:

1. Ask the user for the epic name and brief description
2. Create a new branch with `dev.branch`. Use the epic title as the branch name.
3. Create a PRD in `docs/prds/` using the following template:

```
# Epic N: [Epic Name]

**Status:** Draft

## Objective

[One-line summary of what this epic accomplishes and why it matters.]

## Dependencies

- [List any epics or features this depends on]

## Data Model

### [ModelName]

| Field | Type | Notes |
|-------|------|-------|
| id | bigint | Primary key |
| ... | ... | ... |
| timestamps | | created_at, updated_at |

## User Stories

1. **[Action]** - As a user, I can [do something] so that [benefit]
2. ...

## Tasks

### Task 1: [Task Name]
**Files to create/modify:**
- `path/to/file.php`

### Task 2: [Task Name]
...

## Open Questions

1. [Any decisions that need to be made before or during implementation]
```

4. Enter plan mode to design the implementation approach

The PRD should capture scope, requirements, and success criteria before planning begins.
