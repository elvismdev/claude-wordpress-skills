# Architecture

**Analysis Date:** 2026-02-06

## Pattern Overview

**Overall:** Plugin-based skill and command delivery system for Claude Code

**Key Characteristics:**
- Modular plugin architecture with separate skill and command definitions
- Skill-driven pattern matching and code analysis
- Multi-layer abstraction: skills provide knowledge, commands provide entry points, references provide deep context
- Marketplace-enabled distribution model with support for local/submodule installation

## Layers

**Plugin Layer:**
- Purpose: Expose the plugin to Claude Code and marketplace systems
- Location: `.claude-plugin/`
- Contains: Configuration metadata, versioning, plugin registration
- Depends on: None (entry point)
- Used by: Claude Code CLI, marketplace system

**Skill Layer:**
- Purpose: Define code review expertise, patterns, and automation logic
- Location: `skills/wp-performance-review/`
- Contains: SKILL.md with YAML frontmatter, trigger phrases, detection workflows
- Depends on: Reference files for context
- Used by: Claude Code natural language processing, slash commands

**Command Layer:**
- Purpose: Expose slash command entry points to Claude Code users
- Location: `commands/`
- Contains: Markdown files with command metadata and invocation instructions
- Depends on: Skills (commands reference and execute skills)
- Used by: Claude Code slash command system

**Reference Knowledge Layer:**
- Purpose: Provide detailed, searchable knowledge bases for pattern detection and code examples
- Location: `skills/wp-performance-review/references/`
- Contains: Pattern catalogs, optimization guides, anti-pattern examples with PHP/JS code samples
- Depends on: None (leaf nodes)
- Used by: Skills for deep analysis, Claude for context when executing code reviews

## Data Flow

**Skill Activation Flow:**

1. User provides natural language request (e.g., "review this plugin for performance issues")
2. Claude Code analyzes request against trigger phrases in SKILL.md (line 3: "performance review", "slow WordPress", "optimization audit", etc.)
3. Skill triggers and provides code review workflow context
4. Claude applies file-type-specific checks (`plugin.php`, templates, JavaScript, etc.) from SKILL.md
5. Reference files (anti-patterns.md, caching-guide.md, etc.) loaded for pattern context
6. Code review output formatted with severity levels (CRITICAL/WARNING/INFO)

**Command Execution Flow:**

1. User invokes slash command (`/wp-perf-review [path]` or `/wp-perf [path]`)
2. Command metadata from markdown YAML frontmatter provides description and argument handling
3. Command file references skill name (e.g., "Use and follow the **wp-performance-review** skill")
4. For `/wp-perf-review`: Full Code Review Workflow from SKILL.md executed (lines 26-32)
5. For `/wp-perf`: Abbreviated Quick Detection pattern subset executed (lines 98-137 grep patterns)
6. Results reported with file:line references and severity levels

**State Management:**
- Stateless: No persistent state between invocations
- Configuration state: Plugin version, skill name, command arguments passed via YAML frontmatter
- Knowledge state: Pattern catalogs in reference files used as context during analysis

## Key Abstractions

**Skill Definition:**
- Purpose: Encapsulate WordPress code review expertise as reusable pattern
- Examples: `skills/wp-performance-review/SKILL.md`
- Pattern: YAML frontmatter (name, description, trigger phrases) + markdown content with structured sections (When to Use, Code Review Workflow, File-Type Specific Checks, Search Patterns)

**Command Definition:**
- Purpose: Create user-facing entry point with optional arguments
- Examples: `commands/wp-perf-review.md`, `commands/wp-perf.md`
- Pattern: YAML frontmatter (description, argument-hint) + instructions to invoke skill with specific approach

**Reference Knowledge Base:**
- Purpose: Provide searchable, categorized pattern catalogs and examples
- Examples: `skills/wp-performance-review/references/anti-patterns.md` (quick lookup table + deep dives), `caching-guide.md`, `wp-query-guide.md`
- Pattern: Markdown tables for quick lookup + code examples with comments explaining bad/good patterns

**Pattern Detection:**
- Purpose: Identify code anti-patterns through grep-based search or manual code analysis
- Pattern: Structured search patterns (lines 98-137 in SKILL.md) organized by severity and file type
- Applied to: PHP files, JavaScript, block.json, template files

## Entry Points

**Natural Language (Automatic Skill Trigger):**
- Location: Handled by Claude Code LLM via trigger phrases in SKILL.md line 3
- Triggers: User mentions "performance review", "slow WordPress", "500 error", "timeout", etc.
- Responsibilities: Activate skill, run full code review workflow, provide detailed analysis

**Slash Command: `/wp-perf-review [path]`:**
- Location: `commands/wp-perf-review.md`
- Triggers: User types `/wp-perf-review` followed by optional file/directory path
- Responsibilities: Run full Code Review Workflow from SKILL.md with all checks, provide line numbers and severity levels, suggest fixes

**Slash Command: `/wp-perf [path]`:**
- Location: `commands/wp-perf.md`
- Triggers: User types `/wp-perf` followed by optional file/directory path
- Responsibilities: Fast triage using only grep patterns from lines 101-137, report critical patterns only, suggest comprehensive review if issues found

## Error Handling

**Strategy:** Documentation-based pattern guidance rather than runtime error handling

**Patterns:**
- No error handling code (plugin is documentation/knowledge, not executable)
- Patterns documented in reference files with "bad/good" examples showing consequences
- Anti-patterns catalog (`anti-patterns.md`) explains impact of each issue (e.g., "CRITICAL: Unbounded query", "WARNING: Full table scan")
- Search patterns (grep commands) designed to avoid false positives by excluding non-critical contexts (e.g., grep filters to exclude admin paths)

## Cross-Cutting Concerns

**Logging:** Not applicable (plugin is documentation-driven)

**Validation:** Entry point validation through YAML frontmatter (command description, argument-hint), skill trigger phrase matching

**Authentication:** Not applicable (plugin accessed through Claude Code, which handles auth)

**Code Style:** WordPress PHP Coding Standards documented in CLAUDE.md (lines 54-61): spaces inside parentheses, array() not [], Yoda conditions, snake_case, prefix functions

**Versioning:** Synchronized across `.claude-plugin/plugin.json` (version field) and `.claude-plugin/marketplace.json` (metadata.version field)

**Knowledge Maintenance:** Skill descriptions capped at 1024 characters (CLAUDE.md line 37), trigger phrases maintained in SKILL.md frontmatter, patterns organized by severity (CRITICAL/WARNING/INFO) for triage order

---

*Architecture analysis: 2026-02-06*
