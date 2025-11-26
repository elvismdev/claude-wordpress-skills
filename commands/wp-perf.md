---
description: Quick WordPress performance scan - runs critical pattern detection first
argument-hint: [path]
---

Perform a **quick** WordPress performance scan using the **wp-performance-review** skill.

**Target**: $ARGUMENTS (if empty, use current working directory)

## Quick Scan Instructions

1. **Load the skill**: Read the wp-performance-review SKILL.md

2. **Run "Search Patterns for Quick Detection" only**:
   - Execute the grep commands from the skill to find critical issues fast
   - Focus on CRITICAL severity patterns first (unbounded queries, session_start, query_posts, polling)

3. **Report findings immediately**:
   - List any matches with file:line references
   - Note severity level for each match
   - Skip deep analysis unless critical issues are found

4. **If critical issues found**: Suggest running full `/wp-perf-review` for detailed analysis

This is a fast triage scan. For comprehensive review with fixes, use `/wp-perf-review` instead.
