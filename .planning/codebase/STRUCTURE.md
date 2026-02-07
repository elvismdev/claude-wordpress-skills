# Codebase Structure

**Analysis Date:** 2026-02-06

## Directory Layout

```
claude-wordpress-skills/
├── .claude-plugin/           # Plugin configuration and metadata
│   ├── plugin.json           # Plugin manifest with name, version, author
│   └── marketplace.json      # Marketplace registration and plugin list
├── skills/                   # Skill definitions (core expertise modules)
│   └── wp-performance-review/
│       ├── SKILL.md          # Main skill file with trigger phrases and workflows
│       └── references/       # Supporting knowledge bases
│           ├── anti-patterns.md
│           ├── caching-guide.md
│           ├── wp-query-guide.md
│           └── measurement-guide.md
├── commands/                 # Slash command definitions
│   ├── wp-perf-review.md     # Full performance review command
│   └── wp-perf.md            # Quick triage scan command
├── README.md                 # User-facing documentation with installation and usage
├── CHANGELOG.md              # Version history and feature changes
├── CONTRIBUTING.md           # Contribution guidelines
├── LICENSE                   # MIT license
├── CLAUDE.md                 # Developer guidance for Claude Code
└── .gitignore               # Standard git ignore patterns
```

## Directory Purposes

**.claude-plugin:**
- Purpose: Configuration directory for Claude Code plugin system
- Contains: JSON metadata files for plugin registration and marketplace
- Key files: `plugin.json` (plugin definition), `marketplace.json` (distribution config)

**skills/**
- Purpose: Root directory for all skill definitions
- Contains: Subdirectories for each skill (currently only wp-performance-review)
- Naming: `skills/wp-{skill-name}/` following kebab-case pattern

**skills/wp-performance-review/**
- Purpose: WordPress performance review expertise module
- Contains: Main skill definition and reference knowledge bases
- Key files: `SKILL.md` (skill logic), `references/*` (context)

**skills/wp-performance-review/references/**
- Purpose: Deep knowledge bases for pattern matching and context
- Contains: Markdown files with pattern catalogs, code examples, optimization guides
- Key files:
  - `anti-patterns.md`: Quick lookup table of bad patterns with severity + code examples
  - `caching-guide.md`: Caching strategies for WordPress
  - `wp-query-guide.md`: WP_Query optimization reference
  - `measurement-guide.md`: Performance measurement approaches

**commands/**
- Purpose: Slash command entry points
- Contains: Markdown files that define commands and their behavior
- Naming: `{command-name}.md` matching slash command format (e.g., `wp-perf-review.md` for `/wp-perf-review`)

## Key File Locations

**Entry Points:**
- `skills/wp-performance-review/SKILL.md`: Primary skill triggered by natural language or referenced by commands
- `commands/wp-perf-review.md`: Full review command, invokes complete Code Review Workflow
- `commands/wp-perf.md`: Quick scan command, invokes grep pattern search subset

**Configuration:**
- `.claude-plugin/plugin.json`: Plugin manifest (name, version, author, keywords)
- `.claude-plugin/marketplace.json`: Marketplace registration (owner, version, plugin list)
- `CLAUDE.md`: Developer instructions for Claude Code (repository structure, code standards, versioning)

**Core Logic:**
- `skills/wp-performance-review/SKILL.md`: All code review logic, file-type checks, search patterns, output format
- `skills/wp-performance-review/references/anti-patterns.md`: Pattern detection rules, code examples
- `skills/wp-performance-review/references/caching-guide.md`: Caching best practices
- `skills/wp-performance-review/references/wp-query-guide.md`: Database query optimization
- `skills/wp-performance-review/references/measurement-guide.md`: Performance measurement approaches

**Documentation:**
- `README.md`: User installation, usage, command reference, trigger phrases
- `CHANGELOG.md`: Version history
- `CONTRIBUTING.md`: Contribution process
- `LICENSE`: MIT license text

## Naming Conventions

**Files:**
- Skill definitions: `SKILL.md` (required, all caps)
- Reference files: `{topic}-guide.md` or `{topic}.md` (kebab-case topics: anti-patterns, caching-guide)
- Command files: `{command-name}.md` matching slash command name (kebab-case: wp-perf-review.md)
- Config files: `plugin.json`, `marketplace.json`, `CLAUDE.md` (all lowercase json, all caps markdown)

**Directories:**
- Skill root: `skills/wp-{skill-name}/` (kebab-case, wp- prefix)
- Reference subdirectory: Always `references/` within skill directory
- Command directory: Always `commands/` at root level

**YAML Frontmatter:**
- Skill definition: `name` (kebab-case), `description` (max 1024 chars with trigger phrases)
- Command definition: `description`, `argument-hint` (optional args)

## Where to Add New Code

**New Skill:**
- Create directory: `skills/wp-{skill-name}/`
- Create main file: `skills/wp-{skill-name}/SKILL.md` with YAML frontmatter
  ```yaml
  ---
  name: skill-name
  description: Brief description with trigger phrases. Max 1024 chars.
  ---
  ```
- Create knowledge bases: `skills/wp-{skill-name}/references/*.md` as needed
- Update: `.claude-plugin/marketplace.json` to register new skill in plugins array
- Update: `README.md` to add entry to Available Skills table with status indicator

**New Slash Command:**
- Create file: `commands/{command-name}.md` with YAML frontmatter
  ```yaml
  ---
  description: What the command does
  argument-hint: [optional-args]
  ---
  ```
- Reference skill in content: "Use and follow the **{skill-name}** skill..."
- Use `$ARGUMENTS` placeholder to reference user-provided path
- Document in `README.md` Slash Commands section

**Enhanced Reference Knowledge:**
- Add to existing reference file: `skills/wp-performance-review/references/{guide}.md`
- Follow existing pattern: Markdown table of patterns + detailed explanations with code examples
- Use "❌ BAD" and "✅ GOOD" comment markers for visual distinction
- Include severity labels: CRITICAL, WARNING, INFO
- Add PHP code examples following WordPress PHP Coding Standards (from CLAUDE.md)

## Special Directories

**.planning/codebase/**
- Purpose: Generated documentation directory (not committed to main codebase)
- Contains: Mapping documents created by Claude Code analysis (ARCHITECTURE.md, STRUCTURE.md, etc.)
- Generated: Yes (by /gsd:map-codebase command)
- Committed: No (local analysis only)

**.git/**
- Purpose: Git repository metadata
- Committed: Yes (repository tracking)

## File Locations for Future Maintenance

**To add skill:**
1. Create directory structure: `skills/wp-{skill-name}/references/`
2. Primary file: `skills/wp-{skill-name}/SKILL.md`
3. Register in: `.claude-plugin/marketplace.json` (plugins array)
4. Document in: `README.md` (Available Skills table)

**To add command:**
1. File: `commands/{cmd-name}.md`
2. Document in: `README.md` (Slash Commands table)

**To update version:**
1. `.claude-plugin/plugin.json` (version field)
2. `.claude-plugin/marketplace.json` (metadata.version field)
3. Update: `CHANGELOG.md` with changes

**To add patterns/guides:**
1. Edit existing: `skills/wp-performance-review/references/*.md`
2. Or create new reference: `skills/wp-performance-review/references/{new-guide}.md`

---

*Structure analysis: 2026-02-06*
