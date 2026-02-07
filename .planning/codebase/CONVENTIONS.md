# Coding Conventions

**Analysis Date:** 2026-02-06

## Overview

This codebase documents WordPress performance review practices and skill definitions for Claude Code. Content is written primarily in Markdown with YAML frontmatter for plugin configuration. There is no compiled code (no Python, JavaScript, TypeScript, or traditional programming languages). Instead, conventions focus on documentation structure, code examples, and knowledge representation.

## Naming Patterns

**Files:**
- Skill definition files: `SKILL.md` (located in `skills/skill-name/SKILL.md`)
- Command files: descriptive with hyphens (e.g., `wp-perf-review.md`, `wp-perf.md`)
- Reference guides: descriptive nouns with hyphens (e.g., `anti-patterns.md`, `caching-guide.md`, `wp-query-guide.md`)
- Configuration files: lowercase with dots (`plugin.json`, `marketplace.json`)
- Documentation files: UPPERCASE.md (`README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `CLAUDE.md`)

**Sections within documentation:**
- Markdown headers use title case with descriptive names
- Code example comments use clear intent (e.g., "❌ BAD", "✅ GOOD", "CRITICAL", "WARNING", "INFO")

**PHP Code Examples:**
- Functions: `snake_case` with prefix (e.g., `prefix_function_name()`)
- Variables: `$snake_case` (e.g., `$post_id`, `$query`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `HOUR_IN_SECONDS`, `DISABLE_WP_CRON`)
- WordPress functions: Exact WordPress naming (e.g., `get_posts()`, `wp_cache_get()`, `add_action()`)

## Code Style

**PHP Code Examples:**
All PHP code examples follow [WordPress PHP Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/):

- Spaces inside parentheses: `function_name( $arg )` not `function_name($arg)`
- Array syntax: Use `array()` not `[]`
- Yoda conditions: `if ( true === $value )` not `if ( $value === true )`
- Snake_case for variables and functions
- Prefix all custom functions to avoid collisions: `prefix_function_name()`
- Indentation: 4 spaces
- Opening braces on same line: `function() {`

**Markdown Formatting:**
- Use Markdown tables for comparisons and quick reference
- Use code blocks with language specification (e.g., ```php, ```javascript, ```bash)
- Use inline backticks for file paths: `` `src/services/user.ts` ``
- Use bold for emphasis: `**CRITICAL**`, `**WARNING**`

## Import Organization

Not applicable - this codebase contains documentation and skill definitions, not traditional imports. However, skill content organization follows this pattern:

**Content Structure within Skills:**
1. **Frontmatter** - YAML metadata (name, description)
2. **Overview section** - High-level introduction
3. **When to Use** - Use cases and boundaries
4. **Main content** - Organized by file type or concern
5. **Quick reference** - Tables and patterns for scanning
6. **Code examples** - BAD/GOOD comparisons
7. **References** - Links to deeper documentation

**Reference Loading:**
- Skills link to supporting documentation via mentions like "Load `references/anti-patterns.md`"
- Deep references are kept separate in `references/` subdirectories
- Cross-references between skills use relative file paths

## Documentation Structure

**Skill.md Files** (`skills/wp-performance-review/SKILL.md`):
```yaml
---
name: skill-name
description: What this skill does and trigger phrases. Max 1024 chars.
---

# Title

Introduction paragraph.

## When to Use
- Use when...
- Don't use for...

## Code Review Workflow
1. Step one
2. Step two

## File-Type Specific Checks
### PHP Files
- Pattern to check
- What it means

## Quick Reference
- Quick lookup table
- Common patterns

## Deep-Dive References
- Links to references/
```

**Command Files** (`commands/wp-perf-review.md`):
```yaml
---
description: What the command does
argument-hint: [optional-args]
---

Instructions for Claude on how to execute.
```

**Reference Files** (`skills/wp-performance-review/references/anti-patterns.md`):
- Quick lookup tables
- Detailed explanations
- Code examples organized by category
- "BAD/GOOD" comparison patterns

## Severity Labels

**Consistent usage across all content:**
- `**CRITICAL**` - Will cause failures at scale (OOM, 500 errors, DB locks, cache bypass)
- `**WARNING**` - Degrades performance under load or causes inefficiency
- `**INFO**` - Optimization opportunity, nice-to-have improvement

Used in:
- Inline descriptions: "**CRITICAL**: Unbounded query"
- Severity tables: Listed with pattern and impact
- Comments in code: `// CRITICAL: Never use - breaks main query`

## Comments and Examples

**Comment Style:**
```php
// ❌ BAD: Short description of the problem.
bad_code_example();

// ✅ GOOD: Short description of the solution.
good_code_example();
```

**Example Organization:**
- Always show the problem first (❌ BAD)
- Then show the solution (✅ GOOD)
- Include explanatory comment inline or above code
- For multiple good patterns, show all with ✅

**JSDoc/Comment Patterns:**
- Used for inline documentation in PHP examples
- Comments explain WHY, not just WHAT
- Severity badges included for quick scanning: `// CRITICAL:`, `// WARNING:`, `// INFO:`

## Content Standards

**Code Examples:**
- Every anti-pattern example has a corresponding GOOD example
- Real WordPress functions and hooks used (no pseudocode)
- Examples are complete and runnable (not fragments)
- Output shows realistic scenarios

**References:**
- Organized by concern (database, hooks, caching, etc.)
- Include severity, trigger patterns, and fixes
- Show context (when this pattern appears, what files to check)

**Markdown Tables:**
- Used for quick lookup (pattern → severity → issue)
- Used for comparison (aspect → `/wp-perf-review` → `/wp-perf`)
- Tables include all necessary context in columns

## File Organization

**Root level:**
- `README.md` - Project overview and installation
- `CONTRIBUTING.md` - Contribution guidelines and standards
- `CLAUDE.md` - Claude Code specific guidance (versioning, structure)
- `CHANGELOG.md` - Version history with changes

**`.claude-plugin/`:**
- `plugin.json` - Package metadata (name, version, author, homepage)
- `marketplace.json` - Marketplace registration with plugin list

**`skills/`:**
- Organized as `skills/{skill-name}/`
- Each skill has `SKILL.md` (main definition)
- Optional `references/` subdirectory for supporting docs

**`commands/`:**
- Organized as `commands/{command-name}.md`
- Each command is standalone
- Commands reference and delegate to skills

## Whitespace and Formatting

**Markdown:**
- Blank lines between sections (headers, paragraphs, lists)
- 2 spaces for nested list items
- Code blocks enclosed in triple backticks with language
- Line length: No hard limit, but readability preferred

**JSON:**
- 2-space indentation
- Trailing commas: No
- Consistent key ordering: alphabetical or logical grouping

## No Active Code Files

**Important:** This codebase contains NO source code in traditional languages:
- No `.js`, `.ts`, `.py`, `.php` source files (only examples in docs)
- No compiled/executable code
- No dependencies to manage (no `package.json` with `dependencies`)
- No unit tests to run
- Content is documentation, configuration, and knowledge representation only

**Testing approach:** Skills are tested by copying to Claude's skill directory and restarting Claude Code. No automated test framework.

---

*Convention analysis: 2026-02-06*
