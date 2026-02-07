---
description: Quick WordPress plugin standards check - fast pattern detection for plugin architecture issues
argument-hint: [path]
---

Use the **wp-plugin-development** skill to perform a quick plugin standards check.

**Target**: $ARGUMENTS (if empty, use current working directory)

Focus only on the "Search Patterns for Quick Detection" section -- run the grep commands to find plugin architecture issues fast. Report matches with file:line references and severity levels. Skip deep analysis.

If critical issues are found, suggest running `/wp-plugin-review` for comprehensive architecture review with fix recommendations.
