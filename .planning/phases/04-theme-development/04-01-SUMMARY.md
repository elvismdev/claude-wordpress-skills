---
phase: 04-theme-development
plan: 01
subsystem: wordpress-theme
tags: [wordpress, theme-development, block-theme, fse, theme.json, full-site-editing, template-hierarchy, global-styles, style-variations]

# Dependency graph
requires:
  - phase: 03-gutenberg-blocks
    provides: Block development skill with block markup patterns and InnerBlocks usage referenced in theme templates
  - phase: 02-plugin-development
    provides: Plugin architecture skill with hooks patterns referenced in functions.php
  - phase: 01-security-review
    provides: Security review skill with escaping patterns referenced in template output
provides:
  - Complete wp-theme-development skill with theme.json v3 validation, block template review, template parts, global styles, style variations, and classic-to-block migration
  - Theme type auto-detection (block/classic/hybrid/child/WordPress.org)
  - useRootPaddingAwareAlignments validation for full-width alignment
  - Hardcoded styles detection in block themes
affects: [04-02-theme-json-guide, 04-03-template-patterns, slash-commands]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "theme.json v3 schema validation with version/settings/styles/templateParts/customTemplates"
    - "Block template hierarchy review (HTML files with block markup)"
    - "Template parts validation (parts/ directory with area designation)"
    - "Theme type detection from file structure (templates/index.html vs index.php vs hybrid)"
    - "useRootPaddingAwareAlignments checking when root padding present"
    - "Cross-language review (PHP, HTML templates, JSON files in single report)"

key-files:
  created:
    - skills/wp-theme-development/SKILL.md
  modified: []

key-decisions:
  - "Block themes are primary focus (WP 6.6+ with theme.json v3), classic themes covered for migration guidance"
  - "Theme type auto-detection adjusts review guidance (block/classic/hybrid/child/WordPress.org)"
  - "theme.json v3 is primary with v2 WARNING and v1 CRITICAL severity"
  - "Hardcoded styles in block themes = WARNING (defeats FSE purpose)"
  - "useRootPaddingAwareAlignments validation when root-level padding present"
  - "Cross-reference pattern maintained: brief mention + pointer to other skills"

patterns-established:
  - "Pattern: theme.json v3 complete schema with settings (color, typography, spacing, layout, border, shadow)"
  - "Pattern: Block template hierarchy (templates/*.html with block markup, not PHP)"
  - "Pattern: Template parts registration in theme.json matching parts/ directory files"
  - "Pattern: Style variations in styles/ directory with alternate theme.json files"
  - "Pattern: Block patterns in patterns/ directory with PHP files and pattern headers"
  - "Pattern: Theme type detection from file existence (templates/index.html + theme.json = block theme)"
  - "Pattern: BAD/GOOD code pairs with PHP (WordPress PHP Coding Standards), HTML (block markup), JSON (theme.json v3)"

# Metrics
duration: 5min
completed: 2026-02-06
---

# Phase 4 Plan 1: Create wp-theme-development SKILL.md Summary

**Complete theme.json v3 validation skill with block template review, template parts, global styles, style variations, theme type auto-detection, and classic-to-block migration patterns**

## Performance

- **Duration:** 5 min
- **Started:** 2026-02-06T21:53:49Z
- **Completed:** 2026-02-06T21:58:47Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments
- Created 1,243-line wp-theme-development SKILL.md with 12 sections matching prior skills structure
- theme.json v3 validation (version, settings, styles, templateParts, customTemplates) with v2→v3 migration guidance
- Block template hierarchy review (templates/*.html with block markup, not PHP)
- Template parts validation (parts/ directory with area designation: header/footer/uncategorized)
- useRootPaddingAwareAlignments validation for proper full-width block alignment
- Theme type auto-detection (block/classic/hybrid/child/WordPress.org) with context-aware guidance
- Hardcoded styles detection in block themes (inline styles, hardcoded hex colors, pixel font sizes)
- Cross-references to wp-security-review (template escaping), wp-plugin-development (hooks), wp-block-development (block markup)

## Task Commits

Each task was committed atomically:

1. **Task 1: Create SKILL.md with YAML frontmatter and 12-section structure** - `1fa10f7` (feat)

## Files Created/Modified
- `skills/wp-theme-development/SKILL.md` - Complete theme development review skill (1,243 lines) with theme.json v3 validation, block template hierarchy, template parts, global styles, style variations, block patterns, child themes, classic-to-block migration

## Decisions Made

**Theme development architecture:**
- Block themes primary focus (WP 6.6+ with theme.json v3), classic themes secondary for migration context
- Theme type auto-detection from file structure adjusts review guidance
- theme.json v3 is current standard (v2 = WARNING, v1 = CRITICAL)
- useRootPaddingAwareAlignments essential when root padding present

**Hardcoded styles severity:**
- Hardcoded inline styles in block theme templates = WARNING (defeats theme.json purpose, prevents user customization)
- Hardcoded color/font values in style.css = WARNING (should use CSS variables from theme.json)
- Missing useRootPaddingAwareAlignments with root padding = WARNING (full-width blocks won't reach viewport edges)

**Cross-reference pattern maintained:**
- Template escaping → brief reminder + "See wp-security-review"
- functions.php hooks → brief mention + "See wp-plugin-development"
- Block markup → brief context + "See wp-block-development"
- No duplication of other skills' content

**Multi-language file review:**
- PHP files (functions.php, classic templates, pattern files)
- HTML template files (templates/, parts/)
- JSON files (theme.json, style variations in styles/)
- All intermixed in single report grouped by actual file path

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None - SKILL.md creation completed without issues.

## Next Phase Readiness

- SKILL.md complete with all 12 sections and required content
- Ready for 04-02 (theme.json reference doc)
- Cross-references to wp-security-review, wp-plugin-development, wp-block-development established
- Theme type detection patterns defined for context-aware reviews
- theme.json v3 validation patterns ready for reference doc expansion

## Self-Check: PASSED

**Files verified:**
- skills/wp-theme-development/SKILL.md exists (1,243 lines)

**Commits verified:**
- 1fa10f7 exists in git log

---
*Phase: 04-theme-development*
*Completed: 2026-02-06*
