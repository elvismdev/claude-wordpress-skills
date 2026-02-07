---
phase: 03-gutenberg-blocks
plan: 01
subsystem: skill-development
tags: [gutenberg, block-editor, react, jsx, block-json, interactivity-api, wordpress-blocks]

# Dependency graph
requires:
  - phase: 02-plugin-development
    provides: Plugin architecture patterns and cross-reference structure
  - phase: 01-security-review
    provides: Security patterns for cross-referencing
provides:
  - Complete block development review skill covering PHP and JavaScript/React
  - block.json schema validation patterns
  - Editor component patterns (useBlockProps, RichText, InspectorControls, BlockControls)
  - Static vs dynamic block detection
  - InnerBlocks usage patterns
  - Block deprecation handling
  - Interactivity API directive review (WP 6.5+)
  - Grep patterns for both PHP and JS/JSX files
affects: [03-02-reference-docs, 03-03-commands, block-review-workflows]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "First skill covering both PHP and JavaScript/React code"
    - "Dual-architecture block pattern: React for editor, HTML/PHP for frontend"
    - "block.json as single source of truth"
    - "BAD/GOOD pairs for both PHP (WordPress PHP standards) and JS/JSX (WordPress JS standards)"
    - "Grep patterns cover both PHP and JS/JSX files with different regex"
    - "Context-aware detection: single block, multi-block, block library, theme blocks"

key-files:
  created:
    - skills/wp-block-development/SKILL.md
  modified: []

key-decisions:
  - "First skill to cover JavaScript/React alongside PHP - established pattern for dual-language skills"
  - "Interactivity API (WP 6.5+) gets dedicated coverage with WP version markers throughout"
  - "Source file review (src/) vs build file review (build/) - decided to review src/ and flag missing/stale build/"
  - "Security cross-references for escaping patterns, plugin cross-references for architecture patterns - no duplication"
  - "Grep patterns need different regex for JS files vs PHP files - documented in pattern comments"

patterns-established:
  - "12-section structure works for dual-language skills (PHP + JS/JSX)"
  - "BAD/GOOD code pairs effective for both WordPress PHP Coding Standards and WordPress JS coding standards"
  - "File-Type Specific Checks section handles both .php and .js/.jsx files"
  - "Search Patterns section provides separate grep commands for PHP and JS/JSX with appropriate flags"
  - "Context detection identifies single/multi-block/library/theme contexts"
  - "Version Compatibility Reference table maps features to WordPress version requirements"

# Metrics
duration: 5min
completed: 2026-02-06
---

# Phase 03 Plan 01: Core Block Development Skill Summary

**Complete wp-block-development SKILL.md (1,444 lines) covering block.json schema, React/JSX editor patterns, static/dynamic blocks, InnerBlocks, deprecations, and Interactivity API with dual PHP/JS grep patterns**

## Performance

- **Duration:** 5 min
- **Started:** 2026-02-06T23:38:51Z
- **Completed:** 2026-02-06T23:44:03Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments

- Created comprehensive block development review skill covering BOTH PHP and JavaScript/React code (first dual-language skill)
- 12 sections covering block.json validation, editor components (useBlockProps, RichText, InspectorControls, BlockControls), static/dynamic blocks, InnerBlocks patterns, block deprecations, Interactivity API directives, server-side rendering, attribute validation, and build toolchain
- Grep patterns for quick detection covering both PHP files (--include="*.php") and JS/JSX files (--include="*.js" --include="*.jsx") with different regex patterns
- Context-aware detection for single block plugins, multi-block plugins, block libraries, and theme blocks
- Cross-references to wp-security-review for security depth and wp-plugin-development for plugin architecture
- BAD/GOOD code pairs following WordPress PHP Coding Standards (spaces in parentheses, array() not [], Yoda conditions) and WordPress JS coding standards (tab indentation, camelCase, PascalCase)

## Task Commits

Each task was committed atomically:

1. **Task 1: Create SKILL.md with YAML frontmatter and 12-section structure** - `ee81f2f` (feat)

## Files Created/Modified

- `skills/wp-block-development/SKILL.md` - Core block development review skill (1,444 lines) with block.json schema validation, React/JSX editor patterns, static vs dynamic block detection, InnerBlocks usage, block deprecation handling, Interactivity API directives (WP 6.5+), server-side rendering patterns, attribute type/source/selector validation, import patterns (@wordpress/* vs window.wp.*), block context detection, @wordpress/scripts build config, grep patterns for PHP and JS/JSX, severity definitions, output format, common false positives, version compatibility reference, deep-dive reference links

## Decisions Made

**Source vs Build Review:**
- Review src/ files for code patterns (developer intent)
- Flag if build/ directory missing or stale (check index.asset.php timestamp)
- Do NOT review build/ files for code quality (they are compiled output)
- Rationale: Source code reveals intent, build files are generated artifacts

**Grep Pattern Differences:**
- JS grep patterns use different regex than PHP patterns
- Document with `--include="*.js" --include="*.jsx"` for JS files
- Document with `--include="*.php"` for PHP files
- Rationale: File extensions and pattern syntax differ between languages

**Interactivity API Treatment:**
- WP 6.5+ version markers throughout
- Key distinction emphasized: Interactivity API = FRONTEND only, editor still uses React
- Dedicated reference doc planned for 03-02
- Rationale: New API (WP 6.5+) needs clear scope boundaries and version awareness

**Security and Plugin Crossover:**
- Brief reminders for security-relevant patterns (unescaped output, user input in state)
- Cross-reference wp-security-review for depth
- Brief mentions for plugin patterns (register_block_type, ABSPATH)
- Cross-reference wp-plugin-development for architecture depth
- Rationale: Avoid duplication, maintain skill focus, provide navigation path

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None - research and existing skill structures provided clear guidance.

## Next Phase Readiness

**Ready for 03-02:**
- SKILL.md provides complete foundation for reference docs
- BLK-02 through BLK-25 requirements are mapped and covered
- Reference doc structure defined: block-json-guide.md, editor-patterns.md, dynamic-blocks-guide.md, interactivity-api-guide.md
- Cookbook format proven effective in prior phases

**Ready for 03-03:**
- /wp-block-review and /wp-block command structure clear from security/plugin precedent
- Quick scan (grep-based) vs full review (skill workflow) boundary established
- Delegation pattern works: /wp-block-review can suggest /wp-sec-review or /wp-plugin-review

**No blockers identified.**

## Self-Check: PASSED

Verified files exist:
- FOUND: skills/wp-block-development/SKILL.md

Verified commits exist:
- FOUND: ee81f2f

---
*Phase: 03-gutenberg-blocks*
*Completed: 2026-02-06*
