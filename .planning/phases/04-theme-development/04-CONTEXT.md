# Phase 4: Theme Development - Context

**Gathered:** 2026-02-06
**Status:** Ready for planning

<domain>
## Phase Boundary

Deliver the `wp-theme-development` skill for WordPress theme code review -- block theme structure (FSE), theme.json v3 validation, template hierarchy (HTML block templates and classic PHP templates), template parts, global styles, style variations, block patterns in themes, and classic-to-block theme migration patterns. Targets WP 6.6+ but includes classic theme coverage for migration guidance. Code-level review only -- no theme deployment, marketplace management, or visual design assessment.

</domain>

<decisions>
## Implementation Decisions

### Detection scope & depth
- **Deep coverage (dedicated sections in SKILL.md):** theme.json v3 structure validation (version, settings, styles, patterns, templateParts, customTemplates), template hierarchy for block themes (templates/ directory with .html files: index.html, single.html, page.html, archive.html, 404.html, search.html, home.html, front-page.html), template parts (parts/ directory: header.html, footer.html, sidebar.html with area designation), FSE templates with block markup, global styles (color palettes, typography scales, spacing presets, layout settings, border and shadow), style variations (styles/ directory with alternate theme.json files), block patterns in themes (patterns/ directory with PHP files and pattern headers), useRootPaddingAwareAlignments and layout system, child theme patterns for block themes
- **Surface-level coverage (scan patterns + brief guidance):** Navigation block configuration for menus, custom template registration via customTemplates in theme.json, block theme fonts management (fontFamilies in theme.json, local font hosting), theme.json v2 vs v3 differences (flag v2 with upgrade path), add_theme_support() calls still valid in block themes (wp-block-styles, responsive-embeds, editor-styles), classic theme conditional tags (is_front_page, is_singular, etc.), classic get_template_part() patterns, widget areas (classic themes only -- block themes use template parts)
- **Context-aware detection:** Distinguish between:
  - Block theme (has templates/index.html + theme.json + style.css with minimal content)
  - Classic theme (has index.php + style.css + functions.php with template tags)
  - Hybrid theme (classic theme with theme.json for partial block support -- growing pattern)
  - Child theme (inherits from parent -- different review concerns around overrides and compatibility)
  - WordPress.org theme directory submission (stricter standards -- Theme Check requirements)

### Block theme vs classic theme emphasis
- **Block themes are primary focus** -- WP 6.6+ is the target, block themes are the modern standard
- **Classic themes get migration guidance** -- many production sites still use classic themes, so migration path coverage is valuable
- **Hybrid themes acknowledged** -- classic themes can adopt theme.json incrementally, skill should detect and guide this pattern
- Auto-detect theme type from file structure: templates/index.html = block theme, index.php without templates/ = classic theme, both = hybrid
- Adjust review guidance per detected type: block theme gets FSE-focused review, classic gets migration suggestions, hybrid gets incremental adoption guidance

### theme.json depth & version handling
- **v3 is primary** (WP 6.6+) -- all examples and deep coverage use v3 syntax
- **v2 flagged with upgrade notes** -- if theme.json has "version": 2, flag as WARNING with specific v2→v3 migration steps
- **v1 flagged as deprecated** -- CRITICAL severity, must upgrade
- Deep coverage of theme.json sections:
  - `settings` -- color (palette, gradients, duotone, custom, defaultPalette), typography (fontSizes, fontFamilies, customFontSize, fontWeight, letterSpacing, textDecoration), spacing (padding, margin, blockGap, units), layout (contentSize, wideSize), border (color, radius, style, width), shadow (presets, defaultPresets), dimensions (aspectRatio, minHeight), position (sticky)
  - `styles` -- root-level styles, elements (heading, link, button, caption, cite), blocks (per-block style overrides), typography, color, spacing, border
  - `templateParts` -- name, title, area (header/footer/uncategorized)
  - `customTemplates` -- name, title, postTypes array
  - `patterns` -- array of pattern slugs from WordPress.org pattern directory
- **Validation rules:** Required fields, valid value types, deprecated properties, unknown properties detection

### Style system & anti-patterns
- **Hardcoded styles detection (THM-14):** Flag inline style attributes in templates, <style> tags in template files, hardcoded color values instead of theme.json presets, hardcoded font sizes instead of fluid typography, hardcoded spacing instead of spacing presets
- **Block stylesheets (THM-15):** Per-block CSS files for conditional loading (wp_enqueue_block_style), style.css minimal approach for block themes, proper use of theme.json for styling instead of manual CSS
- **useRootPaddingAwareAlignments (THM-20):** Required for proper full-width alignment handling, common gotcha when layouts break at wide/full widths
- **Style variations (THM-19):** styles/ directory with alternate theme.json files, proper structure validation, naming conventions
- Severity: Hardcoded styles in block themes = WARNING (defeats theme.json purpose), missing root padding aware alignments = INFO (layout enhancement)

### Cross-references to existing skills
- **wp-security-review:** Template escaping in classic themes (esc_html in template output), child theme security (parent function overrides), sanitization in theme customizer callbacks
- **wp-plugin-development:** Theme hooks (after_setup_theme, wp_enqueue_scripts, init for CPT registration in themes), functions.php as a mini-plugin, theme activation hooks
- **wp-block-development:** Block patterns registration (theme patterns/ vs plugin-registered patterns), block theme templates use block markup, block supports defined in theme.json settings
- **wp-performance-review:** Asset enqueuing in themes, conditional loading, image optimization with theme add_theme_support calls
- Same cross-reference pattern as Phases 2 and 3: brief mention + pointer, don't duplicate

### Reference doc structure
- Follow established cookbook format: quick reference table + detailed patterns + edge cases + WP nuances
- **Four reference docs:**
  - `theme-json-guide.md` -- Complete v3 schema reference with every section explained, property catalog with valid values, settings/styles/templateParts/customTemplates/patterns, v2→v3 migration notes, common mistakes, WordPress.org theme review requirements for theme.json
  - `template-patterns.md` -- Template hierarchy for both classic and block themes side-by-side, template parts and areas, conditional tags, get_template_part() for classic, block markup in HTML templates for block themes, custom templates registration
  - `fse-guide.md` -- Full Site Editing comprehensive guide: site editor workflow, global styles UI mapping to theme.json, style variations creation, block patterns in themes (patterns/ directory with PHP headers), navigation configuration, font management, layout system (contentSize, wideSize, root padding)
  - `classic-to-block-guide.md` -- Step-by-step migration: functions.php → theme.json mapping table, template.php → template.html conversion, sidebar widgets → template parts, Customizer → global styles, add_theme_support → theme.json settings equivalents, incremental adoption path for hybrid approach
- BAD/GOOD pairs throughout in PHP + HTML template markup, WordPress PHP Coding Standards

### Finding output style
- Mirror all prior skills exactly:
  - Group findings by FILE (PHP, HTML template files, theme.json intermixed by actual file path)
  - Each finding: line number, severity, issue description, explanation, BAD/GOOD code pair
  - Summary at end with total counts by severity
- **Severity mapping for themes:**
  - CRITICAL: Theme won't work or will be rejected from WordPress.org theme directory (missing required files like index.php/templates/index.html, invalid theme.json version/structure, missing style.css header, template hierarchy violations)
  - WARNING: Theme works but has quality/compatibility issues (hardcoded styles in block theme, deprecated theme.json v2, missing style variations support, classic patterns where block patterns preferred, missing child theme compatibility considerations)
  - INFO: Best practice improvements (could use theme.json for settings instead of add_theme_support, missing custom template registration, style variation naming could be more descriptive, font could use local hosting instead of Google Fonts CDN)

### Quick scan vs full review boundary
- `/wp-theme` (quick scan): Grep-based pattern detection -- scan for missing required files, hardcoded styles in templates, deprecated theme.json version, missing style.css headers, inline styles in HTML templates, missing index.php safety files, add_theme_support calls that should be theme.json settings
- `/wp-theme-review` (full review): Complete skill workflow -- theme type detection, theme.json validation, template hierarchy review, style system analysis, pattern registration review, classic-to-block migration opportunities, produces structured report
- Same boundary pattern as all prior skills

### Claude's Discretion
- Exact grep patterns for quick detection (HTML template patterns may need different regex than PHP)
- Depth of Customizer coverage for classic themes (enough to identify and suggest migration, not a Customizer tutorial)
- How to handle starter themes and theme frameworks (Underscores, theme boilerplate generators)
- Block locking patterns in templates (lock certain blocks from user editing)
- Whether to include WordPress.org Theme Check plugin alignment details or just reference the tool

</decisions>

<specifics>
## Specific Ideas

- Use the existing skills (wp-security-review, wp-plugin-development, wp-block-development) as structural templates -- same 12-section layout
- The description field in YAML frontmatter should include trigger phrases: "theme review", "theme development", "theme.json", "block theme", "FSE", "Full Site Editing", "template parts", "template hierarchy", "global styles", "style variations", "classic theme", "child theme", "theme patterns", "theme.json validation", "hardcoded styles", "useRootPaddingAwareAlignments", "classic-to-block"
- Theme reviews must handle the duality of reviewing both PHP files (functions.php, classic templates) and HTML block template files (templates/, parts/) in the same codebase
- theme.json v3 is current for WP 6.6+ -- skill should flag v1 as CRITICAL (deprecated), v2 as WARNING (upgrade available)
- For block themes, style.css is primarily for metadata (Theme Name, Version, etc.) not actual styling -- this is a key paradigm shift from classic themes
- Patterns in themes (patterns/ directory) use PHP files with specific header comments, not to be confused with block patterns registered via PHP in plugins

</specifics>

<deferred>
## Deferred Ideas

None -- all theme development concerns fit within phase scope

</deferred>

---

*Phase: 04-theme-development*
*Context gathered: 2026-02-06*
