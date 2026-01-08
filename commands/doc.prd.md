---
allowed-tools: Read, Write, Glob, Grep, AskUserQuestion
description: Create a PRD for a new epic
---

## Context

- Existing PRDs: !`ls docs/prds/*.md 2>/dev/null || echo "No PRDs found"`
- Current branch: !`git branch --show-current`

## Your task

Create a PRD for a new epic through an interview process.

### Steps

1. **Ask for epic name** if not provided as argument

2. **Determine next epic number**
   - Parse existing PRDs in `docs/prds/` to find the highest epic number
   - Increment by 1 for the new epic

3. **Interview for PRD content** - Ask the user about:
   - **Objective** - One-line summary of what this epic accomplishes and why
   - **Dependencies** - Any epics or features this depends on
   - **Data Model** - Tables/fields needed (skip if none required)
   - **User Stories** - What users can do and why it matters
   - **Tasks** - Breakdown of implementation work with files to modify
   - **Open Questions** - Decisions needed before/during implementation

4. **Create PRD** at `docs/prds/{epic-name}.md` using this template:

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

### Guidelines

- Keep the interview conversational but focused
- Skip Data Model section if the epic doesn't require new tables
- Tasks should identify specific files to create/modify when possible
- Open Questions should capture uncertainty that blocks implementation
