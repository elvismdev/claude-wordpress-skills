# Technology Stack: WordPress Claude Skills Plugin

**Project:** WordPress Claude Code Skills Build-Out
**Researched:** 2026-02-06

## Recommended Stack

### Core Framework

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Markdown | CommonMark | Skill definition format | Human-readable, version-controllable, Claude-native format |
| YAML | 1.2 | Frontmatter metadata | Standard for skill/command metadata in Claude Code ecosystem |
| JSON | - | Plugin configuration | Required by Claude Code plugin system (.claude-plugin/) |

**Rationale:** Documentation-driven architecture. No runtime code means no traditional framework dependencies. Markdown is the native format for Claude Code skills, YAML frontmatter is the standard metadata format, and JSON is required for plugin registration.

### Documentation Standards

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| WordPress PHP Coding Standards | Latest | PHP code examples | Official WordPress standard, required for theme/plugin directory |
| WordPress JavaScript Standards | Latest | JS/JSX code examples | Official standard for block development |
| CommonMark | 0.30+ | Markdown syntax | Consistent rendering across tools |

**Rationale:** All code examples must follow official WordPress coding standards. This is non-negotiable for a WordPress-focused skill plugin. Users expect examples they can copy directly into WordPress projects.

### WordPress Target Versions

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| WordPress Core | 6.6+ | Target platform | Current stable version (Feb 2026), includes theme.json v3, Interactivity API refinements |
| PHP | 8.0+ | Examples assume | WordPress 6.6 requires PHP 7.4+, but modern patterns assume 8.0+ |
| Block Editor (Gutenberg) | Bundled with WP 6.6 | Block development patterns | Plugin version ahead of core, but target core bundled version |

**Rationale:** WordPress 6.6 is current stable. Theme.json v3 is living spec as of 6.6. Interactivity API introduced in 6.5, refined in 6.6. Skills should target current stable + one version back (6.5+) for broader applicability.

## Installation

No installation required for development—plugin is pure documentation.

### For Users (Plugin Installation)

```bash
# Option 1: Marketplace (recommended)
/plugin marketplace add elvismdev/claude-wordpress-skills
/plugin install claude-wordpress-skills@claude-wordpress-skills

# Option 2: Local clone
git clone https://github.com/elvismdev/claude-wordpress-skills.git ~/.claude/plugins/wordpress
```

## Validation Checklist

Before committing skill content:

- [ ] All YAML frontmatter description fields under 1024 chars
- [ ] All YAML frontmatter name fields under 64 chars
- [ ] PHP examples follow WordPress PHP Coding Standards
- [ ] Severity labels consistent: CRITICAL / WARNING / INFO
- [ ] Grep patterns tested against real WordPress codebases

## Sources

- [WordPress PHP Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/)
- [WordPress JavaScript Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/javascript/)
- [Theme.json Version 3 Reference](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/)

---

*Stack research completed: 2026-02-06*
