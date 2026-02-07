---
phase: 03-gutenberg-blocks
plan: 02
subsystem: reference-documentation
tags: [block-json, editor-patterns, dynamic-blocks, interactivity-api, react, jsx, server-rendering, reference-docs]

# Dependency graph
requires:
  - phase: 03-gutenberg-blocks
    plan: 01
    provides: SKILL.md foundation for reference doc content
provides:
  - Complete block.json schema reference (apiVersion differences, attributes, supports, assets)
  - React/JSX editor patterns reference (useBlockProps, RichText, controls, InnerBlocks, data API)
  - Dynamic blocks reference (render_callback, render file, WP_Block, InnerBlocks gotcha)
  - Interactivity API reference (WP 6.5+ directives, state management, context vs state)
affects: [03-03-commands, block-development-workflows, skill-depth]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Cookbook format for JavaScript/React reference docs (same as PHP references)"
    - "Complete directive catalog for Interactivity API with examples"
    - "Critical gotcha documentation: InnerBlocks with dynamic blocks MUST save InnerBlocks.Content"
    - "Security pattern: NEVER escape $content from InnerBlocks (breaks embeds)"
    - "State management distinction: global state vs scoped context"

key-files:
  created:
    - skills/wp-block-development/references/block-json-guide.md
    - skills/wp-block-development/references/editor-patterns.md
    - skills/wp-block-development/references/dynamic-blocks-guide.md
    - skills/wp-block-development/references/interactivity-api-guide.md
  modified: []

key-decisions:
  - "apiVersion 3 requires useBlockProps.save() - className NOT auto-injected (critical for migration guide)"
  - "DO NOT use wp_kses_post() on InnerBlocks $content - breaks core/embed blocks and oEmbed"
  - "Interactivity API viewScriptModule required (NOT viewScript) - ES module support for imports"
  - "Context vs state clearly distinguished: state=global/shared, context=local/scoped"
  - "Dynamic block InnerBlocks must return InnerBlocks.Content in save - #1 gotcha documented prominently"

patterns-established:
  - "Quick reference table at top of each reference doc for rapid lookup"
  - "BAD/GOOD code pairs work equally well for JavaScript/JSX as for PHP"
  - "Directive catalog format: table + per-directive sections with syntax/purpose/examples"
  - "Cross-references at bottom of each doc provide navigation between related concepts"
  - "Version markers (WP 6.5+) clearly indicate new API requirements"

# Metrics
duration: 10min
completed: 2026-02-06
---

# Phase 03 Plan 02: Block Development Reference Docs Summary

**Four comprehensive reference docs (5,191 lines total) providing deep-dive coverage of block.json schema, React/JSX editor patterns, dynamic rendering, and Interactivity API**

## Performance

- **Duration:** 10 min
- **Started:** 2026-02-06T23:40:45Z
- **Completed:** 2026-02-06T23:51:40Z
- **Tasks:** 2
- **Files created:** 4

## Accomplishments

- Created 4 reference docs totaling 5,191 lines (exceeding 2000+ line target)
- block-json-guide.md (1,452 lines): Complete block.json schema with apiVersion 1/2/3 differences, attributes deep dive, supports catalog, asset registration
- editor-patterns.md (1,507 lines): React/JSX patterns covering useBlockProps, RichText, controls (InspectorControls/BlockControls), InnerBlocks, useSelect/useDispatch, deprecations
- dynamic-blocks-guide.md (1,031 lines): Server-side rendering with render_callback vs render file, WP_Block object, InnerBlocks critical gotcha, escaping rules
- interactivity-api-guide.md (1,201 lines): WP 6.5+ Interactivity API with complete directive catalog, state management, context vs state, integration patterns
- All docs follow cookbook format: quick reference table + detailed patterns + edge cases + BAD/GOOD pairs
- Cross-references between docs create navigation paths
- Security and plugin architecture cross-references to avoid duplication

## Task Commits

Each task was committed atomically:

1. **Task 1: Create block-json-guide.md and editor-patterns.md** - `d4ec108` (feat) - *Note: Committed in prior execution*
2. **Task 2: Create dynamic-blocks-guide.md and interactivity-api-guide.md** - `c3cc744` (feat)

## Files Created/Modified

- `skills/wp-block-development/references/block-json-guide.md` - Complete block.json metadata reference (1,452 lines) with $schema for IDE validation, apiVersion 1/2/3 differences and migration guide, complete attributes schema (all types, sources, selectors), supports catalog (color, spacing, typography, alignment, etc.), asset registration (editorScript/script/viewScript/viewScriptModule), categories/icons/variations, blockHooks (WP 6.4+), render property, common mistakes table

- `skills/wp-block-development/references/editor-patterns.md` - React/JSX editor patterns reference (1,507 lines) with @wordpress/* packages quick reference, useBlockProps() and useBlockProps.save() patterns, useInnerBlocksProps hook order (critical), RichText component with allowedFormats, InspectorControls (settings sidebar) with group prop (WP 6.2+), BlockControls (toolbar) with common components, InnerBlocks patterns (allowedBlocks, template, templateLock, orientation), InnerBlocks with dynamic blocks gotcha, attributes and state management, useSelect/useDispatch data API (performance: single useSelect call), useEntityProp for post meta, block deprecation patterns (apiVersion in deprecations, migration function, isEligible), import patterns (@wordpress/* vs window.wp.*), @wordpress/scripts build process, i18n patterns, anti-patterns section

- `skills/wp-block-development/references/dynamic-blocks-guide.md` - Server-side rendering reference (1,031 lines) with static vs dynamic decision guide and decision tree, render_callback pattern (function signature, accessing attributes, using $content, WP_Block object), render PHP file pattern (file resolution, precedence over render_callback), get_block_wrapper_attributes() with merging, WP_Block properties (name, context, parsed_block, block_type), block context in PHP, InnerBlocks with dynamic blocks CRITICAL gotcha (must return InnerBlocks.Content in save, $content contains inner blocks), escaping rules (DO NOT use wp_kses_post on $content - breaks embeds), hybrid blocks (static save + dynamic enhancement), querying data (WP_Query, caching with transients, avoiding N+1), common mistakes table

- `skills/wp-block-development/references/interactivity-api-guide.md` - WP 6.5+ Interactivity API reference (1,201 lines) with complete data-wp-* directive catalog (10 directives), setup requirements (viewScriptModule NOT viewScript, wp_interactivity_state(), store()), architecture overview (frontend only, not editor), directive catalog with examples (interactive, bind, on, class, style, text, context, watch, init, each), state management (server-side with wp_interactivity_state, client-side with store), actions (user-triggered) vs callbacks (side effects), context vs state distinction (global/shared vs local/scoped), integration with dynamic blocks, security (never sensitive data in state), common mistakes, when NOT to use Interactivity API, WP 6.5+ version markers throughout

## Decisions Made

**apiVersion 3 className Injection:**
- apiVersion 3 blocks do NOT auto-inject className in save function
- MUST explicitly call useBlockProps.save() to get wrapper attributes
- Migration guide includes deprecation pattern for apiVersion 2 → 3
- Rationale: Critical for block validation, prevents "unexpected content" errors

**InnerBlocks with Dynamic Blocks:**
- Documented as #1 gotcha: dynamic blocks with InnerBlocks MUST return InnerBlocks.Content in save
- Why: InnerBlocks are saved to database from save function, render_callback receives via $content
- Lifecycle: edit → save (writes inner blocks) → render_callback (receives $content)
- Rationale: Most common block validation error, needs prominent documentation

**Escaping $content from InnerBlocks:**
- DO NOT use wp_kses_post() on $content parameter
- Why: Breaks core/embed blocks (YouTube, Twitter) and oEmbed processing
- Inner blocks already sanitized by WordPress during save process
- Exception documented: If certain no embeds and need restricted HTML
- Rationale: Common mistake causing embed failures, security vs functionality balance

**viewScriptModule vs viewScript:**
- Interactivity API requires viewScriptModule, NOT viewScript
- Why: Needs ES module support for import statements (@wordpress/interactivity)
- viewScript loads as classic script (no imports support)
- Rationale: Technical requirement, common mistake causing import failures

**Context vs State Distinction:**
- State: Global to namespace, shared across all instances
- Context: Local to DOM subtree, scoped to specific instance via data-wp-context
- Use state for shared data, context for instance-specific data
- Rationale: Fundamental architecture concept, prevents confusion

## Deviations from Plan

None - plan executed exactly as written. All 4 reference docs created with appropriate depth, cookbook format, and cross-references.

## Issues Encountered

None - 03-RESEARCH.md provided comprehensive verified code examples and patterns. Prior skill reference docs established effective cookbook format.

## Next Phase Readiness

**Ready for 03-03:**
- Reference docs provide deep-dive content for slash commands to delegate to
- Complete directive catalog supports /wp-block-review Interactivity API checks
- Dynamic block patterns support detection of render_callback vs render file
- InnerBlocks gotcha can be flagged in quick scan (/wp-block) and detailed review
- Cross-reference structure supports command delegation pattern

**No blockers identified.**

## Self-Check: PASSED

Verified files exist:
- FOUND: skills/wp-block-development/references/block-json-guide.md (1,452 lines)
- FOUND: skills/wp-block-development/references/editor-patterns.md (1,507 lines)
- FOUND: skills/wp-block-development/references/dynamic-blocks-guide.md (1,031 lines)
- FOUND: skills/wp-block-development/references/interactivity-api-guide.md (1,201 lines)

Verified commits exist:
- FOUND: d4ec108 (block-json-guide, editor-patterns - from prior execution)
- FOUND: c3cc744 (dynamic-blocks-guide, interactivity-api-guide)

---
*Phase: 03-gutenberg-blocks*
*Completed: 2026-02-06*
