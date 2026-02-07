---
description: Quick WordPress security scan - fast triage using high-risk pattern detection
argument-hint: [path]
---

Use the **wp-security-review** skill to perform a quick security triage scan.

**Target**: $ARGUMENTS (if empty, use current working directory)

Focus only on the "Search Patterns for Quick Detection" section—run the grep commands to find high-risk security patterns fast. Report matches with file:line references and severity levels. Skip deep analysis.

If critical issues are found, suggest running `/wp-sec-review` for comprehensive analysis with CWE references and fix recommendations.
