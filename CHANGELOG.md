# Changelog

All notable changes to this project will be documented in this file.

## [2.0.0] - 2026-02-06

### Added

- **wp-security-review** skill: Code-level security vulnerability detection (XSS, SQL injection, CSRF, privilege escalation, file upload security) with 5 reference docs and 2 slash commands (`/wp-sec-review`, `/wp-sec`)
- **wp-plugin-development** skill: Plugin architecture and WordPress.org standards review (plugin structure, lifecycle hooks, CPT/taxonomies, Settings API, hooks, i18n) with 4 reference docs and 2 slash commands (`/wp-plugin-review`, `/wp-plugin`)
- **wp-block-development** skill: Gutenberg block development patterns (block.json, dynamic vs static blocks, InnerBlocks, Interactivity API for WP 6.5+) with 4 reference docs and 2 slash commands (`/wp-block-review`, `/wp-block`)
- **wp-theme-development** skill: Block theme development and Full Site Editing patterns (theme.json v3, template hierarchy, global styles, style variations) with 4 reference docs and 2 slash commands (`/wp-theme-review`, `/wp-theme`)
- **wp-woocommerce-dev** skill: WooCommerce extension development (HPOS compatibility, payment gateways, WC CRUD, cart fragments optimization, Action Scheduler, template overrides) with 3 reference docs and 2 slash commands (`/wp-woo-review`, `/wp-woo`)

### Changed

- Updated README.md with all 6 skills documented, including complete trigger phrases and command listings
- Updated marketplace.json with comprehensive skill descriptions
- Bumped version to 2.0.0 reflecting complete plugin with all planned WordPress development domains covered
- Enhanced skill descriptions to reflect modern WordPress development (WP 6.5+ for blocks, WP 6.6+ for themes, WC 8.2+ for WooCommerce)

## 1.3.1 - 2025-11-28

### Improved

- **Enhanced skill trigger keywords** for better automatic invocation
  - Added triggers: "before launch", "anti-patterns", "slow queries", "scale WordPress", "code review", "optimization audit"
  - Updated README.md trigger phrases table to match SKILL.md description
  - Added natural language examples: "Check my theme before launch", "Find anti-patterns in this plugin"

## 1.3.0 - 2025-11-26

### Added

- **Transients & Options checks** — Detect dynamic transient keys causing wp_options table bloat, frequently-changing data misuse, and large data storage on shared hosting without object cache
- **WP-Cron bottleneck detection** — Identify missing `DISABLE_WP_CRON` constant, long-running callbacks blocking cron queue, and duplicate schedules from missing `wp_next_scheduled()` checks
- **Conditional asset loading checks** — Flag `wp_enqueue_script`/`wp_enqueue_style` calls without conditional tags like `is_page()` or `is_singular()`
- **New grep patterns** for quick detection:
  - Asset loading without conditionals
  - Dynamic transient keys (`set_transient` with variables)
  - `wp_schedule_event` without `wp_next_scheduled` guard
- **Comprehensive code examples** with BAD/GOOD patterns for all new checks
- **CLAUDE.md** project documentation for Claude Code guidance

## 1.2.1 - 2025-11-25

### Changed

- Added metadata section to marketplace.json for better plugin discovery

## 1.2.0 - 2025-11-25

### Changed

- Refined README installation instructions
- Improved plugin metadata consistency

## 1.1.0 - 2025-11-25

### Added

- **Slash commands** for explicit skill invocation
  - `/wp-perf-review [path]` — Full WordPress performance code review
  - `/wp-perf [path]` — Quick triage scan (critical patterns only)

### Improved

- Commands explicitly reference skill workflow sections for better LLM orchestration
- Commands instruct Claude to use the skill's Output Format
- Commands differentiate between quick scan (grep patterns) and full review (deep analysis)
- Clear instructions for loading reference files when needed

## 1.0.0 - 2025-11-25

### Added

- **wp-performance-review** skill with comprehensive performance code review capabilities
  - Database query anti-patterns detection (unbounded queries, missing WHERE, LIKE patterns, NOT IN)
  - Hooks & actions analysis (expensive init code, DB writes on page load)
  - Caching issue detection (uncached functions, object cache patterns)
  - AJAX & external request optimization (admin-ajax alternatives, polling patterns)
  - Template performance checks (N+1 queries, get_template_part usage)
  - PHP code anti-patterns (in_array performance, heredoc escaping)
  - JavaScript bundle analysis (full library imports, defer/async)
  - Block Editor patterns (registerBlockStyle overhead, InnerBlocks)
  - Platform-specific guidance (VIP, WP Engine, Pantheon, self-hosted)
- Reference documentation:
  - `anti-patterns.md` — Complete catalog of 45+ anti-patterns
  - `wp-query-guide.md` — WP_Query optimization patterns
  - `caching-guide.md` — Caching strategy implementation
  - `measurement-guide.md` — High-traffic preparation and monitoring
- Plugin marketplace manifest for Claude Code installation
- MIT License
- Contributing guidelines

### Technical Details

- All code examples follow WordPress PHP Coding Standards
- Severity levels: CRITICAL, WARNING, INFO
- 2,600+ lines of documentation across 5 files
- 28 quick-lookup patterns in anti-patterns.md
- 90+ code examples with BAD/GOOD comparisons
