---
description: Quick WordPress performance scan - fast triage using critical pattern detection
argument-hint: [path]
---

Use the **wp-performance-review** skill to perform a quick triage scan.

**Target**: $ARGUMENTS (if empty, use current working directory)

Focus only on the "Search Patterns for Quick Detection" section—run the grep commands to find critical issues fast. Report matches with file:line references and severity levels. Skip deep analysis.

If critical issues are found, suggest running `/wp-perf-review` for comprehensive analysis with fixes.