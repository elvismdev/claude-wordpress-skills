# Phase 2: Plugin Development - Context

**Gathered:** 2026-02-06
**Status:** Ready for planning

<domain>
## Phase Boundary

Deliver the `wp-plugin-development` skill for WordPress plugin architecture review and WordPress.org submission standards compliance. Covers plugin headers, activation/deactivation/uninstall lifecycle, custom post types and taxonomies, Settings API, REST API integration, AJAX handlers, hooks system (actions and filters), internationalization (i18n/l10n), coding standards, and Plugin Check (PCP) tool compliance. Code-level review only — no plugin deployment, marketplace management, or premium licensing patterns.

</domain>

<decisions>
## Implementation Decisions

### Detection scope & depth
- **Deep coverage (dedicated sections in SKILL.md):** Plugin headers and metadata, activation/deactivation/uninstall hooks, custom post types and taxonomies (register_post_type, register_taxonomy), Settings API (register_setting, add_settings_section, add_settings_field), hooks system (add_action/add_filter patterns, priority, removal), internationalization (text domains, __(), _e(), _n(), _x()), prefixing and namespacing
- **Surface-level coverage (scan patterns + brief guidance):** REST API endpoint registration (deep security patterns live in wp-security-review), AJAX handler registration (security patterns in wp-security-review), shortcode registration (deprecated pattern for blocks but still common), widget registration (classic widgets — blocks preferred in WP 6.x+), admin notices and dismissible messages
- **WordPress.org compliance checks:** Plugin Check (PCP) tool standards — proper headers, no direct file access, text domain matching slug, no deprecated functions, readme.txt/readme.md requirements, license compatibility (GPLv2+)
- **Context-aware detection:** Distinguish between:
  - WordPress.org submission (strictest standards — PCP compliance required)
  - Private/enterprise plugins (more flexibility on naming, no readme.txt needed)
  - Must-use plugins (mu-plugins — different loading behavior, no activation hooks)
  - Drop-in plugins (special files like object-cache.php, db.php)

### Security skill crossover
- **Do NOT duplicate security patterns** — the plugin skill references wp-security-review for security concerns
- **Cross-reference approach:** When the plugin review encounters a security-relevant pattern (e.g., form handler without nonce, AJAX without capability check), flag it as "See wp-security-review skill for detailed security analysis" rather than repeating the full pattern
- **Plugin-specific security notes:** Include brief reminders of the three-step pattern (nonce + capability + sanitize) in the context of plugin handlers, but point to security skill for depth
- **Slash command integration:** If /wp-plugin-review finds security concerns, suggest running /wp-sec-review for comprehensive security analysis

### Finding output style
- Mirror the security/performance skill output format exactly:
  - Group findings by FILE (consistent with other skills)
  - Each finding: line number, severity, issue description, explanation, ❌ BAD / ✅ GOOD code pair with fix
  - Summary at end with total counts by severity
- **Severity mapping for plugins:**
  - CRITICAL: Will cause WordPress.org rejection OR breaks plugin functionality (missing headers, wrong text domain, missing uninstall cleanup, direct DB table creation without dbDelta)
  - WARNING: Non-standard patterns that cause maintainability issues or compatibility problems (global namespace pollution, missing prefixing, hardcoded paths, tight coupling)
  - INFO: Best practice improvements (could use register_activation_hook instead of manual check, missing flush_rewrite_rules on activation, etc.)
- **No additional fields beyond what security/performance use** — keep format consistent across all skills

### Reference doc structure
- Follow the performance/security skill pattern: reference docs are **deep-dive companions** to SKILL.md
- Each reference doc is a **cookbook** — quick reference table + detailed patterns + edge cases + WP nuances
- Four reference docs planned:
  - `architecture-patterns.md` — Plugin file organization, singleton vs static patterns, autoloading, dependency injection, main plugin class patterns, conditional loading
  - `hooks-guide.md` — Action/filter lifecycle, priority system, hook removal patterns, custom hooks, pluggable functions, hook naming conventions
  - `oop-patterns.md` — Class-based plugin architecture, namespaces, autoloading (PSR-4), trait usage, abstract classes for extensibility, service container patterns
  - `api-patterns.md` — Settings API complete workflow, Options API best practices, REST API endpoint registration (non-security aspects like schema validation, sanitize_callback, default values), Transients API for plugin data, custom database tables with dbDelta
- ❌ BAD / ✅ GOOD code pairs throughout, WordPress PHP Coding Standards

### Quick scan vs full review boundary
- `/wp-plugin` (quick scan): Grep-based pattern detection — scan for missing plugin headers, missing text domain, unprefixed functions/classes, direct file access without ABSPATH check, deprecated functions, missing activation/deactivation hooks, global namespace pollution
- `/wp-plugin-review` (full review): Complete skill workflow — file structure analysis, architecture review, WordPress.org compliance check, hooks analysis, i18n audit, Settings API review, produces structured report
- Same boundary pattern as security/performance skills

### Claude's Discretion
- Exact grep patterns for quick detection (tuned during implementation)
- Ordering of check categories within SKILL.md sections
- Depth of OOP pattern coverage (enough to guide good architecture, not a PHP design patterns textbook)
- Whether to include a "Migration Patterns" section for classic-to-modern plugin updates (likely brief mention only)
- Platform context section depth (WordPress.org vs private distribution)

</decisions>

<specifics>
## Specific Ideas

- Use the existing `wp-performance-review` SKILL.md as the structural template — same 12-section layout
- The description field in YAML frontmatter should include trigger phrases: "plugin review", "plugin development", "plugin architecture", "WordPress.org", "plugin check", "plugin submission", "plugin standards", "hooks", "actions", "filters", "Settings API", "custom post type", "taxonomy", "internationalization", "i18n", "text domain", "plugin headers"
- Plugin Check (PCP) tool is the official WordPress.org compliance checker — the skill should align with its checks so developers can pre-validate before submission
- Slash commands follow the exact same pattern as security/performance skills
- Plugin findings should be constructive — "use register_activation_hook() instead" not just "activation handling is wrong"

</specifics>

<deferred>
## Deferred Ideas

None — all plugin development concerns fit within phase scope

</deferred>

---

*Phase: 02-plugin-development*
*Context gathered: 2026-02-06*
