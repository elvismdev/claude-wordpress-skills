# Project Research Summary

**Project:** WordPress Claude Code Skills Build-Out (4 new skills)
**Domain:** WordPress Development Tooling (Code Review Skills)
**Researched:** 2026-02-06
**Confidence:** HIGH

## Executive Summary

This project adds four new code review skills to an existing WordPress Claude Code plugin: security review, Gutenberg blocks, theme development, and plugin development. Research reveals that WordPress 6.6+ (current stable) has fundamentally changed best practices compared to pre-2022 patterns. Block themes (FSE) replaced classic themes, block.json replaced JavaScript-only block registration, and the Interactivity API introduced a new paradigm for interactive blocks. The existing wp-performance-review skill provides a proven architectural pattern: ~500-line SKILL.md with 12 standardized sections, 3-6 reference docs (300-1200 lines each), and dual commands (full review + quick scan). This pattern should be replicated with domain-specific content.

The recommended approach is documentation-driven architecture (no runtime code) with markdown skills, YAML frontmatter, and comprehensive reference docs. Each skill targets WordPress 6.6+ and follows WordPress PHP Coding Standards for all examples. The critical success factor is avoiding false positives—skills must be context-aware (admin vs frontend, WP version, hosting platform) to maintain user trust. Security and plugin skills form the foundation (referenced by other skills), followed by Gutenberg blocks, then themes (most cross-skill dependencies).

Key risks center on version-blind advice (recommending deprecated pre-6.0 patterns), context-blind flags (flagging correct admin-only code), and platform-agnostic caching recommendations (ignoring shared hosting vs VIP). Mitigation requires explicit version requirements in all examples, execution context checks before flagging issues, and platform qualifiers for hosting-dependent features. Research confidence is HIGH—based on official WordPress Developer Handbook, Block Editor Handbook, and current 2026 security vulnerability data.

## Key Findings

### Recommended Stack

The project uses documentation-driven architecture with no traditional framework dependencies. Markdown is the native format for Claude Code skills (CommonMark spec), YAML frontmatter provides skill metadata, and JSON handles plugin configuration. All code examples must follow WordPress PHP Coding Standards and WordPress JavaScript Standards—this is non-negotiable for a WordPress-focused plugin. Users expect examples they can copy directly into WordPress.org-compliant projects.

**Core technologies:**
- **Markdown (CommonMark):** Skill definition format — human-readable, version-controllable, Claude-native
- **YAML 1.2:** Frontmatter metadata — standard for skill/command metadata in Claude Code ecosystem
- **JSON:** Plugin configuration — required by Claude Code plugin system (.claude-plugin/)
- **WordPress 6.6+:** Target platform — current stable, includes theme.json v3 and Interactivity API refinements
- **PHP 8.0+:** Example assumption — WordPress 6.6 requires 7.4+ but modern patterns assume 8.0+

**Version requirements:** Target WordPress 6.6 (current stable Feb 2026) with patterns applicable to 6.5+. This captures the Full Site Editing (FSE) era post-5.9, block.json era post-5.8, and Interactivity API post-6.5.

### Expected Features

Research identified table stakes, differentiators, and anti-features for each of four skills:

#### wp-security-review (HIGH complexity)
**Must have (table stakes):**
- XSS detection (35% of vulnerabilities) — missing esc_html(), esc_attr(), esc_url()
- SQL injection detection (15% of vulnerabilities) — unprepared queries, missing $wpdb->prepare()
- CSRF/nonce verification (11% of vulnerabilities) — wp_verify_nonce() in forms/AJAX
- Input sanitization — sanitize_text_field(), sanitize_email(), etc.
- Capability checks — current_user_can() before privileged operations
- REST API permission callbacks — required since WP 5.5

**Should have (competitive):**
- Privilege escalation detection — CVSS 10.0 pattern in 2026
- WordPress-specific anti-patterns — serialize() without protection, custom auth instead of WP functions
- Dangerous function detection — eval(), base64_decode(), unserialize(), exec()

**Defer (v2+):**
- Server configuration scanning — not code-level
- CVE database lookup — requires real-time API

#### wp-gutenberg-blocks (HIGH complexity)
**Must have (table stakes):**
- block.json validation — required since WP 5.8
- Dynamic vs static block patterns — render callback detection
- useBlockProps() hook validation — required for proper block wrapper
- Attribute schema validation — types, defaults, sources
- InnerBlocks usage — nested block patterns
- Block deprecation handling — migration patterns

**Should have (competitive):**
- Interactivity API patterns — new in WP 6.5, future of blocks
- Block context (providesContext/usesContext) — parent-child communication
- Block variations vs patterns guidance — common confusion point
- InnerBlocks.Content in dynamic blocks — common bug

**Defer (v2+):**
- CSS-in-JS patterns — React styling decision, not WP-specific
- Custom block editor — advanced, rare use case
- TypeScript patterns — language choice, not WP-specific

#### wp-theme-development (MEDIUM complexity)
**Must have (table stakes):**
- theme.json structure validation — core of block themes
- Template hierarchy — foundation of WordPress theming
- FSE templates (HTML) — templates/*.html block markup
- Block theme vs classic detection — auto-detect by theme.json presence
- Global styles — design system via theme.json
- Template parts — reusable components (header, footer)

**Should have (competitive):**
- theme.json v2 vs v3 differences — version-specific features
- Hardcoded styles anti-pattern — WordPress.org requirement
- Block stylesheet patterns — performance optimization
- Style variations — alternative design options

**Defer (v2+):**
- Page builder plugin integration — third-party tools
- WooCommerce theming — plugin-specific
- CSS preprocessor patterns — build tooling choice

#### wp-plugin-development (MEDIUM-HIGH complexity)
**Must have (table stakes):**
- Plugin header validation — WordPress.org requirement
- Prefix convention — 4-5 char unique prefix to prevent collisions
- Activation/deactivation hooks — lifecycle management
- Custom post types — register_post_type() review
- Settings API usage — register_setting() patterns
- Internationalization (i18n) — text domain, __(), _e()

**Should have (competitive):**
- Plugin Check standards compliance — WordPress.org submission (PCP tool)
- REST API endpoint registration — modern API pattern with permission_callback
- AJAX handler patterns — wp_ajax_ hook structure
- WP_Query optimization — performance in plugins
- OOP patterns for large plugins — 1000+ lines

**Defer (v2+):**
- Multi-network support — advanced, rare
- Testing framework setup — assume tests exist
- Plugin update mechanisms — WordPress.org handles updates

### Architecture Approach

The existing wp-performance-review skill validates the architectural pattern. Each new skill replicates this proven structure: one SKILL.md (~500-600 lines) with 12 standardized sections, 3-6 reference docs organized by sub-domain, and dual slash commands (full review + quick scan). Skills remain isolated—no shared code—with cross-references for overlapping topics.

**Major components:**
1. **SKILL.md (500-600 lines)** — Core skill file with YAML frontmatter, triggers, workflow, file-type checks, grep patterns, code examples, severity definitions
2. **references/ directory (3-6 files)** — Sub-domain deep dives: vulnerability-patterns.md, escaping-guide.md (security), block-patterns.md, block-json-guide.md (Gutenberg), theme-json-guide.md (themes), architecture-patterns.md (plugins)
3. **Slash commands (2 per skill)** — `/wp-[domain]-review` (full workflow), `/wp-[domain]` (quick scan using grep patterns)

**Cross-skill reference strategy:**
- Security patterns referenced by themes (template escaping), plugins (SQL injection), blocks (nonce handling)
- Performance patterns referenced by plugins (WP_Query optimization), themes (template queries)
- Plugin patterns referenced by blocks (registration), themes (hooks)
- Explicit cross-references instead of code duplication: "See wp-security-review for escaping guide"

**Reference granularity:** Security and Gutenberg need 5-6 references (deeper technical domains with multiple sub-specialties). Themes and plugins need 3-4 references (more unified patterns with fewer distinct sub-domains).

### Critical Pitfalls

Research identified 12 domain-specific pitfalls. Top 5 critical issues:

1. **Context-blind performance flags** — Flagging `posts_per_page => -1` universally without checking if code runs in admin/cron/CLI context. Prevention: Check is_admin(), wp_doing_cron(), defined('WP_CLI') before flagging. Add severity qualifiers: "CRITICAL in public-facing code, acceptable in admin/CLI."

2. **WordPress version-blind advice** — Recommending pre-6.0 patterns (e.g., registerBlockTypeFromMetadata removed, defer/async only via strategy parameter in WP 6.3+). Prevention: State version requirements explicitly ("WordPress 6.3+ introduces strategy parameter"), include deprecation warnings, reference official WordPress Developer Handbook for current patterns.

3. **Platform-agnostic cache advice** — Recommending object cache patterns without checking wp_using_ext_object_cache(). On shared hosting without Redis/Memcached, transients create database bloat. Prevention: Qualify cache advice with hosting context, provide fallback patterns for shared hosting, check wp_using_ext_object_cache() before recommending transients.

4. **Security false positives (missing nonces in REST)** — Flagging missing wp_verify_nonce() in REST API endpoints (WordPress handles this via rest_cookie_check_errors()) or public read-only operations. Prevention: Only flag nonces for state-changing operations (POST/DELETE/UPDATE), don't flag REST endpoints, distinguish read from write operations.

5. **Gutenberg anti-patterns from pre-block.json era** — Teaching block development without block.json (pre-WP 5.8 pattern). Prevention: Always recommend block.json as canonical, mark pre-5.8 patterns as "legacy," include apiVersion: 3 in examples (current standard).

## Implications for Roadmap

Based on combined research, recommended 4-phase structure aligned with dependency order:

### Phase 1: wp-security-review
**Rationale:** Foundational skill with zero dependencies on other skills. Security patterns (escaping, sanitization, SQL injection prevention) are referenced by all subsequent skills. Independent domain can be built without cross-references.

**Delivers:**
- SKILL.md with XSS detection, SQL injection detection, CSRF/nonce verification, capability checks
- 5 reference docs: vulnerability-patterns.md, escaping-guide.md, sanitization-guide.md, auth-patterns.md, nonce-csrf-guide.md
- 2 commands: /wp-sec-review (full), /wp-sec (quick scan)

**Addresses features:**
- XSS detection (35% of vulnerabilities), SQL injection (15%), CSRF/nonce (11%)
- REST API permission callback validation
- Dangerous function detection (eval, exec, unserialize)

**Avoids pitfalls:**
- Pitfall 4: Security false positives in REST API (check for REST context before flagging nonces)
- Pitfall 10: InnerBlocks sanitization (document that InnerBlocks content is pre-sanitized)

**Research needs:** Standard patterns, skip /gsd:research-phase (official docs are comprehensive)

### Phase 2: wp-plugin-development
**Rationale:** Second foundational skill. Plugin architecture patterns (hooks, OOP, registration) inform block development in Phase 3. Security skill from Phase 1 already exists for cross-reference.

**Delivers:**
- SKILL.md with plugin header validation, activation hooks, CPT/taxonomy registration, Settings API, hooks, i18n
- 4 reference docs: architecture-patterns.md, hooks-guide.md, oop-patterns.md, api-patterns.md
- 2 commands: /wp-plugin-review (full), /wp-plugin (quick scan)

**Addresses features:**
- Plugin Check standards compliance (WordPress.org submission)
- REST API endpoint registration patterns
- AJAX handler structure
- OOP patterns for large plugins (1000+ lines)

**Avoids pitfalls:**
- Pitfall 8: Plugin architecture advice ignoring OOP (show procedural for small plugins, OOP for 1000+ lines)
- Pitfall 3: Platform-agnostic cache (check wp_using_ext_object_cache(), provide shared hosting fallbacks)

**Research needs:** Standard patterns, skip /gsd:research-phase (Plugin Handbook is authoritative)

### Phase 3: wp-gutenberg-blocks
**Rationale:** Depends on Phase 2 (plugin skill) for block registration patterns and Phase 1 (security skill) for nonce handling in block AJAX. Block patterns inform theme development in Phase 4 (pattern registration).

**Delivers:**
- SKILL.md with block.json validation, dynamic vs static blocks, useBlockProps, attributes, InnerBlocks, deprecation
- 6 reference docs: block-patterns.md, block-json-guide.md, react-patterns.md, attributes-guide.md, dynamic-blocks-guide.md, interactivity-api-guide.md
- 2 commands: /wp-block-review (full), /wp-block (quick scan)

**Addresses features:**
- block.json validation (WP 5.8+)
- Interactivity API patterns (WP 6.5+, future of blocks)
- Block context for parent-child communication
- Block variations vs patterns guidance

**Avoids pitfalls:**
- Pitfall 5: Pre-block.json patterns (always show block.json as canonical, apiVersion: 3)
- Pitfall 2: Version-blind advice (state WP 6.5+ for Interactivity API, WP 5.8+ for block.json)
- Pitfall 7: Overly aggressive block style warnings (flag 5+ styles, not any style)

**Research needs:** Interactivity API may need deeper research (new in 6.5, patterns emerging). Consider /gsd:research-phase for Interactivity API specifics.

### Phase 4: wp-theme-development
**Rationale:** Most dependent on other skills. References security (template escaping), performance (template queries), blocks (pattern registration, block theme concepts), plugin (hooks for theme customization). Build last to maximize cross-reference opportunities.

**Delivers:**
- SKILL.md with theme.json structure, template hierarchy, FSE templates, template parts, block theme detection, global styles
- 4 reference docs: theme-json-guide.md, template-patterns.md, fse-guide.md, classic-to-block-guide.md
- 2 commands: /wp-theme-review (full), /wp-theme (quick scan)

**Addresses features:**
- theme.json v3 structure validation (settings, styles, patterns)
- Block theme vs classic theme detection
- Hardcoded styles anti-pattern detection
- Style variations and root padding aware alignments

**Avoids pitfalls:**
- Pitfall 6: FSE theme advice that's actually classic (default to block themes, label classic patterns)
- Pitfall 2: Version-blind advice (theme.json v3 is WP 6.6+, FSE standard since 5.9)
- Pitfall 11: Outdated script enqueue (show WP 6.3+ strategy parameter)

**Research needs:** Standard patterns, skip /gsd:research-phase (Theme Handbook and theme.json reference are comprehensive)

### Phase Ordering Rationale

**Dependency-driven sequence:** Security → Plugin → Gutenberg → Theme minimizes cross-reference complexity. Security and Plugin are foundational (zero external dependencies). Gutenberg references both. Theme references all three.

**Build order benefits:**
1. Early phases can reference completed later phases without blocking work
2. Phase 3 (Gutenberg) may need Interactivity API research, but Phases 1-2 proceed without research pauses
3. Phase 4 (Theme) benefits from all prior phases existing for cross-reference examples
4. Testing can validate cross-references work correctly in Phase 4 (references to 1, 2, 3 all testable)

**Avoids pitfall propagation:** Building Security first establishes correct escaping patterns that Phases 2-4 reference. Building Plugin second establishes hook patterns that Phases 3-4 reference. Prevents contradictory advice across skills.

### Research Flags

**Needs deeper research:**
- **Phase 3 (Gutenberg):** Interactivity API patterns — New in WP 6.5, patterns still emerging in 2026. Consider /gsd:research-phase focused on data- directives, store configuration, when to use vs React. Official docs exist but community patterns less established than other APIs.

**Standard patterns (skip research-phase):**
- **Phase 1 (Security):** Well-documented via WordPress Security Handbook, CVE analysis from WP vulnerability databases, escaping/sanitization docs are comprehensive
- **Phase 2 (Plugin):** Plugin Handbook is authoritative, WordPress.org Plugin Check tool (PCP) defines standards, hooks and OOP patterns mature
- **Phase 4 (Theme):** Theme Handbook and theme.json reference (living spec) are comprehensive, FSE patterns documented since WP 5.9

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | No runtime dependencies, documentation-driven architecture proven by existing skill |
| Features (Security) | HIGH | CVE vulnerability data from 2026, WordPress Security Handbook comprehensive |
| Features (Gutenberg) | HIGH | Block Editor Handbook authoritative, block.json well-documented |
| Features (Gutenberg Interactivity API) | MEDIUM | WP 6.5+ feature, official docs exist but patterns emerging |
| Features (Theme) | HIGH | Theme.json v3 reference (living spec), FSE standard since WP 5.9 |
| Features (Plugin) | HIGH | Plugin Handbook mature, Plugin Check (PCP) tool defines standards |
| Architecture | HIGH | Existing wp-performance-review validates 12-section SKILL.md, reference count, command pair |
| Pitfalls | HIGH | Based on existing skill analysis + official docs verification + 2026 WordPress patterns |

**Overall confidence:** HIGH

### Gaps to Address

Minor gaps that need validation during implementation:

- **Interactivity API edge cases:** WP 6.5 introduced Interactivity API. Official docs exist but community best practices still evolving. During Phase 3, may need /gsd:research-phase to catalog real-world directive usage patterns and common mistakes.

- **Plugin Check (PCP) tool coverage:** WordPress.org Plugin Check tool defines submission standards. During Phase 2, verify current PCP checks (tool updated frequently) to ensure skill patterns match official requirements. Tool available as WP plugin for testing.

- **theme.json v3 living spec:** theme.json is "living specification" that can change with WordPress releases. During Phase 4, verify theme.json schema against WP 6.6+ docs to ensure examples match current version.

- **WordPress 6.7+ release impact:** If WordPress 6.7 releases before project completion, verify no breaking changes to block.json (apiVersion 4?), theme.json (v4?), or deprecated functions used in examples.

- **False positive validation:** Each skill needs testing against real codebases to identify false positive scenarios. Test with: official WordPress themes (Twenty Twenty-Four), popular plugins (Gutenberg, WooCommerce), starter themes. Add false positive scenarios to "Common Mistakes" section.

## Sources

### Primary (HIGH confidence)
- [WordPress PHP Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/) — code example formatting
- [WordPress JavaScript Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/javascript/) — JS/JSX examples
- [Block Editor Handbook](https://developer.wordpress.org/block-editor/) — Gutenberg patterns, block.json, Interactivity API
- [Theme.json v3 Reference](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/) — theme configuration
- [Plugin Handbook - Best Practices](https://developer.wordpress.org/plugins/plugin-basics/best-practices/) — plugin patterns
- [WordPress Security - Theme Handbook](https://developer.wordpress.org/themes/advanced-topics/security/) — security patterns
- [Plugin Check (PCP) Tool](https://wordpress.org/plugins/plugin-check/) — WordPress.org standards

### Secondary (MEDIUM confidence)
- [WordPress Vulnerabilities Database 2026](https://wpsecurityninja.com/wordpress-vulnerabilities-database/) — CVE statistics (35% XSS, 15% SQL injection, 11% CSRF)
- [WordPress Security Vulnerabilities History 2026](https://www.glorywebs.com/blog/wordpress-security-update-history) — privilege escalation CVSS 10.0
- [Latest Trends in WordPress Development for 2026](https://wpdeveloper.com/latest-trends-in-wordpress-development/) — FSE adoption, Interactivity API usage
- [WordPress Plugin Architecture: OOP and Design Patterns](https://www.voxfor.com/wordpress-plugin-architecture-oop-design-patterns/) — OOP patterns for large plugins

### Tertiary (LOW confidence)
- [Gutenberg Blocks in 2026: AI Era](https://vapvarun.com/gutenberg-blocks-2026-wordpress-block-editor-ai-era/) — community perspective on Interactivity API adoption
- [WordPress Full Site Editing - Troubleshooting](https://fullsiteediting.com/lessons/troubleshooting-block-themes/) — FSE common issues

---
*Research completed: 2026-02-06*
*Ready for roadmap: yes*
