---
allowed-tools: Read, Write, Glob, Grep, Bash(git log:*), Bash(gh pr:*), Skill
description: Document a completed epic (changelog + learnings)
---

## Context

- Existing PRDs: !`ls docs/prds/*.md 2>/dev/null || echo "No PRDs found"`
- Existing changelogs: !`ls docs/changelog/*.md 2>/dev/null || echo "No changelogs yet"`
- Current branch: !`git branch --show-current`
- Recent commits on branch: !`git log main..HEAD --oneline 2>/dev/null || git log -10 --oneline`

## Your task

Document what was delivered for an epic.

### Steps

1. **Ask for the epic name** if not provided as an argument

2. **Validate PRD exists**
   - Check that `docs/prds/{epic}.md` exists
   - If not found, abort with: "No PRD found for epic '{epic}'. Documentation can only be created for existing PRDs."

3. **Create the changelog** by running `/doc.changelog {epic}`

4. **Generate learnings**
   - Read the PRD to understand planned scope
   - Review git commits and code changes to see what was actually implemented
   - Focus on HOW things were done, not just what
   - Identify insights about:
     - Technical decisions made
     - Patterns used or established
     - Challenges overcome
     - Things that worked well or could be improved

5. **Append learnings to `docs/learnings.md`**
   - Add bullet points directly (no section header, no date)
   - Keep learnings concise and actionable

### Guidelines

- Focus on learnings that will help future work
- Capture technical decisions and their rationale
- Note any patterns or conventions established
- Keep bullet points concise
