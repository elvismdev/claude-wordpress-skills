---
description: Quick WordPress block scan - fast pattern detection for block.json issues, missing useBlockProps, deprecated APIs, and render callback problems across PHP and JS/JSX files
argument-hint: [path]
---

Use the **wp-block-development** skill to perform a quick block development check.

**Target**: $ARGUMENTS (if empty, use current working directory)

Focus only on the "Search Patterns for Quick Detection" section -- run the grep commands to find block development issues fast across both PHP and JavaScript/JSX files. Report matches with file:line references and severity levels. Skip deep analysis.

If critical issues are found, suggest running `/wp-block-review` for comprehensive block review with fix recommendations.
