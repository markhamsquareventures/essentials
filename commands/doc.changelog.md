---
allowed-tools: Read, Write, Glob, Grep, Bash(git log:*), Bash(gh pr:*)
description: Create a changelog entry for a completed epic
---

## Context

- Existing PRDs: !`ls docs/prds/*.md 2>/dev/null | grep -v changelog || echo "No PRDs found"`
- Existing changelogs: !`ls docs/prds/*.changelog.md 2>/dev/null || echo "No changelogs yet"`
- Current branch: !`git branch --show-current`
- Recent commits on branch: !`git log main..HEAD --oneline 2>/dev/null || git log -10 --oneline`

## Your task

Create a changelog documenting what was delivered for an epic.

### Steps

1. **Ask for the epic name** if not provided as an argument

2. **Validate PRD exists**
   - Check that `docs/prds/{epic}.md` exists
   - If not found, abort with: "No PRD found for epic '{epic}'. Changelogs can only be created for existing PRDs."

3. **Read the PRD** for context on the epic's goals and planned scope

4. **Review git commits** to understand what was actually implemented
   - Use `git log` to see commits on the branch
   - Focus on what shipped, not what was planned

5. **Ask for the PR link** (use `gh pr list` to help find it if available)

6. **Generate the changelog** at `docs/prds/{epic}.changelog.md`:

```
# Changelog: {Epic Name}

**Date**: {YYYY-MM-DD}
**PR**: {link to PR}

## Summary

{2-3 sentence description of what this epic delivered}

## Key Changes

- {Most significant change or feature}
- {Another important change}
- {Bug fixes or improvements}
```

### Guidelines

- Focus on what shipped, not what was descoped
- Highlight the most important context, not every change
- Use technical language appropriate for internal team
- Keep it concise and scannable
- Do NOT update the PRD status - that is handled separately
