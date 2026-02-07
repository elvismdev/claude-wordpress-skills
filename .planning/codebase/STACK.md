# Technology Stack

**Analysis Date:** 2026-02-06

## Languages

**Primary:**
- Markdown - Skill definitions, documentation, and command specifications
- YAML - Plugin metadata and command configuration in frontmatter

**Secondary:**
- PHP - Code examples in anti-patterns documentation
- JavaScript - Code examples in performance patterns documentation
- Bash - Code examples in search patterns and reference documentation

## Runtime

**Environment:**
- Claude Code CLI - Required runtime for skill loading and execution

**Platform:**
- Platform-independent (documented for WordPress VIP, WP Engine, Pantheon, and self-hosted environments)

## Frameworks

**Core:**
- Claude Code Plugin Framework - Plugin structure defined in `.claude-plugin/`

**Skills Framework:**
- No explicit skill framework library; uses markdown-based YAML frontmatter specification
- Trigger phrase matching system for automatic skill activation

**Testing:**
- Not applicable - Plugin loaded by Claude Code runtime

**Build/Dev:**
- None required; Markdown and JSON are plain text formats

## Key Dependencies

**Critical:**
- Claude Code CLI - Runtime platform that loads and executes skills
- No external package dependencies

**Infrastructure:**
- GitHub - Repository hosting and distribution
- Claude Code Marketplace - Optional distribution channel for plugin registration

## Configuration

**Environment:**
- Version managed in plugin metadata files
- No environment variables or secrets required for skill definitions

**Build:**
- `.claude-plugin/plugin.json` - Plugin metadata (name, version, author, homepage)
- `.claude-plugin/marketplace.json` - Marketplace registration metadata

## Platform Requirements

**Development:**
- Git for version control
- Text editor for markdown files
- No runtime or package manager required

**Production:**
- Claude Code CLI installed on user's system
- Git (for cloning repository) or marketplace client (for plugin installation)
- Access to target WordPress codebase for skill to analyze

## Versioning

**Current Version:** 1.3.1

**Version Management:**
- Versions synchronized across `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
- Semantic versioning used (major.minor.patch)
- Changelog maintained in `CHANGELOG.md`

---

*Stack analysis: 2026-02-06*
