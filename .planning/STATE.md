# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-02-06)

**Core value:** Every WordPress developer using Claude Code gets comprehensive, expert-level code review and development guidance across all major WordPress domains.
**Current focus:** Phase 4 complete — ready for Phase 5

## Current Position

Phase: 5 of 5 (WooCommerce Development) — COMPLETE
Plan: 3 of 3 in current phase
Status: Project complete - All phases delivered
Last activity: 2026-02-07 — Completed 05-03-PLAN.md

Progress: [██████████████] 100% (15/15 plans complete)

## Performance Metrics

**Velocity:**
- Total plans completed: 15
- Average duration: 8 minutes
- Total execution time: 2.0 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01 Security Review | 3/3 | 15m | 5m |
| 02 Plugin Dev | 3/3 | 53m | 18m |
| 03 Gutenberg | 3/3 | 16m | 5m |
| 04 Theme Dev | 3/3 | 15m | 5m |
| 05 WooCommerce | 3/3 | <15m | 5m |

**Recent Trend:**
- Last 5 plans: 04-03 (<1m), 05-01 (5m), 05-02 (9m), 05-03 (<1m)
- Trend: Slash command plans complete in <1 minute; SKILL.md plans complete in 5 minutes; reference doc plans complete in 9-28 minutes

*Updated after each plan completion*

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- Code-level security only (no server hardening) — keeps skill focused on what Claude can review
- Full-stack Gutenberg coverage (PHP + React/JSX) — WP 6.x+ block development requires both patterns
- Same slash command pattern per skill (full + quick) — consistent UX proven by performance skill
- All 5 skills at same depth — consistent quality, no half-built skills
- WP 6.x+ target only — modern APIs without legacy compatibility burden
- Dependency-driven build order — Security → Plugin → Gutenberg → Theme → WooCommerce
- CWE references in security findings (01-01) — industry-standard vulnerability classification
- Context-aware severity (01-01) — admin vs public code gets different risk levels
- Three-pillar security model (01-01) — sanitize, authorize, escape framework
- Late escaping emphasis (01-01) — escape at output, not storage
- Cookbook format for references (01-02) — quick lookup + detailed patterns proven by performance skill
- BAD/GOOD code pairs throughout (01-02) — side-by-side comparison most effective for learning
- False positive documentation (01-02) — explicitly document when security measures aren't needed
- Security crossover pattern (02-01) — cross-reference wp-security-review instead of duplicating security patterns
- Context-aware plugin detection (02-01) — WordPress.org vs private vs mu-plugin vs drop-in contexts adjust severity
- WordPress PHP Coding Standards in all examples (02-01) — spaces in parens, array() not [], Yoda conditions
- Cookbook format for plugin references (02-02) — quick reference table + patterns + edge cases + WP nuances, proven effective
- Cross-references between reference docs (02-02) — architecture → hooks → OOP → API creates navigation paths
- Security cross-references in APIs (02-02) — permission_callback, nonces, capabilities point to wp-security-review skill
- dbDelta quirks documented (02-02) — two spaces after PRIMARY KEY critical for WordPress custom table creation
- Slash command delegation pattern (02-03) — /wp-plugin-review and /wp-plugin mirror security command structure exactly
- First dual-language skill (03-01) — wp-block-development covers both PHP and JavaScript/React with separate BAD/GOOD patterns
- Source vs build review (03-01) — review src/ files for code patterns, flag missing/stale build/, don't review build/ for quality
- Grep patterns differ by language (03-01) — JS patterns need different regex than PHP, documented with --include flags
- Interactivity API scoping (03-01) — WP 6.5+ frontend-only, editor still React, version markers throughout
- apiVersion 3 className injection (03-02) — NOT auto-injected in save, must use useBlockProps.save() explicitly, migration guide critical
- InnerBlocks dynamic block gotcha (03-02) — MUST return InnerBlocks.Content in save even if dynamic, $content contains saved inner blocks
- DO NOT escape $content (03-02) — wp_kses_post() breaks core/embed blocks, inner blocks already sanitized
- viewScriptModule required (03-02) — Interactivity API needs ES modules, viewScript doesn't support imports
- Block themes primary focus (04-01) — WP 6.6+ with theme.json v3, classic themes covered for migration guidance only
- Theme type auto-detection (04-01) — file structure determines block/classic/hybrid/child/WordPress.org context
- theme.json v3 breaking changes (04-01, 04-02) — defaultFontSizes and defaultSpacingSizes default to true in v3, must explicitly set false
- useRootPaddingAwareAlignments essential (04-01, 04-02) — required when root padding present, object notation only (not CSS shorthand)
- Hardcoded styles severity (04-01) — inline styles in block themes = WARNING (defeats FSE purpose, prevents user customization)
- Template hierarchy identical (04-02) — block (.html) and classic (.php) themes use same fallback chain, WordPress selects based on file extension
- FSE shifts control (04-02) — from theme developers (code) to site users (visual Site Editor), theme.json defines defaults
- Three migration strategies (04-02) — full conversion (big bang), hybrid theme (gradual), Create Block Theme plugin (automated export)
- Local font hosting required (04-02) — WordPress.org submission mandates GPL fonts, privacy compliance (GDPR), .woff2 format recommended
- HPOS compatibility as primary WooCommerce focus (05-01) — CRITICAL if missing declare_compatibility(), breaks modern WC stores since 8.2
- Code-level payment security only (05-01) — NOT PCI compliance audit, focus on raw card data anti-patterns, HTTPS checks, webhook verification
- Cart fragments still major performance issue (05-01) — site-wide loading flagged as CRITICAL despite WC 7.8 improvements, recommend conditional dequeuing or Mini-Cart Block
- Hooks-first philosophy for WC templates (05-01) — template overrides with deleted do_action() calls flagged as CRITICAL, suggest hooks instead
- Most cross-referential skill (05-01) — wp-woocommerce-dev touches all 5 prior skills (security for payment data, plugin for WC extensions, blocks for WC Blocks, themes for template overrides, performance for cart fragments/queries)
- Cookbook format for WC references (05-02) — quick reference table + detailed patterns + edge cases + WC nuances, proven effective across all phases
- Payment security anti-patterns emphasized (05-02) — never store/log raw card data repeated throughout wc-extension-guide.md with CRITICAL severity
- Cart fragments detailed analysis (05-02) — wc-performance-guide.md provides problem statement, three solutions (conditional dequeuing, Mini-Cart Block, complete disable), and detection patterns
- Template hooks preservation critical (05-02) — wc-template-guide.md emphasizes preserving all do_action() and apply_filters() calls in overrides, deletion flagged as CRITICAL
- Three WC reference docs totaling 3,228 lines (05-02) — wc-extension-guide.md (1,336 lines), wc-template-guide.md (886 lines), wc-performance-guide.md (1,006 lines)
- Version 2.0.0 for complete build-out (05-03) — major milestone with all 6 WordPress development domains covered
- wp-woo-review references all 5 other skills (05-03) — most cross-referential command (reflects WC integration depth)
- CHANGELOG.md [2.0.0] entry lists all 5 new skills (05-03) — detailed feature descriptions for all phases 1-5

### Pending Todos

None yet.

### Blockers/Concerns

None yet.

## Session Continuity

Last session: 2026-02-07
Stopped at: Completed 05-03-PLAN.md (WooCommerce slash commands + infrastructure)
Resume file: None
Next action: Project complete - all 15/15 plans delivered
