---
description: WordPress theme development review - validates theme.json v3 structure, block theme templates, template parts, global styles, style variations, block patterns, and classic-to-block migration opportunities for WP 6.6+
argument-hint: [file-or-directory]
---

Use and follow the **wp-theme-development** skill to perform a comprehensive WordPress theme development review.

**Target**: $ARGUMENTS (if empty, use current working directory)

Execute the full Code Review Workflow from the skill, starting with theme type detection (block/classic/hybrid/child). Load reference files as needed for deeper analysis, and format output using the skill's Output Format section with severity levels (Critical/Warning/Info). Review PHP, HTML template, and JSON files. If security concerns are found, suggest running `/wp-sec-review`. If plugin architecture issues are found, suggest running `/wp-plugin-review`. If block markup issues are found, suggest running `/wp-block-review`.
