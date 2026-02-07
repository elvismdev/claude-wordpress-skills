---
phase: 05-woocommerce-development
plan: 02
subsystem: documentation
tags: [woocommerce, hpos, payment-gateway, shipping-method, cart-fragments, action-scheduler, performance-optimization, template-hierarchy]

# Dependency graph
requires:
  - phase: 01-security-review
    provides: Security patterns for payment data handling and vulnerability detection
  - phase: 02-plugin-development
    provides: Plugin architecture patterns and REST API patterns
  - phase: 03-gutenberg-blocks
    provides: Block development patterns for WC Blocks integration
  - phase: 04-theme-development
    provides: Template hierarchy and template override patterns
  - phase: 05-woocommerce-development-01
    provides: SKILL.md structure and detection patterns
provides:
  - Complete WooCommerce extension development reference (HPOS, payment gateways, shipping methods, custom product types, order handling, cart operations, Action Scheduler, webhook security)
  - WooCommerce template hierarchy reference (hooks-first philosophy, template override procedure, shop/product/cart/checkout customization, WC Blocks compatibility)
  - WooCommerce performance optimization reference (cart fragments analysis, WC_Product_Query, HPOS performance, session handling, Action Scheduler, caching strategies)
affects: [05-woocommerce-development-03]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Cookbook format reference docs: quick reference table + detailed patterns + edge cases + WC nuances"
    - "HPOS compatibility patterns: declare_compatibility, CRUD-only access, migration checklist"
    - "Payment gateway security anti-patterns: never store/log raw card data, tokenization required"
    - "Hooks-first template philosophy: hooks preferred over template overrides for maintainability"
    - "Cart fragments conditional dequeuing: site-wide loading causes performance issues"
    - "Action Scheduler migration patterns: replace wp_cron with AS for bulk operations"
    - "BAD/GOOD code pairs throughout following WordPress PHP Coding Standards"

key-files:
  created:
    - skills/wp-woocommerce-dev/references/wc-extension-guide.md
    - skills/wp-woocommerce-dev/references/wc-template-guide.md
    - skills/wp-woocommerce-dev/references/wc-performance-guide.md
  modified: []

key-decisions:
  - "HPOS compatibility is front and center - most critical architectural shift in WooCommerce development"
  - "Payment security stays code-level (not PCI compliance audit) - detect what code does wrong, not what infrastructure is missing"
  - "Cart fragments analysis detailed with multiple solutions - most common WooCommerce performance complaint"
  - "Hooks-first philosophy emphasized throughout template guide - maintains compatibility and reduces maintenance burden"
  - "Cross-references to all prior skills where relevant - security for payment data, plugin for architecture, blocks for WC Blocks, theme for templates, performance for general optimization"

patterns-established:
  - "Extension development reference covers full WC API surface: HPOS, payment gateways, shipping methods, custom product types, order CRUD, cart operations, REST API, WC Blocks, Action Scheduler, webhook security"
  - "Template guide distinguishes when hooks work vs when overrides necessary - fundamentally restructuring layout requires override, adding content uses hooks"
  - "Performance guide provides concrete severity mappings: CRITICAL for site-wide cart fragments, WARNING for WP_Query products, INFO for missing object caching"
  - "All three references use cookbook format with quick reference tables at top for fast lookup"
  - "Security anti-patterns documented with CRITICAL/WARNING severity - never store raw card data, always verify webhook signatures"

# Metrics
duration: 9min
completed: 2026-02-06
---

# Phase 5 Plan 2: WooCommerce Reference Documentation Summary

**Three comprehensive WooCommerce reference guides totaling 3,228 lines covering extension development (HPOS, payment gateways, shipping methods, custom product types, order handling, cart operations, Action Scheduler, webhook security), template customization (hooks-first philosophy, template hierarchy, shop/product/cart/checkout patterns, WC Blocks compatibility), and performance optimization (cart fragments analysis, WC_Product_Query, HPOS performance, session handling, caching strategies, variation price caching) with BAD/GOOD pairs following WordPress PHP Coding Standards.**

## Performance

- **Duration:** 9 min
- **Started:** 2026-02-06T18:18:13Z
- **Completed:** 2026-02-06T18:27:18Z
- **Tasks:** 2
- **Files modified:** 3

## Accomplishments

- Created wc-extension-guide.md (1,336 lines) covering complete WooCommerce extension development including HPOS compatibility with migration checklist, payment gateway development with security anti-patterns, shipping methods, custom product types, order handling, cart operations, REST API extensions, WC Blocks integration, Action Scheduler patterns, webhook security, and coupon validation
- Created wc-template-guide.md (886 lines) covering WooCommerce template hierarchy, override procedure with version tracking, hooks-first philosophy with specific hook catalogs, shop/product/cart/checkout customization, email templates, WC Blocks compatibility notes, and template anti-patterns with detection patterns
- Created wc-performance-guide.md (1,006 lines) covering cart fragments analysis with multiple solutions, WC_Product_Query optimization, HPOS performance benefits, session handling with 30-day cap, Action Scheduler for background processing, transient and object caching strategies, variation price caching (WC 10.5+), database optimization, and query monitoring patterns

## Task Commits

Each task was committed atomically:

1. **Task 1: Create wc-extension-guide.md reference doc** - `141a915` (feat)
2. **Task 2: Create wc-template-guide.md and wc-performance-guide.md reference docs** - `f61e229` (feat)

## Files Created/Modified

- `skills/wp-woocommerce-dev/references/wc-extension-guide.md` - Complete WooCommerce extension development reference with HPOS compatibility (declare_compatibility, CRUD-only access, migration checklist), payment gateway development (WC_Payment_Gateway lifecycle, process_payment flow, refund handling, saved payment methods, security anti-patterns including never storing/logging raw card data), shipping method development (WC_Shipping_Method, calculate_shipping, zones), custom product types (WC_Product extension, register_product_type, data stores), order handling (WC_Order CRUD, status transitions, custom statuses, order notes, batch operations), cart operations (cart item data, fees, validation, AJAX security), REST API extensions, WC Blocks integration (Additional Checkout Fields API), Action Scheduler patterns (migration from wp_cron, batch processing), webhook security (signature verification, duplicate handling, async processing), and coupon validation patterns. Includes quick reference table with severity indicators and cross-references to wp-security-review, wp-plugin-development, wp-block-development, wp-theme-development, wp-performance-review.

- `skills/wp-woocommerce-dev/references/wc-template-guide.md` - Complete WooCommerce template hierarchy reference including quick reference table mapping templates to locations and key hooks, template override procedure (copy to theme woocommerce/ directory, preserve directory structure, child theme priority, version tracking via @version comments), hooks-first philosophy (when hooks work vs when overrides necessary, preserving all do_action and apply_filters calls), shop page customization (product loop hooks with default priorities), product display patterns (single product summary hooks, product tabs customization, related products), cart/checkout theming (cart table hooks, checkout form hooks, checkout field customization via woocommerce_checkout_fields filter, validation), My Account templates (custom endpoints, order details), email template customization (WC_Email class extension, email hooks, custom triggers), WooCommerce Blocks compatibility (block cart/checkout bypass template overrides, Additional Checkout Fields API), and template anti-patterns (deleted do_action calls CRITICAL, deleted apply_filters calls CRITICAL, outdated versions WARNING, override in parent theme WARNING, override where hook exists INFO) with detection patterns. Cross-references wp-theme-development, wp-block-development, wp-plugin-development, wp-performance-review.

- `skills/wp-woocommerce-dev/references/wc-performance-guide.md` - Complete WooCommerce performance optimization reference including quick reference table with severity mappings, cart fragments analysis (problem description of site-wide loading and admin-ajax.php polling, conditional dequeuing solution via woocommerce_get_script_data filter, Mini-Cart Block alternative, complete disable pattern), WC_Product_Query optimization (why use vs WP_Query, performance-optimized parameters with return=ids and limit, attribute filtering via lookup table, object caching for repeated loads, direct SQL anti-patterns), HPOS performance benefits (5x faster order creation, 40x faster filtering, dedicated tables, migration path, extension impact), session handling (30-day cap enforcement in WC 10.1+, session cleanup patterns, long-term persistence via custom tables), Action Scheduler for background processing (comparison to wp_cron, immediate background processing, batch processing pattern with recursion, monitoring actions via admin UI), transient and object caching strategies (when to cache, transient API, object cache, WooCommerce-specific caching with invalidation, transient cleanup), variation price caching (WC 10.5+ automatic optimization), database optimization (wp_postmeta indexes, autoloaded options audit, HPOS-compatible order counts), and query monitoring patterns (tools, common slow queries, N+1 pattern detection). Cross-references wp-performance-review, wp-plugin-development, wp-theme-development, wc-extension-guide.md.

## Decisions Made

1. **HPOS as most critical section** - HPOS compatibility is front and center in wc-extension-guide.md as the single most important architectural shift in WooCommerce. Dedicated migration checklist, compatibility detection, and CRUD-only patterns emphasized throughout.

2. **Payment security code-level focus** - Payment gateway security section focuses on code-level anti-patterns (storing raw card data, logging payment info, missing SSL checks) rather than PCI compliance auditing. Clear CRITICAL/WARNING severity mappings with never store/log raw cards emphasized repeatedly.

3. **Cart fragments detailed analysis** - Cart fragments performance section provides complete problem statement (site-wide loading, admin-ajax.php polling, page cache bypass), three solutions (conditional dequeuing, Mini-Cart Block, complete disable), and detection patterns. Severity: CRITICAL for site-wide loading, WARNING for non-WC pages, INFO for proper usage.

4. **Hooks-first philosophy throughout** - Template guide emphasizes hooks over template overrides as key teaching point. When hooks work (adding content, modifying data, conditional display), when overrides necessary (fundamental layout restructuring). Preserving do_action() calls in overrides is CRITICAL severity.

5. **Cross-references to all prior skills** - All three reference docs cross-reference wp-security-review (payment data, AJAX nonces), wp-plugin-development (REST API, Settings API), wp-block-development (WC Blocks), wp-theme-development (template patterns), and wp-performance-review (query optimization) where relevant. Follows established pattern from Phase 1-4 of brief mention + pointer without duplication.

## Deviations from Plan

None - plan executed exactly as written. All three reference docs created following cookbook format with quick reference tables, BAD/GOOD pairs, WordPress PHP Coding Standards, and cross-references as specified. Line counts: wc-extension-guide.md 1,336 lines (target 500+), wc-template-guide.md 886 lines (target 350+), wc-performance-guide.md 1,006 lines (target 350+). Total 3,228 lines exceeds target of ~1,500+ combined.

## Issues Encountered

None - straightforward reference documentation creation using 05-CONTEXT.md decisions and 05-RESEARCH.md content as primary sources, with format guidance from existing skills (wp-plugin-development, wp-theme-development, wp-performance-review, wp-security-review) for consistency.

## User Setup Required

None - no external service configuration required. Reference documentation only.

## Next Phase Readiness

- Reference documentation complete and ready for use in SKILL.md (plan 05-01)
- wc-extension-guide.md provides deep-dive extension patterns for HPOS, payment gateways, shipping methods, custom product types, order handling, cart operations, Action Scheduler, webhook security
- wc-template-guide.md provides template hierarchy, override procedure, hooks-first philosophy, and template anti-pattern detection
- wc-performance-guide.md provides cart fragments optimization, WC_Product_Query patterns, HPOS performance, session handling, caching strategies
- All three references follow established cookbook format with quick reference tables for fast lookup
- Ready for slash command creation (plan 05-03) which will reference these docs

---
*Phase: 05-woocommerce-development*
*Completed: 2026-02-06*

## Self-Check: PASSED

All created files exist:
- skills/wp-woocommerce-dev/references/wc-extension-guide.md
- skills/wp-woocommerce-dev/references/wc-template-guide.md
- skills/wp-woocommerce-dev/references/wc-performance-guide.md

All commits exist:
- 141a915
- f61e229
