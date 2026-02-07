# Research Summary: Feature Definition for 4 WordPress Skills

**Domain:** WordPress Development (Security, Block Editor, Themes, Plugins)
**Researched:** 2026-02-06
**Overall confidence:** HIGH

---

## Executive Summary

This research examined four WordPress skill domains to define what each code review skill should cover. The skills target different aspects of WordPress development but share common foundations in WordPress APIs, security patterns, and coding standards.

**Key Finding:** Each skill has a clear identity with minimal overlap. Security focuses on vulnerability detection, Blocks on editor integration, Themes on design systems and templates, and Plugins on architecture and WordPress.org compliance. The existing wp-performance-review skill already covers database optimization and caching, so these skills should reference it rather than duplicate that content.

**Complexity Ranking:** wp-security-review (highest) and wp-gutenberg-blocks are the most complex, requiring deep domain expertise and context-aware analysis. wp-theme-development and wp-plugin-development are medium complexity with well-established patterns.

**2026 Context:** The WordPress ecosystem is actively evolving with the Interactivity API (WP 6.5+), theme.json version 3, and stricter Plugin Check standards. Skills should balance current best practices with forward-looking patterns.

---

## Key Findings

**Stack:** WordPress 6.5+ with Block Editor (Gutenberg), PHP 7.4+ (8.0+ recommended), modern JavaScript (React 18+), theme.json version 2/3 for block themes

**Architecture:** Skills follow the same structure as wp-performance-review: SKILL.md (~500 lines) with YAML frontmatter, file-type specific checks, grep patterns, severity levels (CRITICAL/WARNING/INFO), and reference documents. Each skill is independent but can reference others.

**Critical pitfall:** The Block Editor ecosystem has significant breaking changes via block deprecation. Any skill covering blocks must understand save function validation, migration patterns, and the InnerBlocks content persistence gotcha.

---

## Implications for Roadmap

Based on research, the suggested milestone structure would be:

### Phase 1: Foundation (Security + Plugin)
**Rationale:** Security is the highest priority (35% XSS, 15% SQL injection, 11% CSRF) and applies to all WordPress code. Plugin development patterns provide the architectural foundation that themes and blocks build upon.

**Skills:**
- wp-security-review (CRITICAL priority, prevents vulnerabilities)
- wp-plugin-development (foundational patterns, WordPress.org standards)

**Dependencies:** These two skills are independent and can be developed in parallel. Both reference official WordPress APIs and have stable, well-documented patterns.

### Phase 2: Modern WordPress (Blocks + Themes)
**Rationale:** Block Editor and FSE represent modern WordPress. Blocks must be developed before themes because block themes integrate blocks via templates. The Interactivity API (WP 6.5+) is the future direction.

**Skills:**
- wp-gutenberg-blocks (modern development, Interactivity API)
- wp-theme-development (FSE, theme.json, design systems)

**Dependencies:** Theme skill references block patterns and block stylesheets, so blocks should be documented first. However, both can be developed in parallel if coordination exists.

---

## Phase Ordering Rationale

**Why Security + Plugin first:**
1. Security affects all WordPress code (themes, plugins, blocks)
2. Plugin architecture patterns are foundational (hooks, actions, filters)
3. Both have stable, mature APIs with official documentation
4. Immediate value: prevent vulnerabilities and WordPress.org rejections

**Why Blocks + Themes second:**
1. Block themes integrate blocks, so understanding blocks first provides context
2. Both represent "modern WordPress" and appeal to the same developer audience
3. Interactivity API is cutting-edge (WP 6.5+), positioning skills as forward-looking
4. Theme skill can reference block patterns/variations documented in block skill

**Alternative consideration:** If resources are limited, prioritize Security > Plugin > Blocks > Themes (sequential). Security has the highest ROI for preventing critical issues.

---

## Research Flags for Implementation

### wp-security-review: Likely needs deeper research
**Reason:** Privilege escalation detection and object injection patterns require PHP internals knowledge. May need additional research on:
- Magic method exploitation patterns (__toString, __destruct, __wakeup)
- Advanced unserialize() attack vectors
- WordPress-specific gadget chains

**Mitigation:** Start with table-stakes checks (XSS, SQL injection, CSRF) in MVP. Add advanced patterns in iteration 2.

### wp-gutenberg-blocks: Standard patterns, minimal additional research
**Reason:** Official Block Editor Handbook is comprehensive and authoritative. Interactivity API is well-documented since WP 6.5.

**Note:** Block deprecation patterns are complex but documented. InnerBlocks gotchas are GitHub issue threads with clear solutions.

### wp-theme-development: Standard patterns, minimal additional research
**Reason:** theme.json specification is stable. Template hierarchy is foundational WordPress knowledge.

**Note:** FSE is still evolving but patterns are established. Style variations and block patterns are well-documented.

### wp-plugin-development: Standard patterns, Plugin Check reference needed
**Reason:** Plugin Handbook is mature. Plugin Check tool provides definitive standards for WordPress.org submission.

**Action:** Reference Plugin Check GitHub repo for exact validation rules: https://github.com/WordPress/plugin-check

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Security patterns | HIGH | Official WordPress docs, stable APIs, CVE analysis available |
| Security threats (2026) | MEDIUM | Current trends from WebSearch + security blogs, threats evolve |
| Block Editor APIs | HIGH | Official Block Editor Handbook, mature documentation |
| Interactivity API | MEDIUM | New in WP 6.5, patterns emerging but official docs exist |
| theme.json | HIGH | Stable specification, version differences documented |
| FSE patterns | MEDIUM-HIGH | Still evolving but core patterns established |
| Plugin architecture | HIGH | Mature Plugin Handbook, established best practices |
| Plugin Check standards | HIGH | Official tool with definitive validation rules |

**Overall assessment:** Research is comprehensive for MVP development. Medium-confidence areas (Interactivity API, FSE evolution, 2026 threat landscape) can be validated during implementation with official documentation.

---

## Feature Summary by Skill

### wp-security-review

**Table Stakes (10 features):**
- SQL Injection Detection (15% of vulnerabilities)
- XSS Prevention (35% of vulnerabilities, most common)
- CSRF/Nonce Verification (11.35% of vulnerabilities)
- Input Sanitization
- Output Escaping
- Capability Checks
- Authentication Validation
- REST API Permission Callbacks
- File Upload Validation
- Direct File Access Prevention

**Differentiators (10 features):**
- Privilege Escalation Detection (CVSS 10.0 pattern in 2026)
- Dangerous Function Detection (eval, base64_decode, unserialize)
- Nonce Context Validation
- WordPress-specific Anti-patterns
- AJAX Security Patterns
- Object Injection Prevention
- Magic Method Exploits
- wp_kses() vs wp_kses_post() validation
- Security Header Recommendations
- Constants Check (DISALLOW_FILE_EDIT, etc.)

**Complexity:** HIGH - Requires deep PHP security knowledge, regex patterns, context-aware analysis

### wp-gutenberg-blocks

**Table Stakes (10 features):**
- block.json Validation
- Render Patterns (Dynamic vs Static)
- InnerBlocks Usage
- useBlockProps() Hook
- Attribute Schema
- Edit vs Save Function consistency
- Block Supports
- Block Deprecation
- Script/Style Enqueuing
- Server-side Rendering

**Differentiators (10 features):**
- Interactivity API Patterns (new in WP 6.5)
- Block Variations vs Patterns guidance
- InnerBlocks.Content Gotcha in dynamic blocks
- useInnerBlocksProps Hook Order
- Block Context (providesContext/usesContext)
- Block Bindings API
- Block Patterns Registration
- Block Hooks (auto-insertion)
- Custom Block Controls (Inspector, Toolbar)
- wp_kses_post() performance anti-pattern

**Complexity:** HIGH - React + WordPress APIs, complex state management, deprecation handling

### wp-theme-development

**Table Stakes (10 features):**
- theme.json Structure
- Template Hierarchy
- Template Parts
- FSE Templates (HTML)
- Conditional Tags
- get_template_part()
- Theme Supports
- Navigation Menus
- Global Styles
- Block Patterns

**Differentiators (10 features):**
- Block Theme vs Classic Theme Detection
- theme.json Version Differences (v2 vs v3)
- Hardcoded Styles Anti-pattern detection
- Block Stylesheets Pattern
- Child Theme Safety
- Custom Templates Registration
- Template Part Areas
- Style Variations
- Pattern Categories
- Root Padding Aware Alignments

**Complexity:** MEDIUM - Two paradigms (classic + FSE), but patterns are well-documented

### wp-plugin-development

**Table Stakes (10 features):**
- Plugin Header Validation
- Prefix Convention
- Activation/Deactivation Hooks
- Uninstall Cleanup
- Custom Post Types
- Custom Taxonomies
- Settings API Usage
- Options API
- Hooks (Actions/Filters)
- Internationalization (i18n)

**Differentiators (10 features):**
- Plugin Check Standards compliance
- REST API Endpoint Registration
- AJAX Handler Patterns
- Admin Menu Registration
- Custom Database Tables
- Cron Job Registration
- Transients API
- WP_Query Optimization (plugin-focused)
- Function/Class Existence Checks
- Conditional Loading (is_admin)

**Complexity:** MEDIUM-HIGH - Broad surface area, WordPress.org standards, but established patterns

---

## MVP Feature Recommendations

### wp-security-review MVP (Target: 500 lines)
**Include:**
1. XSS detection (35% of vulnerabilities)
2. SQL injection (15% of vulnerabilities)
3. CSRF (11% of vulnerabilities)
4. Input sanitization patterns
5. Capability checks
6. REST API permission callbacks
7. File upload validation basics
8. Direct file access checks

**Defer to references:**
- Advanced privilege escalation
- Object injection and magic methods
- Security headers and constants

**Rationale:** Focus on the top 3 vulnerability types (61% of all issues) plus foundational patterns.

### wp-gutenberg-blocks MVP (Target: 500 lines)
**Include:**
1. block.json validation
2. Edit vs save function consistency
3. useBlockProps() validation
4. Attribute schema patterns
5. InnerBlocks basic usage and Content gotcha
6. Script/style enqueuing
7. Dynamic vs static render patterns
8. Block deprecation basics

**Defer to references:**
- Interactivity API deep dive
- Block Context advanced patterns
- Block Bindings API

**Rationale:** Cover foundational block structure before advanced state management patterns.

### wp-theme-development MVP (Target: 500 lines)
**Include:**
1. Block theme vs classic detection
2. theme.json structure (version, settings, styles)
3. Template hierarchy basics
4. Template parts (header, footer)
5. FSE templates validation
6. Hardcoded styles detection
7. Theme supports
8. Global styles patterns

**Defer to references:**
- Style variations
- Advanced layout patterns
- Pattern categories

**Rationale:** Focus on modern block themes while maintaining classic theme compatibility.

### wp-plugin-development MVP (Target: 500 lines)
**Include:**
1. Plugin header validation
2. Prefix conventions
3. Activation/deactivation hooks
4. Custom post types
5. Settings API
6. Internationalization (i18n)
7. Options API
8. Basic REST API endpoint patterns

**Defer to references:**
- Custom database tables
- Advanced Plugin Check compliance
- WP-CLI commands

**Rationale:** Cover WordPress.org submission requirements and foundational architecture patterns.

---

## Cross-Skill Feature Matrix

How features overlap or differ across skills:

| Feature Category | Security | Blocks | Themes | Plugins |
|------------------|----------|--------|--------|---------|
| **Input Sanitization** | ✓ Core | ✓ Attributes | ✓ Theme options | ✓ Settings |
| **Output Escaping** | ✓ Core | ✓ Render | ✓ Templates | ✓ Admin pages |
| **Database Queries** | ✓ SQL injection | — | ✓ Template queries | ✓ Custom tables |
| **Hooks/Filters** | — | ✓ Block hooks | ✓ Theme hooks | ✓ Core pattern |
| **REST API** | ✓ Permissions | ✓ Block attributes | — | ✓ Endpoints |
| **AJAX** | ✓ Nonces | ✓ Dynamic blocks | — | ✓ Handlers |
| **Internationalization** | — | ✓ Block strings | ✓ Theme strings | ✓ Plugin strings |
| **File Structure** | ✓ Direct access | ✓ block.json | ✓ theme.json | ✓ Plugin header |
| **Caching** | — | — | ✓ Template caching | ✓ Transients |
| **Performance** | — | ✓ Asset loading | ✓ Block stylesheets | ✓ Conditional loading |

**Key Insight:** Security patterns (sanitization, escaping) apply across all skills. Each skill should reference wp-security-review for security details rather than duplicating content.

---

## Feature Dependencies

### Security Skill Dependencies
```
Input Sanitization → SQL Injection Detection
Output Escaping → XSS Prevention
Capability Checks → Privilege Escalation Detection
Nonce Verification → CSRF Protection
REST API Permission Callbacks → Authentication Validation
```

### Block Skill Dependencies
```
useBlockProps() → Edit/Save Functions (required wrapper)
InnerBlocks → Block Deprecation (changes break validation)
Attribute Schema → Server-side Rendering (data passed to callback)
Block Variations → Block Patterns (variations simpler than patterns)
Interactivity API → Server-side Rendering (SSR for hydration)
Block Context → InnerBlocks (context flows to children)
```

### Theme Skill Dependencies
```
theme.json → Block Theme Detection (presence indicates type)
Template Hierarchy → Conditional Tags (tags determine template)
FSE Templates → Template Parts (templates compose parts)
Global Styles → theme.json (styles defined in config)
Block Patterns → Template Parts (patterns include parts)
Style Variations → theme.json (variations extend base)
```

### Plugin Skill Dependencies
```
Prefix Convention → Function/Class Checks (prevents collisions)
Activation Hooks → Custom Post Types (CPT triggers rewrite flush)
Settings API → Options API (settings wrap options)
Custom Post Types → REST API Endpoints (auto-registration)
AJAX Handlers → REST API Endpoints (REST is modern alternative)
Cron Jobs → Activation Hooks (scheduled on activation)
Internationalization → Plugin Header (text domain matches slug)
```

---

## Gaps and Open Questions

### Security Skill Gaps
1. **Advanced PHP exploitation:** Magic method patterns and gadget chains need deeper examples
2. **File upload validation:** MIME type spoofing detection needs code examples
3. **Severity thresholds:** Need clear criteria for CRITICAL vs WARNING vs INFO

**Recommendation:** Consult WordPress VIP security docs and Wordfence blog for advanced patterns.

### Block Editor Skill Gaps
1. **Interactivity API state management:** Official docs exist but real-world patterns emerging
2. **Block Context examples:** Need practical parent-child communication examples
3. **Block Bindings API:** Very new, may need additional research during implementation

**Recommendation:** Review WordPress Core blocks (Query Block, Image Block) for reference implementations.

### Theme Skill Gaps
1. **theme.json version 3 features:** Need to document v2 vs v3 differences
2. **Classic to FSE migration:** Not covered deeply, may be out of scope
3. **Child theme patterns:** Brief mention but needs specific gotchas

**Recommendation:** Reference theme.json schema from WordPress/gutenberg GitHub repo.

### Plugin Skill Gaps
1. **Custom database table patterns:** Needs specific $wpdb security patterns
2. **WP-CLI command registration:** Not deeply researched, lower priority
3. **Multisite considerations:** Confirmed as anti-feature (too niche)

**Recommendation:** Database tables and WP-CLI suitable for Phase 2 or reference documents.

---

## Success Metrics

Each skill should enable Claude to:

1. **Detect patterns** - Identify specific code patterns in files via regex/grep
2. **Explain issues** - Describe why a pattern is problematic with severity
3. **Suggest fixes** - Provide specific WordPress function alternatives
4. **Cite references** - Link to official documentation for deep dives

**Quality indicators:**
- Clear severity levels (CRITICAL, WARNING, INFO) with consistent criteria
- Code examples showing BAD ❌ vs GOOD ✅ patterns
- File-type specific checks (functions.php vs block.json vs theme.json)
- Quick grep patterns for fast scanning
- References to official WordPress handbooks

---

## Ready for Roadmap

Research is complete and comprehensive. All four skills have:
- ✓ Clear identity and scope definition
- ✓ Table stakes vs differentiator vs anti-feature categorization
- ✓ Complexity assessment and confidence levels
- ✓ MVP recommendations with feature prioritization
- ✓ Cross-skill relationship mapping
- ✓ Official documentation sources

**Confidence: HIGH** - Ready to proceed with roadmap creation and milestone definition.

---

## Files Created

| File | Purpose |
|------|---------|
| .planning/research/FEATURES.md | Comprehensive feature landscape with all 4 skills detailed |
| .planning/research/FEATURES-SUMMARY.md | Executive summary with roadmap implications (this file) |

---

**Research completed:** 2026-02-06
