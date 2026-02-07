# Architecture Patterns: WordPress Claude Skills Plugin

**Domain:** Claude Code Skills Plugin for WordPress Development
**Researched:** 2026-02-06

## Executive Summary

This architecture research defines how 4 new skills (security-review, gutenberg-blocks, theme-development, plugin-development) should integrate into the existing plugin structure. The analysis reveals that the existing wp-performance-review skill provides a proven architectural pattern: modular skill definition with YAML frontmatter, structured sections, reference doc hierarchy, and dual command approach (full review + quick scan).

Key architectural decisions:
1. **Maintain skill isolation** - Each skill is self-contained with its own SKILL.md and references
2. **Share patterns via cross-references** - Overlapping topics (e.g., security in theme dev) handled through explicit cross-skill references rather than duplication
3. **Consistent internal structure** - All skills follow the same 12-section SKILL.md template established by wp-performance-review
4. **Reference granularity by topic depth** - Security and Gutenberg need 5-6 references (deeper technical domains), theme/plugin need 3-4 references (more unified patterns)

## Recommended Architecture

### Overall Structure

```
skills/
  wp-performance-review/        ← Existing (template for others)
    SKILL.md (507 lines)
    references/
      anti-patterns.md (1199 lines)
      wp-query-guide.md (328 lines)
      caching-guide.md (447 lines)
      measurement-guide.md (356 lines)

  wp-security-review/           ← NEW
    SKILL.md (~550 lines)
    references/
      vulnerability-patterns.md (~1000 lines)
      escaping-guide.md (~400 lines)
      auth-patterns.md (~350 lines)
      sanitization-guide.md (~400 lines)
      nonce-csrf-guide.md (~300 lines)

  wp-gutenberg-blocks/          ← NEW
    SKILL.md (~600 lines)
    references/
      block-patterns.md (~800 lines)
      block-json-guide.md (~400 lines)
      react-patterns.md (~500 lines)
      attributes-guide.md (~350 lines)
      dynamic-blocks-guide.md (~400 lines)
      interactivity-api-guide.md (~350 lines)

  wp-theme-development/         ← NEW
    SKILL.md (~500 lines)
    references/
      theme-json-guide.md (~600 lines)
      template-patterns.md (~500 lines)
      fse-guide.md (~450 lines)
      classic-to-block-guide.md (~400 lines)

  wp-plugin-development/        ← NEW
    SKILL.md (~550 lines)
    references/
      architecture-patterns.md (~700 lines)
      hooks-guide.md (~500 lines)
      oop-patterns.md (~450 lines)
      api-patterns.md (~400 lines)

commands/
  wp-perf-review.md             ← Existing
  wp-perf.md                    ← Existing
  wp-sec-review.md              ← NEW (full security review)
  wp-sec.md                     ← NEW (quick security scan)
  wp-block-review.md            ← NEW (full block review)
  wp-block.md                   ← NEW (quick block scan)
  wp-theme-review.md            ← NEW (full theme review)
  wp-theme.md                   ← NEW (quick theme scan)
  wp-plugin-review.md           ← NEW (full plugin review)
  wp-plugin.md                  ← NEW (quick plugin scan)
```

### SKILL.md Internal Structure (Template)

All skills follow this 12-section pattern established by wp-performance-review:

```markdown
---
name: wp-[domain]-review
description: [Triggers + key capabilities, max 1024 chars]
---

# [Skill Name] Skill

## Overview
[Core principle, approach, what makes this domain unique]

## When to Use
**Use when:** [5-7 scenarios]
**Don't use for:** [3-4 out-of-scope scenarios with references to other skills]

## Code Review Workflow
[Numbered steps: identify → scan critical → check warnings → note optimizations → report]

## File-Type Specific Checks
[Subsections per file type with severity-tagged patterns]

## Search Patterns for Quick Detection
[Bash/grep commands organized by severity for quick scan command]

## Platform Context
[Hosting environment differences, platform-specific patterns]

## Quick Reference: [Domain] Patterns
[Code examples with ❌/✅ comments organized by category]

## Severity Definitions
| Severity | Description |
|----------|-------------|
| **Critical** | [Domain-specific critical definition] |
| **Warning** | [Domain-specific warning definition] |
| **Info** | [Domain-specific info definition] |

## Output Format
[Markdown template for review results]

## Common Mistakes
[Table: Mistake | Why Wrong | Fix]

## Deep-Dive References
[Table: Task → Reference mapping]
```

### Cross-Skill Reference Strategy

**Problem:** Security concerns appear in theme dev, performance patterns appear in plugin dev.

**Solution:** Explicit cross-references with skill boundaries clearly defined.

#### Overlap Matrix

| Topic | Primary Skill | Also Appears In | Strategy |
|-------|--------------|-----------------|----------|
| **XSS/Escaping** | wp-security-review | wp-theme-development | Theme skill includes brief escaping section + "See wp-security-review for comprehensive escaping guide" |
| **SQL Injection** | wp-security-review | wp-plugin-development | Plugin skill mentions prepared statements + cross-reference to security skill |
| **Database Queries** | wp-performance-review | wp-plugin-development | Plugin skill covers architecture patterns, references performance skill for query optimization |
| **Hooks/Actions** | wp-plugin-development | wp-performance-review | Performance skill shows anti-patterns, plugin skill shows architecture patterns |
| **Nonce Verification** | wp-security-review | wp-gutenberg-blocks, wp-plugin-development | Brief mention in context + cross-reference |
| **Block Development** | wp-gutenberg-blocks | wp-theme-development (patterns), wp-plugin-development (structure) | Theme: pattern registration. Plugin: block registration. Gutenberg: full block code review |
| **REST API** | wp-plugin-development | wp-performance-review (admin-ajax alternative) | Performance: mentions REST vs admin-ajax. Plugin: full REST API patterns |
| **Transients/Options** | wp-performance-review | wp-plugin-development | Performance: caching strategy. Plugin: data persistence architecture |

#### Cross-Reference Pattern (Example)

In `wp-theme-development/SKILL.md`:

```markdown
### Template Files (`*.php` in theme root, `/templates/`)
Scan for:
- Missing output escaping → CRITICAL: XSS vulnerability
  - `<?php echo $user_content; ?>` without escaping
  - `esc_html()`, `esc_attr()`, `esc_url()` missing
  - **See wp-security-review for comprehensive escaping guide**
- Database queries in templates → WARNING: Performance issue
  - `new WP_Query()` inside loops
  - **See wp-performance-review for query optimization**
```

### Reference Documentation Granularity

**Principle:** One reference doc per major sub-domain, sized for focused reading (300-1200 lines).

#### wp-security-review (5 references)

| Reference | Size | Coverage | Rationale |
|-----------|------|----------|-----------|
| `vulnerability-patterns.md` | ~1000 lines | Comprehensive XSS, SQL injection, CSRF, auth bypass catalog | Like anti-patterns.md - main reference |
| `escaping-guide.md` | ~400 lines | esc_html vs esc_attr vs esc_url, context-specific escaping | Most common security task |
| `sanitization-guide.md` | ~400 lines | Input sanitization, validation patterns, sanitize_* functions | Input handling deserves own doc |
| `auth-patterns.md` | ~350 lines | Capability checks, user role patterns, permission verification | Complex topic, needs isolation |
| `nonce-csrf-guide.md` | ~300 lines | Nonce creation/verification, CSRF prevention | Frequently misunderstood |

#### wp-gutenberg-blocks (6 references)

| Reference | Size | Coverage | Rationale |
|-----------|------|----------|-----------|
| `block-patterns.md` | ~800 lines | Static vs dynamic, save callback, render callback anti-patterns | Main reference like anti-patterns.md |
| `block-json-guide.md` | ~400 lines | block.json schema, attributes, supports, apiVersion | Configuration deserves own doc |
| `react-patterns.md` | ~500 lines | React/JSX patterns, hooks usage, component anti-patterns | Different language, isolated |
| `attributes-guide.md` | ~350 lines | Attribute types, sources, selectors, defaults | Deep topic, frequently consulted |
| `dynamic-blocks-guide.md` | ~400 lines | Server-side rendering, render_callback, context | Different from static blocks |
| `interactivity-api-guide.md` | ~350 lines | WP 6.5+ Interactivity API patterns | New paradigm in 2026 |

#### wp-theme-development (4 references)

| Reference | Size | Coverage | Rationale |
|-----------|------|----------|-----------|
| `theme-json-guide.md` | ~600 lines | theme.json v3, settings, styles, patterns | Central to modern themes |
| `template-patterns.md` | ~500 lines | Template hierarchy, template parts, FSE templates | Core theme concepts |
| `fse-guide.md` | ~450 lines | Full Site Editing patterns, block themes, global styles | 2026 standard approach |
| `classic-to-block-guide.md` | ~400 lines | Migration patterns, hybrid themes, backward compat | Common transition scenario |

#### wp-plugin-development (4 references)

| Reference | Size | Coverage | Rationale |
|-----------|------|----------|-----------|
| `architecture-patterns.md` | ~700 lines | OOP, MVC-like patterns, factory/strategy/observer, namespace organization | Main architectural reference |
| `hooks-guide.md` | ~500 lines | Action/filter patterns, hook timing, priority management | Core WP plugin mechanism |
| `oop-patterns.md` | ~450 lines | Class structure, singleton (when/when not), dependency injection | Modern plugin architecture |
| `api-patterns.md` | ~400 lines | REST API registration, endpoints, permissions, responses | Modern API approach |

## Slash Command Architecture

### Command Pair Pattern (Full + Quick)

Each skill has 2 commands following the established pattern:

| Command | Purpose | SKILL.md Section Used | Output |
|---------|---------|----------------------|--------|
| `/wp-[domain]-review [path]` | Comprehensive review | Full Code Review Workflow | Grouped by severity, line numbers, fixes |
| `/wp-[domain] [path]` | Fast triage | Search Patterns for Quick Detection only | File:line matches, critical patterns only |

### Grep Pattern Design (Quick Scan Commands)

**Principle:** Grep patterns optimized for signal-to-noise ratio.

```bash
# Security quick scan patterns (wp-sec)
# CRITICAL: Unescaped output
grep -rn "echo \\\$_\|print \\\$_\|<\?=\s*\\\$_" --include="*.php" .

# CRITICAL: SQL without prepare
grep -rn "\\\$wpdb->query\|->get_results\|->get_var" --include="*.php" . | grep -v "prepare"

# CRITICAL: Missing nonce verification
grep -rn "wp_ajax_\|admin-post" --include="*.php" . | grep -v "wp_verify_nonce\|check_ajax_referer"

# WARNING: Direct file access
grep -rn "file_get_contents\|fopen\|readfile" --include="*.php" . | grep -v "ABSPATH"
```

```bash
# Gutenberg quick scan patterns (wp-block)
# CRITICAL: Missing block.json
find . -name "*.php" -path "*/blocks/*" -type f | while read file; do
  dirname "$file"
done | sort -u | while read dir; do
  [ ! -f "$dir/block.json" ] && echo "Missing block.json: $dir"
done

# WARNING: Static blocks without reason
grep -rn "save:\s*function\|save:\s*(" --include="*.js" .

# WARNING: Missing apiVersion in block.json
grep -L "apiVersion" */blocks/*/block.json
```

## Build Order: Dependency-Driven Sequence

### Recommended Order

**Phase 1: Security (Build First)**
- **Why first:** Foundational skill, least dependent on others
- Security patterns are referenced by all other skills
- Establishes escaping/sanitization patterns used in theme, block, plugin development
- Independent domain - can be built without referencing other skills

**Phase 2: Plugin Development (Build Second)**
- **Why second:** Architectural foundation for blocks
- Hook patterns inform theme development
- Architecture patterns referenced by block skill (block registration is plugin-like)
- Establishes OOP patterns used in complex themes

**Phase 3: Gutenberg Blocks (Build Third)**
- **Why third:** Depends on plugin (registration) and security (escaping)
- References plugin skill for block registration patterns
- References security skill for nonce handling in block AJAX
- Block patterns inform theme development (pattern registration)

**Phase 4: Theme Development (Build Last)**
- **Why last:** Most dependent on other skills
- References security (escaping in templates)
- References performance (query optimization in templates)
- References blocks (pattern registration, block theme concepts)
- References plugin (hooks for theme customization)

### Build Dependencies Diagram

```
Security ─────────────────┐
    │                      │
    │                      ▼
    └────────► Plugin ──► Theme
                 │          ▲
                 │          │
                 └─► Block ─┘
```

### Shared Patterns Extraction (Future Consideration)

**Not recommended for v1**, but if duplication becomes problematic:

Create `skills/_shared/` with common references:
- `hooks-reference.md` (used by performance, plugin, theme)
- `escaping-reference.md` (used by security, theme, blocks)
- `database-reference.md` (used by performance, plugin, security)

Skills would reference: `../../_shared/hooks-reference.md`

**Why defer:** Premature abstraction. Build skills first, identify true duplication patterns, then extract if needed.

## File Structure Per New Skill

### wp-security-review

```
skills/wp-security-review/
  SKILL.md
    Frontmatter: name, description with triggers (XSS, SQL injection, CSRF, auth bypass, etc.)
    Overview: Code-level security review (not server hardening)
    When to Use: Security audits, pre-launch checks, vulnerability reports
    Workflow: Scan critical vulnerabilities → warnings → info
    File-Type Checks: PHP (escaping, SQL, nonces), JS (DOM XSS), templates
    Search Patterns: Unescaped output, SQL without prepare, missing nonces
    Platform Context: WordPress VIP security helpers, hosting WAF integration
    Quick Reference: XSS prevention, SQL injection prevention, CSRF protection
    Severity: Critical (exploitable), Warning (hardening opportunity), Info (best practice)
    Output Format: Vulnerability findings with CVSS-like severity
    Common Mistakes: Escaping too early, missing capability checks, nonce reuse
    References: Map to 5 reference docs

  references/
    vulnerability-patterns.md
      Quick Lookup Table (pattern → severity → issue)
      XSS Patterns (reflected, stored, DOM-based)
      SQL Injection Patterns (prepared statements, wpdb usage)
      CSRF/Nonce Patterns
      Auth Bypass Patterns (capability checks, user role validation)
      Path Traversal, File Upload, XXE patterns

    escaping-guide.md
      Context-Specific Escaping (HTML, attribute, URL, JS)
      esc_html() vs esc_attr() vs esc_url() vs esc_js()
      wp_kses() and allowed HTML
      Late escaping principle
      Common escaping mistakes

    sanitization-guide.md
      Input Validation vs Sanitization
      sanitize_text_field(), sanitize_email(), etc.
      wp_unslash() and magic quotes
      Custom sanitization callbacks
      Validation patterns for custom data types

    auth-patterns.md
      current_user_can() capability checks
      User role hierarchy
      Custom capability registration
      is_user_logged_in() patterns
      Admin context checks (is_admin, wp_doing_ajax, wp_doing_cron)

    nonce-csrf-guide.md
      wp_create_nonce() / wp_verify_nonce()
      check_ajax_referer() / check_admin_referer()
      Nonce lifespan and caching implications
      CSRF protection for REST API
      Common nonce mistakes (reuse, front-end exposure)

commands/
  wp-sec-review.md
    description: WordPress security code review - detects XSS, SQL injection, CSRF, auth issues
    argument-hint: [file-or-directory]
    Instructions: Use wp-security-review skill, full workflow, severity-based output

  wp-sec.md
    description: Quick WordPress security scan - fast pattern detection for critical vulnerabilities
    argument-hint: [path]
    Instructions: Use wp-security-review skill, Search Patterns section only, report critical matches
```

### wp-gutenberg-blocks

```
skills/wp-gutenberg-blocks/
  SKILL.md
    Frontmatter: name, description with triggers (block development, Gutenberg, FSE, block.json, etc.)
    Overview: Block Editor code review and development patterns (WP 6.x+)
    When to Use: Block plugin dev, FSE theme blocks, custom block review
    Workflow: Identify block type → check registration → review attributes → validate rendering
    File-Type Checks: block.json, JS/JSX, PHP render callbacks, CSS
    Search Patterns: Missing block.json, static vs dynamic issues, deprecated patterns
    Platform Context: WordPress.com block development, Gutenberg plugin vs core
    Quick Reference: Static vs dynamic blocks, attribute patterns, InnerBlocks usage
    Severity: Critical (breaks editor), Warning (UX/performance issue), Info (best practice)
    Output Format: Block-specific findings with editor/frontend context
    Common Mistakes: Over-using static blocks, missing apiVersion, improper attribute sources
    References: Map to 6 reference docs

  references/
    block-patterns.md
      Static vs Dynamic Blocks (when to use each)
      Block Registration (registerBlockType, block.json)
      Render Callback Anti-Patterns
      Save Function Anti-Patterns
      Block Deprecation Strategies
      Common Block Development Mistakes

    block-json-guide.md
      block.json Schema Reference
      apiVersion (2 vs 3)
      Attributes Configuration
      Supports Configuration
      Style/Script Registration
      Example/Variations

    react-patterns.md
      WordPress React Components (@wordpress/components)
      Block Editor Hooks (useBlockProps, useInnerBlocksProps)
      State Management in Blocks
      JSX Anti-Patterns in WordPress
      Performance Considerations (memoization, useEffect)

    attributes-guide.md
      Attribute Types (string, number, boolean, object, array)
      Attribute Sources (attribute, text, html, query, meta)
      Selectors and Queries
      Default Values and Validation
      Attribute Anti-Patterns

    dynamic-blocks-guide.md
      When to Use Dynamic Rendering
      render_callback Best Practices
      Block Context API
      Server-Side Rendering Performance
      Caching Dynamic Blocks

    interactivity-api-guide.md
      Interactivity API Overview (WP 6.5+)
      Directives (data-wp-bind, data-wp-on, etc.)
      Store Configuration
      When to Use vs React
      Migration from Client-Side Rendering
```

### wp-theme-development

```
skills/wp-theme-development/
  SKILL.md
    Frontmatter: name, description with triggers (theme development, FSE, theme.json, templates, etc.)
    Overview: Theme code review and development patterns (WP 6.x+ block themes)
    When to Use: Theme development, FSE theme review, theme.json configuration
    Workflow: Identify theme type → check structure → review templates → validate theme.json
    File-Type Checks: theme.json, templates/*.html, functions.php, style.css
    Search Patterns: Missing theme.json, classic theme anti-patterns, FSE issues
    Platform Context: WordPress.com theme review, theme directory requirements
    Quick Reference: theme.json v3, template hierarchy, FSE patterns
    Severity: Critical (breaks site), Warning (UX/compatibility issue), Info (best practice)
    Output Format: Theme-specific findings with context (classic vs block theme)
    Common Mistakes: Mixing classic and block patterns, improper theme.json settings
    References: Map to 4 reference docs

  references/
    theme-json-guide.md
      theme.json v3 Reference
      Settings Configuration (color, typography, spacing, layout)
      Styles Configuration (blocks, elements, variations)
      Custom Templates and Template Parts
      Patterns Registration
      Theme.json Anti-Patterns

    template-patterns.md
      Template Hierarchy in Block Themes
      HTML Templates vs PHP Templates
      Template Parts (header, footer, sidebar)
      Template-Specific Files (single, archive, page, home)
      Query Loop Patterns

    fse-guide.md
      Full Site Editing Overview
      Global Styles and Style Variations
      Site Editor Patterns
      Block Theme Requirements
      FSE Performance Considerations

    classic-to-block-guide.md
      Migration Strategies (hybrid themes, phased approach)
      Backward Compatibility Patterns
      Classic Theme Blocks (theme.json in classic themes)
      get_template_part() vs template parts
      Converting PHP Templates to HTML
```

### wp-plugin-development

```
skills/wp-plugin-development/
  SKILL.md
    Frontmatter: name, description with triggers (plugin development, architecture, hooks, OOP, etc.)
    Overview: Plugin architecture and development patterns (WP 6.x+)
    When to Use: Plugin development, architecture review, code organization
    Workflow: Identify plugin scope → check architecture → review hooks → validate API usage
    File-Type Checks: Main plugin file, class files, REST API files
    Search Patterns: Namespace issues, hook misuse, architectural anti-patterns
    Platform Context: WordPress.com plugin review, WordPress.org plugin directory
    Quick Reference: OOP patterns, hook patterns, REST API patterns
    Severity: Critical (breaks functionality), Warning (maintainability issue), Info (best practice)
    Output Format: Architecture-focused findings with scalability context
    Common Mistakes: Singleton overuse, global scope pollution, improper hook priority
    References: Map to 4 reference docs

  references/
    architecture-patterns.md
      Plugin Structure Best Practices
      Namespace Organization
      Autoloading Strategies
      Factory Pattern in WP Plugins
      Strategy Pattern for Extensibility
      Observer Pattern and Hooks
      MVC-Like Architecture in WordPress

    hooks-guide.md
      Action vs Filter Deep Dive
      Hook Priority and Execution Order
      add_action/add_filter Best Practices
      Removing Hooks from Other Plugins
      Custom Hooks for Extensibility
      Hook Performance Considerations

    oop-patterns.md
      Class Structure and Organization
      Singleton Pattern (When to Use/Avoid)
      Dependency Injection in WordPress
      Interface Usage
      Abstract Classes and Inheritance
      Namespacing Strategies

    api-patterns.md
      REST API Endpoint Registration
      Permission Callbacks
      Request Validation and Sanitization
      Response Formatting
      API Versioning
      REST vs admin-ajax.php
```

## Command Grep Pattern Summary

Quick scan commands use these grep strategies:

| Skill | Critical Patterns | Example Grep |
|-------|------------------|--------------|
| **Security** | Unescaped output, SQL without prepare, missing nonces | `grep -rn "echo \\\$_" --include="*.php"` |
| **Gutenberg** | Missing block.json, static blocks without reason | `find */blocks/* -type d ! -exec test -e '{}/block.json' \\;` |
| **Theme** | Missing theme.json, direct DB queries in templates | `grep -rn "new WP_Query\|get_posts" templates/ parts/` |
| **Plugin** | Global scope pollution, missing namespace, hook on init | `grep -rn "function [a-z_]*(" --include="*.php" \| grep -v "namespace"` |

## Cross-Skill Communication Patterns

### When One Skill References Another

**In SKILL.md "When to Use" section:**

```markdown
**Don't use for:**
- Security-only audits (use wp-security-review)
- Block-specific development (use wp-gutenberg-blocks)
```

**In File-Type Specific Checks:**

```markdown
- Missing output escaping → CRITICAL: XSS vulnerability
  - **See wp-security-review skill for escaping guide**
```

**In Deep-Dive References section:**

```markdown
| Task | Reference to Load |
|------|-------------------|
| Security issues in templates | Load wp-security-review skill: escaping-guide.md |
| Query optimization | Load wp-performance-review skill: wp-query-guide.md |
```

## Implementation Checklist Per Skill

### For Each New Skill:

- [ ] **SKILL.md (500-600 lines)**
  - [ ] YAML frontmatter (name, description with 10-15 trigger phrases)
  - [ ] 12 sections following template
  - [ ] File-type specific checks with severity tags
  - [ ] 15-25 grep patterns for quick detection
  - [ ] 50+ code examples with ❌/✅ comments
  - [ ] Cross-references to other skills in "When to Use" and checks sections
  - [ ] Deep-dive references table mapping tasks to reference docs

- [ ] **Reference Docs (3-6 files, 300-1200 lines each)**
  - [ ] Quick lookup table at top (pattern → severity → issue)
  - [ ] Organized by sub-domain (e.g., XSS, SQL injection, CSRF for security)
  - [ ] Code examples following WordPress PHP Coding Standards
  - [ ] ❌ BAD / ✅ GOOD comment pattern
  - [ ] Real-world scenarios and consequences
  - [ ] Platform-specific variations (VIP, Pantheon, etc.)

- [ ] **Slash Commands (2 files)**
  - [ ] Full review command (`/wp-[domain]-review`)
    - YAML: description, argument-hint
    - Instructions: Use skill, full workflow, all references
  - [ ] Quick scan command (`/wp-[domain]`)
    - YAML: description, argument-hint
    - Instructions: Use skill, Search Patterns only, critical issues

- [ ] **Cross-References**
  - [ ] Identify overlapping topics with other skills
  - [ ] Add explicit cross-reference callouts in SKILL.md
  - [ ] Update other skills' "Don't use for" sections if needed

- [ ] **Testing**
  - [ ] Verify all grep patterns produce reasonable signal-to-noise
  - [ ] Test YAML frontmatter description length (< 1024 chars)
  - [ ] Validate all code examples follow WordPress PHP Coding Standards
  - [ ] Check cross-references point to correct skills

## Quality Metrics

| Metric | Target | Rationale |
|--------|--------|-----------|
| SKILL.md length | 500-600 lines | Matches wp-performance-review depth |
| Reference count | 3-6 per skill | Based on domain complexity |
| Reference size | 300-1200 lines | Focused, scannable, not overwhelming |
| Code examples | 50+ per SKILL.md | Enough coverage for pattern recognition |
| Grep patterns | 15-25 per skill | Critical patterns only, avoid noise |
| Cross-references | 3-5 per skill | Enough connection without dependency hell |
| Severity tags | Every pattern tagged | Enables triage workflow |

## Validation Against Existing Pattern

The wp-performance-review skill validates this architecture:

- **SKILL.md:** 507 lines ✓ (target: 500-600)
- **References:** 4 files ✓ (target: 3-6)
- **Reference sizes:** 328-1199 lines ✓ (target: 300-1200)
- **Commands:** 2 (full + quick) ✓
- **Structure:** 12 sections ✓
- **Code examples:** 60+ ✓ (target: 50+)
- **Grep patterns:** 21 ✓ (target: 15-25)

## Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Cross-skill duplication** | Maintenance burden, version skew | Explicit cross-references, defer shared extraction to v2 |
| **Grep pattern false positives** | User frustration with quick scans | Test patterns against real codebases, exclude common false positive paths |
| **Reference doc scope creep** | 2000+ line reference files | Strict sub-domain boundaries, split if exceeds 1200 lines |
| **Security skill too broad** | Overlaps with performance, plugin skills | Keep security skill code-level only, cross-reference for patterns |
| **Gutenberg skill outdated quickly** | WP 6.x changes frequently | Focus on stable APIs (block.json, apiVersion 3), note Gutenberg plugin vs core |
| **Theme skill classic vs block split** | Two paradigms in one skill | Unified skill with clear classic vs block sections, migration guide reference |

## Sources

Security research:
- [WordPress Security Best Practices 2026](https://www.adwaitx.com/wordpress-security-best-practices/)
- [WordPress Vulnerabilities Database 2026](https://wpsecurityninja.com/wordpress-vulnerabilities-database/)
- [WordPress Security Guide 2026](https://www.bitcot.com/wordpress-security-challenges-and-solutions/)
- [Understanding WordPress Security Functions](https://www.voxfor.com/understanding-wordpress-security-functions/)
- [Security Theme Handbook](https://developer.wordpress.org/themes/advanced-topics/security/)
- [XSS Vulnerabilities Prevention](https://wpshout.com/wordpress-xss-attack/)

Gutenberg research:
- [Gutenberg Blocks in 2026](https://vapvarun.com/gutenberg-blocks-2026-wordpress-block-editor-ai-era/)
- [Block Editor Handbook](https://developer.wordpress.org/block-editor/)
- [10up WP Block Editor Best Practices](https://gutenberg.10up.com/reference/Blocks/custom-blocks/)
- [Getting Started – Block Editor Handbook](https://developer.wordpress.org/block-editor/getting-started/)
- [What's new for developers January 2026](https://developer.wordpress.org/news/2026/01/whats-new-for-developers-january-2026/)

Theme development research:
- [Global Settings and Styles (theme.json)](https://developer.wordpress.org/themes/global-settings-and-styles/)
- [Theme.json Version 3 Reference](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/)
- [The Anatomy of theme.json](https://deliciousbrains.com/the-anatomy-of-theme-json-a-developers-cheat-sheet/)
- [Latest Trends in WordPress Development for 2026](https://wpdeveloper.com/latest-trends-in-wordpress-development/)

Plugin development research:
- [WordPress Plugin Architecture: OOP and Design Patterns](https://www.voxfor.com/wordpress-plugin-architecture-oop-design-patterns/)
- [Best Practices – Plugin Handbook](https://developer.wordpress.org/plugins/plugin-basics/best-practices/)
- [WordPress Architecture Guide](https://www.bluehost.com/blog/wordpress-architecture/)
- [Query Monitor Documentation](https://wordpress.org/plugins/query-monitor/)
- [Performance Lab Plugin](https://wordpress.org/plugins/performance-lab/)

---

*Architecture research completed: 2026-02-06*
