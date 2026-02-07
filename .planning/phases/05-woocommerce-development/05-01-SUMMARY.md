---
phase: 05-woocommerce-development
plan: 01
subsystem: woocommerce
tags: [woocommerce, hpos, crud, payment-gateway, shipping, cart-fragments, action-scheduler, template-overrides, wc-blocks]

# Dependency graph
requires:
  - phase: 01-security-review
    provides: Security patterns (payment data handling, REST API auth, escaping)
  - phase: 02-plugin-development
    provides: Plugin architecture patterns (WC as extension, hooks, Settings API)
  - phase: 03-gutenberg-blocks
    provides: Block development patterns (WC Blocks integration, Store API)
  - phase: 04-theme-development
    provides: Template patterns (WC template overrides, hooks-first philosophy)
  - phase: performance-review
    provides: Performance patterns (cart fragments, query optimization, caching)
provides:
  - WooCommerce extension review skill covering HPOS compatibility, CRUD patterns, payment gateways, shipping methods, custom product types, cart operations, template overrides, and performance optimization
  - Context-aware detection for 6 WooCommerce contexts (extension, theme, gateway, shipping, product type, WC Blocks)
  - Cross-references to all 5 prior skills for comprehensive WC review
affects: [06-infrastructure, marketplace-registration]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "HPOS compatibility detection: declare_compatibility() in before_woocommerce_init hook"
    - "CRUD-first data access: wc_get_orders(), wc_get_product(), WC_Order/WC_Product methods"
    - "Payment gateway security: never store raw cards, tokenization, HTTPS checks"
    - "Cart fragments optimization: conditional dequeuing, Mini-Cart Block alternative"
    - "Template override hooks preservation: all do_action() calls must be kept"
    - "Action Scheduler for background processing: replaces wp_cron for WC tasks"
    - "Webhook security: HMAC-SHA256 verification with hash_equals()"

key-files:
  created:
    - skills/wp-woocommerce-dev/SKILL.md
  modified: []

key-decisions:
  - "HPOS compatibility as primary focus—CRITICAL if missing, breaks modern WC stores"
  - "Code-level payment security only—NOT PCI compliance audit (infrastructure-level)"
  - "Cart fragments still major performance issue in 2026 despite WC 7.8 improvements"
  - "Hooks-first philosophy for template customization (key teaching point)"
  - "Most cross-referential skill—touches all 5 prior skills (security, plugin, blocks, themes, performance)"
  - "WP_Query for products flagged as WARNING (future-breaking, custom product tables planned)"
  - "Surface-level coverage for WC Blocks, REST API, coupons, sessions, emails (scan patterns + brief guidance)"

patterns-established:
  - "WooCommerce context auto-detection: extension/theme/gateway/shipping/product type/WC Blocks determine review depth"
  - "HPOS migration checklist: declare_compatibility, CRUD-only access, compatibility mode awareness"
  - "Payment gateway lifecycle: WC_Payment_Gateway class, process_payment() flow, refund handling"
  - "Template override version tracking: @version comment comparison, outdated template detection"
  - "12-section skill structure: consistent with all prior skills (Overview, When to Use, Workflow, File Checks, Search Patterns, Context Detection, Quick Reference, Severity, Output Format, Common Mistakes, Version Compatibility, Deep-Dive References)"

# Metrics
duration: 5min
completed: 2026-02-06
---

# Phase 5 Plan 01: WooCommerce Development SKILL.md Summary

**WooCommerce development review skill covering HPOS compatibility (declare_compatibility, CRUD-only access), payment gateway security (never store raw cards), cart fragments optimization (conditional loading), and template override quality (hooks preservation) for WC 8.2+ through 10.x**

## Performance

- **Duration:** 5 min
- **Started:** 2026-02-07T02:17:54Z
- **Completed:** 2026-02-07T02:23:35Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments

- Created comprehensive WooCommerce development review skill (1,604 lines) with 12 sections mirroring all prior skill structure
- HPOS compatibility front and center—declare_compatibility() detection, CRUD-only validation, direct post access flagged as CRITICAL
- Payment gateway security patterns—never store/log raw card data, HTTPS checks, webhook signature verification
- Cart fragments performance optimization—site-wide loading flagged as CRITICAL, conditional dequeuing patterns
- Template override anti-patterns—deleted hooks flagged as CRITICAL, hooks-first philosophy emphasized
- WooCommerce context auto-detection for 6 contexts (extension, theme integration, payment gateway, shipping method, custom product type, WC Blocks)
- Cross-references to all 5 prior skills (wp-security-review, wp-plugin-development, wp-block-development, wp-theme-development, wp-performance-review)
- BAD/GOOD code pairs throughout following WordPress PHP Coding Standards
- Grep patterns for quick detection organized by severity (CRITICAL/WARNING/INFO)
- Deep coverage for HPOS, CRUD, payment gateways, shipping, product types, hooks, cart operations, cart fragments, Action Scheduler, webhooks, template overrides
- Surface-level coverage for WC Blocks, REST API, coupons, sessions, emails, variations, product caching
- Common false-positive table with 11 WC-specific scenarios
- Version compatibility reference for WC 3.0 through 10.5
- Links to 3 reference docs (wc-extension-guide.md, wc-template-guide.md, wc-performance-guide.md)

## Task Commits

Each task was committed atomically:

1. **Task 1: Create SKILL.md with YAML frontmatter and 12-section structure** - `1ddf2f1` (feat)

## Files Created/Modified

- `skills/wp-woocommerce-dev/SKILL.md` - Complete WooCommerce development review skill with HPOS compatibility, CRUD patterns, payment gateway security, shipping methods, custom product types, WooCommerce hooks catalog, order/cart operations, cart fragments performance, Action Scheduler patterns, webhook security, and template override anti-patterns. Includes context detection for 6 WooCommerce contexts, cross-references to all 5 prior skills, grep patterns for quick detection, and deep-dive reference doc links.

## Decisions Made

None - followed plan as specified. All decisions were locked in 05-CONTEXT.md from discuss-phase workflow.

## Deviations from Plan

None - plan executed exactly as written. No auto-fixes, no blocking issues, no architectural changes needed.

## Issues Encountered

None

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

**Ready for Plan 02 (Reference Docs):**
- SKILL.md complete with links to 3 reference docs (wc-extension-guide.md, wc-template-guide.md, wc-performance-guide.md)
- Deep coverage areas identified: HPOS migration, payment gateways, shipping methods, custom product types, order/cart operations, REST API, Action Scheduler, webhook security, template hierarchy, hooks catalog, cart fragments, session handling
- Surface-level areas identified: WC Blocks integration, coupon validation, email customization, variation performance, product caching
- BAD/GOOD patterns established throughout SKILL.md serve as foundation for reference doc examples
- Cross-references to all 5 prior skills validated—security (payment data), plugin (WC extension), blocks (WC Blocks), themes (template overrides), performance (cart fragments, queries)

**No blockers or concerns.**

---
*Phase: 05-woocommerce-development*
*Completed: 2026-02-06*

## Self-Check: PASSED

All files verified present:
- ✓ skills/wp-woocommerce-dev/SKILL.md (1,604 lines)

All commits verified present:
- ✓ 1ddf2f1 (feat(05-01): create WooCommerce development review skill)
