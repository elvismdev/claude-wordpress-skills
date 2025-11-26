# Claude WordPress Skills

Professional WordPress engineering skills for [Claude Code](https://claude.ai/code) - performance optimization, security auditing, Gutenberg block development, and theme/plugin best practices.

## Available Skills

| Skill | Description | Status |
|-------|-------------|--------|
| **wp-performance-review** | Performance code review and optimization analysis | ✅ |
| **wp-security-review** | Security audit and hardening code review | 🚧 |
| **wp-gutenberg-blocks** | Block Editor / Gutenberg development | 🚧 |
| **wp-theme-development** | Theme development best practices | 🚧 |
| **wp-plugin-development** | Plugin architecture and standards | 🚧 |

## Installation

### Option 1: Add as Marketplace

Subscribe to receive all skills and updates (Recommended):

```bash
# In Claude Code CLI
/plugin marketplace add elvismdev/claude-wordpress-skills

# Install specific skills
/plugin install wp-performance-review@claude-wordpress-skills
```

### Option 2: Clone for Personal Use

```bash
# Clone to your personal skills directory
git clone https://github.com/elvismdev/claude-wordpress-skills.git ~/.claude/skills/wordpress
```

### Option 3: Add to Project

Add as a git submodule for team-wide access:

```bash
# In your project root
git submodule add https://github.com/elvismdev/claude-wordpress-skills.git .claude/skills/wordpress
git commit -m "Add WordPress Claude skills"
```

Team members get the skills automatically when they clone or update the repo.

### Option 4: Copy Individual Skills

Download and extract specific skills:

```bash
# Copy just the performance review skill
cp -r skills/wp-performance-review ~/.claude/skills/
```

## Slash Commands

When installed as a plugin, these commands become available:

| Command | Description |
|---------|-------------|
| `/wp-perf-review [path]` | Full WordPress performance code review with detailed analysis and fixes |
| `/wp-perf [path]` | Quick triage scan using grep patterns (fast, critical issues only) |

### Usage Examples

```bash
# Full review of current directory
/wp-perf-review

# Full review of specific plugin
/wp-perf-review wp-content/plugins/my-plugin

# Quick scan of a theme (fast triage)
/wp-perf wp-content/themes/my-theme

# Quick scan to check for critical issues before deploy
/wp-perf .
```

### Command Differences

| Aspect | `/wp-perf-review` | `/wp-perf` |
|--------|-------------------|------------|
| **Speed** | Thorough (slower) | Fast triage |
| **Depth** | Full analysis + fixes | Critical patterns only |
| **Output** | Grouped by severity with line numbers | Quick list of matches |
| **References** | Loads anti-patterns.md, wp-query-guide.md as needed | Minimal |
| **Use case** | Code review, PR review, optimization | Pre-deploy check, quick audit |

When installed via marketplace, commands are namespaced:

```bash
/claude-wordpress-skills:wp-perf-review [path]
/claude-wordpress-skills:wp-perf [path]
```

## Natural Language Usage

Skills also activate automatically based on context. Just ask naturally:

```
Review this plugin for performance issues
Audit this theme for scalability problems  
Check this code for slow database queries
Help me optimize this WP_Query
```

Claude will detect the context and apply the appropriate skill.

### Trigger Phrases

| Skill | Trigger Phrases |
|-------|-----------------|
| wp-performance-review | "performance review", "optimization audit", "slow WordPress", "scale WordPress", "high-traffic" |

## What's Included

### wp-performance-review

Comprehensive performance code review covering:

- **Database Query Anti-Patterns** - Unbounded queries, missing WHERE clauses, slow LIKE patterns, NOT IN performance
- **Hooks & Actions** - Expensive code on init, database writes on page load, inefficient hook placement
- **Caching Issues** - Uncached function calls, object cache patterns, transient best practices
- **AJAX & External Requests** - admin-ajax.php alternatives, polling patterns, HTTP timeouts
- **Template Performance** - N+1 queries, get_template_part optimization
- **PHP Code Patterns** - in_array() performance, heredoc escaping issues
- **JavaScript Bundles** - Full library imports, defer/async strategies
- **Block Editor** - registerBlockStyle overhead, InnerBlocks handling
- **Platform Guidance** - Patterns for WordPress VIP, WP Engine, Pantheon, self-hosted

Output includes severity levels (Critical/Warning/Info) with line numbers and fix recommendations.

## Requirements

- [Claude Code](https://claude.ai/code) CLI installed
- Skills are loaded automatically - no additional dependencies

## Contributing

🤝 Super welcome to contributions, please! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

Ways to contribute:

- 🐛 Report issues or incorrect/deprecated advice
- 💡 Suggest new anti-patterns or best practices
- 📝 Improve documentation or examples
- 🔧 Submit new skills

## License

MIT License — see [LICENSE](LICENSE) for details.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

---

**Note**: These skills represent community best practices for WordPress development and are not affiliated with or endorsed by any specific company or platform. Some patterns reflect my own experience with WordPress and from years of working alongside engineers far smarter than me - so bias is inevitable. Contributions are always welcome; I'm genuinely curious to hear different approaches and learn together.
