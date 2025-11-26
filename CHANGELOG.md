# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Placeholder for upcoming changes

## [1.1.0] - 2025-11-25

### Added

- **Slash commands** for explicit skill invocation
  - `/wp-perf-review [path]` — Full WordPress performance code review
  - `/wp-perf [path]` — Quick triage scan (critical patterns only)

### Improved

- Commands explicitly reference skill workflow sections for better LLM orchestration
- Commands instruct Claude to use the skill's Output Format
- Commands differentiate between quick scan (grep patterns) and full review (deep analysis)
- Clear instructions for loading reference files when needed

## [1.0.0] - 2025-11-25

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

[Unreleased]: https://github.com/elvismdev/claude-wordpress-skills/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/elvismdev/claude-wordpress-skills/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/elvismdev/claude-wordpress-skills/releases/tag/v1.0.0
