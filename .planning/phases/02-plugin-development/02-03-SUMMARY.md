---
phase: 02-plugin-development
plan: 03
subsystem: documentation
tags: [wordpress, plugin-development, slash-commands, cli]

# Dependency graph
requires:
  - phase: 02-plugin-development
    plan: 01
    provides: wp-plugin-development skill name and workflow sections
provides:
  - /wp-plugin-review slash command for full plugin architecture review
  - /wp-plugin slash command for quick grep-based plugin standards check
affects: [end-user-workflow]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Slash command delegation pattern (mirrors wp-sec-review/wp-sec)"
    - "Security cross-reference in full review command"

key-files:
  created:
    - commands/wp-plugin-review.md
    - commands/wp-plugin.md
  modified: []
---

/wp-plugin-review delegates to wp-plugin-development skill's full Code Review Workflow with security cross-reference; /wp-plugin delegates to Search Patterns for Quick Detection section for grep-based triage.

## Performance

| Metric | Value |
|--------|-------|
| Duration | ~1 minute |
| Tasks | 1 |
| Files created | 2 |

## Accomplishments

- Created `/wp-plugin-review` command delegating to full plugin architecture review workflow
- Created `/wp-plugin` command delegating to grep-based quick plugin standards check
- Both commands follow exact pattern established by security and performance skills
- wp-plugin-review suggests /wp-sec-review for security concerns found during review
- wp-plugin suggests /wp-plugin-review for deeper analysis after quick scan

## Task Commits

| Commit | Description |
|--------|-------------|
| 500bce6 | feat(02-03): create /wp-plugin-review and /wp-plugin slash commands |

## Self-Check

- [x] commands/wp-plugin-review.md exists with valid YAML frontmatter
- [x] commands/wp-plugin.md exists with valid YAML frontmatter
- [x] wp-plugin-review.md references `wp-plugin-development` skill by name
- [x] wp-plugin.md references "Search Patterns for Quick Detection" section
- [x] wp-plugin-review.md suggests /wp-sec-review for security concerns
- [x] wp-plugin.md suggests /wp-plugin-review for deeper analysis
- [x] Both files follow exact pattern of security/performance counterparts

## Deviations from Plan

None.

## Issues Encountered

None.
