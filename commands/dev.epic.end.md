---
allowed-tools: Read, Write, Glob, Grep, Bash(git:*), Skill
description: Complete an epic (checks, docs, PR)
---

## Context

- Current branch: !`git branch --show-current`
- Working directory status: !`git status --short`
- Remote tracking: !`git status -sb`
- Existing PRDs: !`ls docs/prds/*.md 2>/dev/null || echo "No PRDs found"`

## Your task

Complete an epic by running all end-of-epic steps.

### Steps

1. **Ask for the epic name** if not provided as an argument

2. **Validate PRD exists**

   - Check that `docs/prds/{epic}.md` exists
   - If not found, abort with: "No PRD found for epic '{epic}'."

3. **Pre-flight checks** - Abort if any fail:

   - Working directory is clean (no uncommitted changes)
   - Not on main branch
   - Branch is up to date with remote (if tracking)

4. **Run checks** by running `/dev.check`

   - If any check fails, stop and report the failures

5. **Document the epic** by running `/doc.epic {epic}`

6. **Commit documentation** by running `/dev.commit`

7. **Mark PRD as complete**

   - Update the PRD file's status from "In Progress" or "Draft" to "Complete"

8. **Create pull request** by running `/dev.pr`

### On Failure

If any step fails, stop immediately and report:

- Which step failed
- The error output
- Suggested next steps to fix
