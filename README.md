# Claude WordPress Skills

Professional WordPress engineering skills for [Claude Code](https://claude.ai/code) - performance optimization, security auditing, Gutenberg block development, theme/plugin development, and WooCommerce extensions.

## Available Skills

| Skill | Description | Status |
|-------|-------------|--------|
| **wp-performance-review** | Performance code review and optimization analysis | ✅ |
| **wp-security-review** | Security audit and vulnerability detection code review | ✅ |
| **wp-plugin-development** | Plugin architecture and WordPress.org standards review | ✅ |
| **wp-block-development** | Gutenberg block development and Block API patterns review | ✅ |
| **wp-theme-development** | Block theme development and FSE patterns review | ✅ |
| **wp-woocommerce-dev** | WooCommerce extension development and HPOS compatibility review | ✅ |

## Installation

### Option 1: Add as Marketplace

Subscribe to receive all skills and updates (Recommended):

```bash
# In Claude Code CLI
/plugin marketplace add elvismdev/claude-wordpress-skills

# Install specific skills
/plugin install claude-wordpress-skills@claude-wordpress-skills
```

### Option 2: Clone Locally

```bash
git clone https://github.com/elvismdev/claude-wordpress-skills.git ~/.claude/plugins/wordpress
```

### Option 3: Add to Project

Add as a git submodule for team-wide access:

```bash
# In your project root
git submodule add https://github.com/elvismdev/claude-wordpress-skills.git .claude/plugins/wordpress
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

When installed, these commands become available:

| Command | Description |
|---------|-------------|
| `/wp-perf-review [path]` | Full WordPress performance code review with detailed analysis and fixes |
| `/wp-perf [path]` | Quick triage scan using grep patterns (fast, critical issues only) |
| `/wp-sec-review [path]` | Full security audit for XSS, SQL injection, CSRF, authorization bypass |
| `/wp-sec [path]` | Quick security scan for common vulnerability patterns |
| `/wp-plugin-review [path]` | Full plugin architecture review for WordPress.org standards |
| `/wp-plugin [path]` | Quick plugin scan for common architecture issues |
| `/wp-block-review [path]` | Full Gutenberg block development review (PHP + React/JSX) |
| `/wp-block [path]` | Quick block scan for block.json and API issues |
| `/wp-theme-review [path]` | Full theme development review (block themes + theme.json v3) |
| `/wp-theme [path]` | Quick theme scan for FSE and template issues |
| `/wp-woo-review [path]` | Full WooCommerce extension review (HPOS, payment gateways, cart fragments) |
| `/wp-woo [path]` | Quick WooCommerce scan for HPOS compatibility and performance issues |

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

### Command Comparison

| Aspect | `/wp-perf-review` | `/wp-perf` |
|--------|-------------------|------------|
| **Speed** | Thorough (slower) | Fast triage |
| **Depth** | Full analysis + fixes | Critical patterns only |
| **Output** | Grouped by severity with line numbers | Quick list of matches |
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
Check my theme before launch
Find anti-patterns in this plugin
```

Claude will detect the context and apply the appropriate skill.

### Trigger Phrases

| Skill | Trigger Phrases |
|-------|-----------------|
| wp-performance-review | "performance review", "optimization audit", "slow WordPress", "slow queries", "scale WordPress", "high-traffic", "code review", "before launch", "anti-patterns", "timeout", "500 error", "out of memory" |
| wp-security-review | "security review", "security audit", "vulnerability scan", "XSS", "SQL injection", "CSRF", "authorization bypass", "nonce verification", "sanitize", "escape output" |
| wp-plugin-development | "plugin review", "plugin architecture", "WordPress.org standards", "plugin structure", "activation hooks", "custom post types", "Settings API" |
| wp-block-development | "block review", "Gutenberg development", "block.json", "registerBlockType", "InnerBlocks", "Interactivity API", "block patterns", "dynamic blocks" |
| wp-theme-development | "theme review", "block theme", "theme.json", "FSE", "Full Site Editing", "template parts", "global styles", "style variations" |
| wp-woocommerce-dev | "WooCommerce review", "HPOS compatibility", "payment gateway", "WooCommerce extension", "cart fragments", "WC_Order", "Action Scheduler", "WooCommerce templates" |

## What's Included

All six skills provide comprehensive code review with severity levels (Critical/Warning/Info), line numbers, and actionable fix recommendations.

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

### wp-security-review

WordPress-specific security code review covering:

- **XSS Prevention** - Late escaping at output, esc_html/esc_attr/esc_url/esc_js patterns, kses usage
- **SQL Injection** - $wpdb->prepare() validation, direct query detection, user input in queries
- **CSRF Protection** - Nonce verification, wp_verify_nonce patterns, form protection
- **Authorization** - current_user_can() checks, capability validation, privilege escalation
- **File Upload Security** - MIME type validation, file extension checks, path traversal prevention
- **Dangerous Functions** - eval(), create_function(), unserialize(), extract() detection
- **Sensitive Data** - Raw card data storage, API keys in code, password logging

### wp-plugin-development

Plugin architecture and WordPress.org standards review covering:

- **Plugin Structure** - Header validation, file organization, namespace/prefix patterns
- **Lifecycle Hooks** - Activation/deactivation/uninstall patterns, proper cleanup
- **Custom Post Types** - register_post_type validation, capability mapping, REST API support
- **Settings API** - add_settings_section/field patterns, sanitization callbacks
- **Hook Usage** - Action/filter placement, priority conflicts, performance impact
- **Internationalization** - Text domain usage, load_plugin_textdomain(), translation-ready strings
- **WordPress.org Compliance** - Plugin Check requirements, directory structure, readme.txt validation

### wp-block-development

Gutenberg block development review (PHP + React/JSX) covering:

- **block.json Schema** - apiVersion, title, category, attributes, supports validation
- **Editor Components** - useBlockProps, InnerBlocks, RichText, MediaUpload patterns
- **Server-Side Rendering** - render_callback, dynamic blocks, $attributes access
- **Block Deprecations** - deprecated array structure, migration functions, version handling
- **Interactivity API** - WP 6.5+ frontend patterns, data-wp-* directives, store/actions
- **Build Process** - Source vs build file review, @wordpress/scripts setup, asset generation
- **Performance** - registerBlockStyle overhead, block_category_filter usage

### wp-theme-development

Block theme development and Full Site Editing review covering:

- **theme.json v3** - settings, styles, templateParts, customTemplates validation
- **Template Hierarchy** - .html template files, template parts, fallback chain
- **Global Styles** - Typography, color palettes, spacing scales, layout settings
- **Style Variations** - JSON structure, inheritance patterns, user customization
- **Block Patterns** - Pattern registration, block markup, category organization
- **Classic Theme Migration** - Hybrid theme patterns, gradual FSE adoption, backwards compatibility

### wp-woocommerce-dev

WooCommerce extension development review covering:

- **HPOS Compatibility** - declare_compatibility(), WC_Order vs WP_Post patterns, custom table queries
- **Payment Gateway Security** - Raw card data anti-patterns, webhook verification, HTTPS enforcement
- **WooCommerce CRUD** - WC_Product, WC_Order, WC_Customer object usage vs direct database access
- **Hook Preservation** - Template override validation, do_action/apply_filters in templates
- **Cart Fragments Performance** - woocommerce_add_to_cart_fragments usage, Mini-Cart Block alternative
- **Action Scheduler** - as_schedule_single_action patterns, long-running task handling
- **WC Blocks Integration** - Block-based checkout/cart, StoreAPI extensions

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
