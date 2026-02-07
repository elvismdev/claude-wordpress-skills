# Roadmap: Claude WordPress Skills Build-Out

## Overview

This roadmap delivers five complete WordPress code review skills plus infrastructure updates, expanding the plugin from 1 skill (performance) to 6 comprehensive skills. Build order follows dependency hierarchy: security patterns enable plugin development, plugin patterns enable block development, and all three inform theme development. WooCommerce builds last, incorporating patterns from all prior skills. Each skill matches the quality bar of the existing wp-performance-review skill with complete SKILL.md, reference docs, and dual slash commands.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Security Review** - Code-level security patterns and vulnerability detection
- [x] **Phase 2: Plugin Development** - Plugin architecture and WordPress.org standards
- [x] **Phase 3: Gutenberg Blocks** - Block API, block.json, and Interactivity API patterns
- [x] **Phase 4: Theme Development** - Block themes, theme.json, and FSE patterns
- [x] **Phase 5: WooCommerce Development** - WooCommerce extension patterns and infrastructure updates

## Phase Details

### Phase 1: Security Review
**Goal**: Claude can detect and explain WordPress security vulnerabilities in code (XSS, SQL injection, CSRF, privilege escalation)
**Depends on**: Nothing (first phase)
**Requirements**: SEC-01 through SEC-29 (29 requirements)
**Success Criteria** (what must be TRUE):
  1. User can invoke /wp-sec-review on a codebase and receive comprehensive security findings
  2. User can invoke /wp-sec for quick triage of high-risk patterns
  3. Claude automatically detects security review context from trigger phrases like "security issues" or "XSS vulnerability"
  4. All code examples follow WordPress PHP Coding Standards with BAD / GOOD pattern
  5. Security findings include severity levels (CRITICAL/WARNING/INFO) appropriate to impact
**Plans**: 3 plans

Plans:
- [x] 01-01: SKILL.md with vulnerability detection patterns (XSS, SQL injection, CSRF, privilege escalation)
- [x] 01-02: Reference docs (vulnerability-patterns.md, escaping-guide.md, sanitization-guide.md, auth-patterns.md, nonce-csrf-guide.md)
- [x] 01-03: Slash commands (/wp-sec-review full review, /wp-sec quick scan)

### Phase 2: Plugin Development
**Goal**: Claude can review plugin architecture and WordPress.org submission standards
**Depends on**: Phase 1 (references security patterns for capability checks, nonce validation)
**Requirements**: PLG-01 through PLG-29 (29 requirements)
**Success Criteria** (what must be TRUE):
  1. User can invoke /wp-plugin-review on plugin code and receive architecture feedback
  2. User can invoke /wp-plugin for quick standards check
  3. Claude detects plugin context from trigger phrases like "plugin best practices" or "WordPress.org submission"
  4. Plugin reviews validate Plugin Check (PCP) tool standards for WordPress.org compliance
  5. Code examples demonstrate proper prefixing, hooks, i18n, and activation patterns
**Plans**: 3 plans

Plans:
- [x] 02-01: SKILL.md with plugin patterns (headers, activation hooks, CPT/taxonomy, Settings API, REST API, AJAX)
- [x] 02-02: Reference docs (architecture-patterns.md, hooks-guide.md, oop-patterns.md, api-patterns.md)
- [x] 02-03: Slash commands (/wp-plugin-review full review, /wp-plugin quick scan)

### Phase 3: Gutenberg Blocks
**Goal**: Claude can review block development patterns for WordPress 6.5+ (block.json, Interactivity API)
**Depends on**: Phase 1 (security/nonce in block AJAX), Phase 2 (block registration patterns)
**Requirements**: BLK-01 through BLK-30 (30 requirements)
**Success Criteria** (what must be TRUE):
  1. User can invoke /wp-block-review on block code and receive block API feedback
  2. User can invoke /wp-block for quick block.json validation
  3. Claude detects block context from trigger phrases like "Gutenberg block" or "block.json issues"
  4. Block reviews cover both PHP (server-side rendering) and React/JSX (editor patterns)
  5. Interactivity API patterns (data-wp-* directives) are documented with WP 6.5+ version markers
**Plans**: 3 plans

Plans:
- [x] 03-01-PLAN.md -- SKILL.md with block patterns (block.json validation, dynamic vs static, useBlockProps, InnerBlocks, deprecation, Interactivity API)
- [x] 03-02-PLAN.md -- Reference docs (block-json-guide.md, editor-patterns.md, dynamic-blocks-guide.md, interactivity-api-guide.md)
- [x] 03-03-PLAN.md -- Slash commands (/wp-block-review full review, /wp-block quick scan)

### Phase 4: Theme Development
**Goal**: Claude can review block themes and theme.json v3 structure (FSE patterns for WP 6.6+)
**Depends on**: Phase 1 (template escaping), Phase 2 (theme hooks), Phase 3 (block patterns for theme patterns/)
**Requirements**: THM-01 through THM-29 (29 requirements)
**Success Criteria** (what must be TRUE):
  1. User can invoke /wp-theme-review on theme code and receive FSE guidance
  2. User can invoke /wp-theme for quick theme.json validation
  3. Claude auto-detects block theme vs classic theme and provides version-appropriate guidance
  4. Theme reviews validate theme.json v3 structure (settings, styles, patterns)
  5. Hardcoded style anti-patterns are flagged with WordPress.org submission context
**Plans**: 3 plans

Plans:
- [x] 04-01-PLAN.md -- SKILL.md with theme patterns (theme.json v3 validation, template hierarchy, FSE templates, template parts, global styles, style variations, block patterns, child themes)
- [x] 04-02-PLAN.md -- Reference docs (theme-json-guide.md, template-patterns.md, fse-guide.md, classic-to-block-guide.md)
- [x] 04-03-PLAN.md -- Slash commands (/wp-theme-review full review, /wp-theme quick scan)

### Phase 5: WooCommerce Development
**Goal**: Claude can review WooCommerce extensions and theme integration + plugin infrastructure is updated
**Depends on**: Phase 1 (payment security), Phase 2 (WC as plugin extension), Phase 3 (WC blocks), Phase 4 (WC theming)
**Requirements**: WOO-01 through WOO-30 (30 requirements), INF-01 through INF-04 (4 requirements)
**Success Criteria** (what must be TRUE):
  1. User can invoke /wp-woo-review on WooCommerce code and receive extension/theming feedback
  2. User can invoke /wp-woo for quick WC pattern check
  3. Claude detects WooCommerce context from trigger phrases like "WooCommerce extension" or "payment gateway"
  4. WooCommerce reviews cover HPOS compatibility and cart fragment performance patterns
  5. README.md documents all 6 skills with status, triggers, and commands
  6. Plugin version is bumped and CHANGELOG.md reflects all new skills
**Plans**: 3 plans

Plans:
- [x] 05-01-PLAN.md -- SKILL.md with WooCommerce patterns (HPOS compatibility, CRUD data access, payment gateways, shipping methods, custom product types, hooks catalog, cart fragments, Action Scheduler, template overrides)
- [x] 05-02-PLAN.md -- Reference docs (wc-extension-guide.md, wc-template-guide.md, wc-performance-guide.md)
- [x] 05-03-PLAN.md -- Slash commands (/wp-woo-review, /wp-woo) + infrastructure updates (README.md, marketplace.json, plugin.json version, CHANGELOG.md)

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3 -> 4 -> 5

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Security Review | 3/3 | Complete | 2026-02-06 |
| 2. Plugin Development | 3/3 | Complete | 2026-02-06 |
| 3. Gutenberg Blocks | 3/3 | Complete | 2026-02-06 |
| 4. Theme Development | 3/3 | Complete | 2026-02-06 |
| 5. WooCommerce Development | 3/3 | Complete | 2026-02-06 |
