---
description: Quick WooCommerce scan - fast pattern detection for HPOS compatibility issues, payment security anti-patterns, cart fragments loading, template override problems, and direct database access
argument-hint: [path]
---

Use the **wp-woocommerce-dev** skill to perform a quick WooCommerce check.

**Target**: $ARGUMENTS (if empty, use current working directory)

Focus only on the "Search Patterns for Quick Detection" section -- run the grep commands to find WooCommerce development issues fast. Report matches with file:line references and severity levels. Skip deep analysis.

If critical issues are found, suggest running `/wp-woo-review` for comprehensive WooCommerce review with fix recommendations.
