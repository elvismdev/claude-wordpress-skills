# Phase 3: Gutenberg Blocks - Context

**Gathered:** 2026-02-06
**Status:** Ready for planning

<domain>
## Phase Boundary

Deliver the `wp-block-development` skill for WordPress block code review -- block.json validation, editor component patterns (React/JSX), server-side rendering (PHP render_callback), block deprecations, block variations, and Interactivity API directives (WP 6.5+). Covers both PHP and JavaScript/React code. Code-level review only -- no block deployment, marketplace distribution, or full React application patterns outside the WordPress block editor context.

</domain>

<decisions>
## Implementation Decisions

### Detection scope & depth
- **Deep coverage (dedicated sections in SKILL.md):** block.json schema validation (every field: apiVersion, name, title, category, icon, attributes, supports, editorScript, viewScript, render, etc.), register_block_type() registration patterns, dynamic blocks with render_callback / render PHP file, static blocks with save() JSX, block attributes (type, source, selector, default), useBlockProps hook (editor + save), InnerBlocks with allowedBlocks/template, InspectorControls/BlockControls for sidebar/toolbar, block deprecations (migrate + save pairs), Interactivity API directives (data-wp-bind, data-wp-on, data-wp-context, data-wp-watch, data-wp-init, wp_interactivity_state(), wp_interactivity_config())
- **Surface-level coverage (scan patterns + brief guidance):** block transforms (from/to other blocks), block variations (variation picker), block styles (style variations registered in block.json or PHP), SlotFill for extending existing blocks, format types (RichText toolbar extensions), block patterns (PHP-registered block template patterns -- not to be confused with code patterns), block templates and template lock, @wordpress/scripts build toolchain (wp-scripts build/start/lint), block hooks (WP 6.4+ auto-insertion)
- **Context-aware detection:** Distinguish between:
  - Single block plugin (one block, simple structure)
  - Multi-block plugin (shared components, build config for multiple blocks)
  - Block library/collection (published as package, namespacing important)
  - Theme blocks (blocks registered within a theme, different loading context)

### JavaScript/React coverage
- **This is the first skill covering JavaScript/React** -- prior skills were PHP-only
- **Block editor patterns only** -- NOT general React patterns. Cover WordPress-specific APIs: useBlockProps, RichText, InspectorControls, BlockControls, InnerBlocks, useSelect, useDispatch (from @wordpress/data), useEntityProp
- **BAD/GOOD code pairs work for JSX** -- same visual pattern, just different syntax. Use ES6+ modern JS conventions matching WordPress JS coding standards
- **WordPress JS coding standards** in examples: tab indentation in JS files, JSDoc comments, camelCase for variables/functions, PascalCase for components
- **Grep patterns must cover both PHP and JS/JSX files** -- quick scan searches *.php + *.js + *.jsx + *.tsx patterns
- **No TypeScript depth** -- mention that TS is supported but don't make it primary. WordPress core uses JSDoc types, not TypeScript
- **Import patterns** -- detect @wordpress/* package imports (correct) vs window.wp.* global access (legacy/incorrect for blocks)

### Interactivity API treatment
- **Dedicated reference doc** (`interactivity-api-guide.md`) -- Interactivity API is a fundamentally different paradigm from React-based editor code and deserves full treatment
- **WP 6.5+ version marker** throughout -- clearly indicate this is a newer API, not available in earlier WordPress versions
- **Key distinction:** Interactivity API is for the FRONTEND (view), not the editor. Editor still uses React. This is the most common point of confusion.
- **Cover directives:** data-wp-bind, data-wp-on, data-wp-class, data-wp-style, data-wp-text, data-wp-context, data-wp-watch, data-wp-init, data-wp-each, data-wp-interactive
- **Server-side state:** wp_interactivity_state() and wp_interactivity_config() in render_callback
- **Cross-reference:** When Interactivity API patterns involve security (e.g., user input in state), reference wp-security-review

### Reference doc structure
- Follow the established cookbook format: quick reference table + detailed patterns + edge cases + WP nuances
- **Four reference docs organized by concern (not by language):**
  - `block-json-guide.md` -- Complete block.json schema reference with every field explained, validation rules, common mistakes, apiVersion differences (2 vs 3), supports flags catalog
  - `editor-patterns.md` -- React/JSX patterns for the block editor: useBlockProps, RichText, InspectorControls, BlockControls, InnerBlocks, attributes handling, useSelect/useDispatch, custom hooks, toolbar/sidebar controls
  - `dynamic-blocks-guide.md` -- Server-side rendering patterns: render_callback vs render PHP file, WP_Block object, block context, $attributes parameter, dynamic vs static decision guide, hybrid blocks (static save + server enhancements)
  - `interactivity-api-guide.md` -- WP 6.5+ Interactivity API: directive catalog, state management, context passing, server-side setup, when to use vs custom JS
- **BAD/GOOD pairs throughout** -- PHP examples follow WordPress PHP Coding Standards; JS/JSX examples follow WordPress JS coding standards

### Security and plugin skill crossover
- **Do NOT duplicate security or plugin patterns** -- cross-reference wp-security-review and wp-plugin-development
- **Block-specific security:** Escape output in render_callback (esc_html, esc_attr, wp_kses), sanitize block attributes in PHP, validate attribute types. Brief reminder + cross-reference.
- **Block-specific plugin patterns:** register_block_type() is a plugin registration function -- brief mention of proper init hook usage + cross-reference wp-plugin-development for plugin lifecycle depth
- **Slash command integration:** If /wp-block-review finds security concerns, suggest /wp-sec-review. If plugin architecture issues found, suggest /wp-plugin-review.

### Finding output style
- Mirror the security/performance/plugin skill output format exactly:
  - Group findings by FILE (PHP and JS files intermixed by actual file path)
  - Each finding: line number, severity, issue description, explanation, BAD/GOOD code pair
  - Summary at end with total counts by severity
- **Severity mapping for blocks:**
  - CRITICAL: Block won't render OR crashes editor (missing apiVersion, invalid attribute type, save function mismatch causing block validation error, missing useBlockProps in save)
  - WARNING: Block works but has quality/compatibility issues (deprecated API usage, missing supports flags, no deprecation handler for changed save, hardcoded strings without i18n)
  - INFO: Best practice improvements (could use render PHP file instead of render_callback, missing viewScript for frontend interaction, not using block.json for asset registration)

### Quick scan vs full review boundary
- `/wp-block` (quick scan): Grep-based pattern detection across PHP and JS/JSX files -- scan for missing block.json fields, deprecated block API patterns, missing useBlockProps, render_callback without escaping, missing apiVersion, save function without useBlockProps.save, invalid attribute sources
- `/wp-block-review` (full review): Complete skill workflow -- block.json validation, editor pattern review, render callback analysis, deprecation handling check, i18n audit, produces structured report
- Same boundary pattern as security/performance/plugin skills

### Claude's Discretion
- Exact grep patterns for quick detection (tuned during implementation -- JS grep patterns may need different regex than PHP)
- Depth of @wordpress/scripts toolchain coverage (enough to identify misconfiguration, not a build system tutorial)
- Whether to include block theme integration patterns (blocks in theme context) or defer to Phase 4
- How to handle the JSX compilation question (source vs build files -- which to review)
- Block pattern (template) coverage depth vs block code patterns

</decisions>

<specifics>
## Specific Ideas

- Use the existing skills (wp-security-review, wp-plugin-development) as structural templates -- same 12-section layout
- The description field in YAML frontmatter should include trigger phrases: "block review", "Gutenberg", "block.json", "block development", "block editor", "useBlockProps", "InnerBlocks", "Interactivity API", "data-wp-bind", "render_callback", "dynamic block", "static block", "block deprecation", "block attributes", "block supports", "@wordpress/scripts", "wp-scripts"
- Block reviews must handle the duality of reviewing BOTH PHP files and JS/JSX files in the same codebase
- apiVersion 3 is current (WP 6.3+) -- skill should flag apiVersion 1 and 2 as outdated with upgrade guidance
- block.json is the single source of truth for modern blocks -- PHP-only registration (register_block_type with inline args) should be flagged as legacy pattern
- Interactivity API examples should use the directive syntax without JSX (it's PHP template output with HTML attributes)

</specifics>

<deferred>
## Deferred Ideas

None -- all Gutenberg block development concerns fit within phase scope

</deferred>

---

*Phase: 03-gutenberg-blocks*
*Context gathered: 2026-02-06*
