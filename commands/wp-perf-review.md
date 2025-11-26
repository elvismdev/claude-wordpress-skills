---
description: WordPress performance code review - detects database anti-patterns, caching issues, hook problems, and scalability concerns
argument-hint: [file-or-directory]
---

Perform a WordPress performance code review using the **wp-performance-review** skill.

**Target**: $ARGUMENTS (if empty, use current working directory)

## Instructions

1. **Load the skill**: Read `.claude/skills/wp-performance-review/SKILL.md` (or from `~/.claude/skills/`)

2. **Execute the Code Review Workflow** from the skill:
   - Step 1: Identify file types in the target
   - Step 2: Run the "Search Patterns for Quick Detection" (grep commands) to find critical issues first
   - Step 3: Apply "File-Type Specific Checks" based on what files exist
   - Step 4: Check the "Quick Reference: Critical Anti-Patterns" for pattern matching
   - Step 5: Consider "Platform Context" if hosting environment is known

3. **Load references as needed**: If you find issues requiring deeper analysis, load:
   - `references/anti-patterns.md` — Full catalog of 45+ anti-patterns with fixes
   - `references/wp-query-guide.md` — WP_Query optimization details
   - `references/caching-guide.md` — Caching implementation patterns

4. **Format output** using the skill's "Output Format" section:
   - Group by severity: Critical → Warnings → Recommendations
   - Include line numbers for each issue
   - Provide fix suggestions
   - End with summary and estimated impact

Report all findings with the severity levels defined in the skill (Critical/Warning/Info).
