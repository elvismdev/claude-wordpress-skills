---
description: WooCommerce development review - validates HPOS compatibility, payment gateway security, WooCommerce CRUD patterns, hook usage, template override quality, cart fragments performance, and Action Scheduler patterns for WC 8.2+
argument-hint: [file-or-directory]
---

Use and follow the **wp-woocommerce-dev** skill to perform a comprehensive WooCommerce development review.

**Target**: $ARGUMENTS (if empty, use current working directory)

Execute the full Code Review Workflow from the skill, starting with WooCommerce context detection (extension/theme integration/payment gateway/shipping method/custom product type/WC Blocks). Load reference files as needed for deeper analysis, and format output using the skill's Output Format section with severity levels (Critical/Warning/Info). If security concerns are found, suggest running `/wp-sec-review`. If plugin architecture issues are found, suggest running `/wp-plugin-review`. If block-related issues are found, suggest running `/wp-block-review`. If template/theme issues are found, suggest running `/wp-theme-review`. If performance issues are found, suggest running `/wp-perf-review`.
