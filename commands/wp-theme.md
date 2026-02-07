---
description: Quick WordPress theme scan - fast pattern detection for theme.json issues, missing required files, hardcoded styles, and template hierarchy problems across PHP, HTML, and JSON files
argument-hint: [path]
---

Use the **wp-theme-development** skill to perform a quick theme check.

**Target**: $ARGUMENTS (if empty, use current working directory)

Focus only on the "Search Patterns for Quick Detection" section -- run the grep commands to find theme development issues fast across PHP, HTML template, and JSON files. Report matches with file:line references and severity levels. Skip deep analysis.

If critical issues are found, suggest running `/wp-theme-review` for comprehensive theme review with fix recommendations.
