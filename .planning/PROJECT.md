# Claude WordPress Skills — Complete Skills Build-Out

## What This Is

A Claude Code plugin that provides professional WordPress engineering skills for code review and development guidance. Currently has 1 of 6 planned skills implemented (wp-performance-review). This project builds out the remaining 5 skills — security review, Gutenberg blocks, theme development, plugin development, and WooCommerce development — each matching the depth and quality of the existing performance skill.

## Core Value

Every WordPress developer using Claude Code gets comprehensive, expert-level code review and development guidance across all major WordPress domains — not just performance.

## Requirements

### Validated

- ✓ wp-performance-review skill — existing (SKILL.md + 4 references + 2 slash commands)
- ✓ Plugin structure (.claude-plugin/, skills/, commands/) — existing
- ✓ Marketplace registration — existing
- ✓ README with skill table, installation instructions, usage examples — existing
- ✓ CONTRIBUTING.md with skill creation guidelines — existing

### Active

- [ ] wp-security-review skill with full SKILL.md, reference docs, and 2 slash commands (/wp-sec-review, /wp-sec)
- [ ] wp-gutenberg-blocks skill with full SKILL.md, reference docs, and 2 slash commands (/wp-block-review, /wp-block)
- [ ] wp-theme-development skill with full SKILL.md, reference docs, and 2 slash commands (/wp-theme-review, /wp-theme)
- [ ] wp-plugin-development skill with full SKILL.md, reference docs, and 2 slash commands (/wp-plugin-review, /wp-plugin)
- [ ] wp-woocommerce-dev skill with full SKILL.md, reference docs, and 2 slash commands (/wp-woo-review, /wp-woo)
- [ ] Update README.md to reflect all implemented skills (status ✅, trigger phrases, command docs)
- [ ] Update marketplace.json if needed for new skills
- [ ] Update plugin.json version bump
- [ ] Update CHANGELOG.md with new skills

### Out of Scope

- Server/infrastructure hardening (wp-security-review stays code-level only) — Claude reviews code, not server configs
- Multisite-specific skill — could be a future addition
- Automated testing/CI integration — this is a documentation-based skill plugin, no runtime code
- WordPress 5.x backward compatibility — targeting WP 6.x+ modern APIs and patterns

## Context

- Repository is purely documentation-based: Markdown skill definitions + JSON config, no runtime code
- Existing wp-performance-review is ~500 lines of SKILL.md + 4 reference docs (anti-patterns.md, wp-query-guide.md, caching-guide.md, measurement-guide.md)
- Each SKILL.md has YAML frontmatter (name, description with trigger phrases), structured sections, code examples following WordPress PHP Coding Standards
- Slash commands are simple Markdown files in commands/ referencing the parent skill
- Pattern: full review command (thorough) + quick scan command (grep-based triage)
- Audience: all WordPress developers — agency, freelancer, enterprise teams
- Codebase already mapped in .planning/codebase/

## Constraints

- **Frontmatter**: `description` field max 1024 characters, `name` max 64 characters
- **Code standards**: All PHP examples must follow WordPress PHP Coding Standards (spaces in parentheses, array() not [], Yoda conditions, snake_case, prefix custom functions)
- **Severity labels**: CRITICAL / WARNING / INFO — consistent across all skills
- **Comment pattern**: ❌ BAD / ✅ GOOD for code examples
- **WP version**: Target WordPress 6.x+ APIs and patterns
- **Skill depth**: Each skill matches wp-performance-review quality (~500+ line SKILL.md, 3-5 reference docs, 2 slash commands)

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Code-level security only (no server hardening) | Claude reviews code, not infrastructure; keeps skill focused | — Pending |
| Full-stack Gutenberg coverage (PHP + React/JSX) | WP 6.x+ block development requires both server and client patterns | — Pending |
| Same slash command pattern per skill (full + quick) | Consistent UX, proven pattern from performance skill | — Pending |
| All 5 skills at same depth | Consistent quality across the plugin, no half-built skills | — Pending |
| Include WooCommerce skill | Full coverage: extensions, theming, performance, security for WooCommerce | — Pending |
| WP 6.x+ target only | Modern APIs (block.json, theme.json, Interactivity API) without legacy compat burden | — Pending |

---
*Last updated: 2026-02-06 after adding WooCommerce skill to scope*
