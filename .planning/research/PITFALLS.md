# Domain Pitfalls: WordPress Code Review Skills

**Domain:** WordPress Code Review Skills for Claude
**Researched:** 2026-02-06
**Confidence:** HIGH (based on existing codebase analysis, official WordPress documentation, and 2026 community patterns)

## Executive Summary

Creating WordPress code review skills for Claude differs from generic AI code review. The content IS the product—wrong advice directly harms users. This document catalogs pitfalls specific to WordPress skill creation, organized by criticality and mapped to skill types.

**Key insight:** False positives erode trust faster than false negatives. A skill that flags correct code as wrong is worse than one that misses some issues.

## Critical Pitfalls

Mistakes that cause skills to give dangerous or actively harmful advice.

### Pitfall 1: Context-Blind Performance Flags

**What goes wrong:** Flagging `posts_per_page => -1` universally without checking execution context.

**Why it happens:** Simple pattern matching without understanding WHERE the code runs.

**Consequences:**
- Flags correct admin-only code as critical issues
- Users waste time "fixing" non-problems
- Skill loses credibility
- Users stop trusting other recommendations

**Real examples from existing skill:**
```php
// ❌ WRONG TO FLAG: Admin-only export functionality
add_action('admin_post_export_data', function() {
    $posts = get_posts(['posts_per_page' => -1]); // This is fine in admin!
    generate_csv($posts);
});

// ❌ WRONG TO FLAG: WP-CLI command
if (defined('WP_CLI') && WP_CLI) {
    $all_posts = get_posts(['posts_per_page' => -1]); // CLI context is safe
}

// ❌ WRONG TO FLAG: Cron job with small dataset
add_action('daily_sync_categories', function() {
    $categories = get_terms(['taxonomy' => 'category', 'number' => 0]); // Only ~50 categories
});
```

**Prevention:**
1. Check execution context before flagging:
   - `is_admin()` - admin context
   - `wp_doing_cron()` - cron context
   - `defined('WP_CLI')` - CLI context
   - `did_action('rest_api_init')` - REST context
2. Add severity qualifiers: "CRITICAL in public-facing code, acceptable in admin/CLI/cron"
3. Include context checks in skill examples

**Detection:** Skill flags code in `/wp-admin/` files or hooks like `admin_post_*` as critical issues.

**Which skills need this:**
- wp-performance-review ✅ (already handles some cases)
- wp-security-review (needs context awareness)
- wp-plugin-development (needs to teach context checking)

---

### Pitfall 2: WordPress Version-Blind Advice

**What goes wrong:** Recommending patterns that were correct in WP 5.x but are wrong/deprecated in WP 6.x+.

**Why it happens:** Training data includes pre-6.0 advice, which dominated WordPress content until ~2023.

**Consequences:**
- Recommends deprecated functions
- Suggests workarounds for problems that are now solved
- Users implement outdated patterns in new code
- Code reviews miss modern best practices

**Concrete examples:**

```php
// ❌ OUTDATED (pre-WP 6.3): Manual defer/async
wp_enqueue_script('my-script', $url, [], '1.0', true); // Only controls footer placement

// ✅ CURRENT (WP 6.3+): Native strategy parameter
wp_enqueue_script('my-script', $url, [], '1.0', ['strategy' => 'defer']);
```

```php
// ❌ OUTDATED (pre-WP 5.8): registerBlockType without block.json
registerBlockType('namespace/block', { /* all settings in JS */ });

// ✅ CURRENT (WP 5.8+): block.json is canonical
registerBlockType(metadata); // metadata imported from block.json
```

```javascript
// ❌ OUTDATED: wp.blocks.registerBlockTypeFromMetadata
wp.blocks.registerBlockTypeFromMetadata(metadata);

// ✅ CURRENT: Use wp.blocks.registerBlockType
wp.blocks.registerBlockType(metadata);
```

**Prevention:**
1. State WordPress version requirements explicitly: "In WP 6.3+, use..."
2. Include deprecation warnings: "Prior to WP 5.8, developers used X. As of 5.8+, use Y instead."
3. Check official WordPress Developer Handbook for current patterns
4. Flag pre-5.8 block registration patterns as outdated
5. Reference version in examples: `// WordPress 6.3+ only`

**Detection:**
- Skill recommends `wp.blocks.registerBlockTypeFromMetadata` (removed)
- Skill doesn't mention `block.json` for block registration
- Skill treats Classic Editor and Block Editor as equals (Block Editor is default since WP 5.0)
- Skill doesn't mention FSE/block themes as modern pattern

**Which skills need this:**
- wp-gutenberg-blocks (CRITICAL - most affected)
- wp-theme-development (FSE vs classic themes)
- wp-performance-review (WP 6.3+ script strategies)

---

### Pitfall 3: Platform-Agnostic Cache Advice

**What goes wrong:** Recommending object cache patterns without checking if persistent object cache exists.

**Why it happens:** Assuming all WordPress installations have Redis/Memcached available.

**Consequences:**
- Recommends `wp_cache_set()` patterns that only work with persistent cache
- Users on shared hosting implement patterns that degrade performance
- Transients in database (without object cache) cause table bloat

**Real examples:**

```php
// ❌ DANGEROUS on shared hosting without object cache:
set_transient("user_{$user_id}_cart", $data, HOUR_IN_SECONDS);
// With 10,000 users = 10,000 rows in wp_options table!

// ✅ SAFE: Check for persistent cache first
if (wp_using_ext_object_cache()) {
    set_transient("user_{$user_id}_cart", $data, HOUR_IN_SECONDS);
} else {
    // Use wp_cache (non-persistent) or skip caching on shared hosting
    wp_cache_set("cart_{$user_id}", $data, 'user_carts', HOUR_IN_SECONDS);
}
```

**Platform-specific patterns:**

| Platform | Object Cache | Advice |
|----------|--------------|--------|
| WordPress VIP | Always available | Use `wpcom_vip_*` helper functions (pre-cached) |
| WP Engine | Usually available | Check `wp_using_ext_object_cache()` |
| Pantheon | Usually available | Check `wp_using_ext_object_cache()` |
| Shared hosting | Usually NOT available | Avoid dynamic transients, use `wp_cache` |

**Prevention:**
1. Always qualify cache advice with hosting context
2. Include `wp_using_ext_object_cache()` checks in examples
3. Provide fallback patterns for shared hosting
4. Recognize VIP-specific code and recommend `wpcom_vip_*` alternatives
5. Don't recommend transients for user-specific data without cache check

**Detection:**
- Skill recommends transients without mentioning `wp_using_ext_object_cache()`
- Skill doesn't differentiate between VIP/managed hosting and shared hosting
- Skill recommends dynamic transient keys universally

**Which skills need this:**
- wp-performance-review ✅ (already mentions this in some places)
- wp-plugin-development (needs hosting environment guidance)

---

### Pitfall 4: Security False Positives - Missing Nonce Verification

**What goes wrong:** Flagging missing `wp_verify_nonce()` in REST API endpoints or public read-only operations.

**Why it happens:** Applying blanket "always verify nonces" rule without understanding WordPress authentication architecture.

**Consequences:**
- Flags correct REST API code as insecure
- Users add unnecessary nonce checks that break functionality
- Creates security theater without actual security improvement

**Real examples:**

```php
// ❌ WRONG TO FLAG: REST API handles nonce verification automatically
register_rest_route('myapp/v1', '/posts', [
    'methods' => 'GET',
    'callback' => 'myapp_get_posts',
    'permission_callback' => 'is_user_logged_in', // Auth check is correct
]);

function myapp_get_posts($request) {
    // NO NEED for wp_verify_nonce() - WordPress does this in rest_cookie_check_errors()
    return get_posts(['posts_per_page' => 10]);
}
```

```php
// ❌ WRONG TO FLAG: Public read operation doesn't need nonce
add_action('wp_ajax_nopriv_get_public_posts', 'get_public_posts');

function get_public_posts() {
    // This is public data - no nonce needed
    $posts = get_posts(['posts_per_page' => 10, 'post_status' => 'publish']);
    wp_send_json_success($posts);
}
```

```php
// ✅ CORRECT TO FLAG: State-changing operation needs nonce
add_action('wp_ajax_delete_post', 'handle_delete');

function handle_delete() {
    // ❌ MISSING: check_ajax_referer('delete_post_nonce');
    wp_delete_post($_POST['post_id']);
}
```

**Prevention:**
1. Only flag missing nonces for state-changing operations (POST/DELETE/UPDATE)
2. Don't flag REST API endpoints (WordPress handles this)
3. Don't flag public read-only AJAX handlers (`wp_ajax_nopriv_*` for GET operations)
4. Include context: "Nonce verification required for state-changing operations"
5. Reference official docs: "REST API automatically validates nonces via `rest_cookie_check_errors()`"

**Detection:**
- Skill flags ALL `wp_ajax_*` handlers without nonce verification
- Skill flags REST API endpoints for missing nonces
- Skill doesn't distinguish read operations from write operations

**Which skills need this:**
- wp-security-review (CRITICAL)
- wp-plugin-development (needs to teach when nonces are needed)

---

### Pitfall 5: Gutenberg Anti-Patterns from Pre-block.json Era

**What goes wrong:** Teaching block development patterns that predate `block.json` (WordPress 5.8, December 2020).

**Why it happens:** Most block development tutorials online are from 2018-2020 era.

**Consequences:**
- Users create blocks without `block.json` (less performant)
- Miss modern features like server-side rendering declarations
- Create maintenance burden (all config in JS instead of JSON)

**Outdated patterns:**

```javascript
// ❌ OUTDATED: All configuration in registerBlockType call
registerBlockType('myblock/card', {
    title: 'Card Block',
    description: 'A card component',
    category: 'common',
    icon: 'smiley',
    attributes: {
        title: { type: 'string' }
    },
    edit: EditComponent,
    save: SaveComponent
});
```

```php
// ❌ OUTDATED: PHP registration without block.json
register_block_type('myblock/card', [
    'editor_script' => 'myblock-editor',
    'render_callback' => 'render_card_block',
]);
```

**Modern pattern:**

```json
// ✅ CURRENT: block.json (WordPress 5.8+)
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "myblock/card",
  "title": "Card Block",
  "category": "common",
  "icon": "smiley",
  "description": "A card component",
  "attributes": {
    "title": { "type": "string" }
  },
  "editorScript": "file:./index.js",
  "render": "file:./render.php"
}
```

```javascript
// ✅ CURRENT: Minimal JS registration
import metadata from './block.json';
registerBlockType(metadata);
```

**Prevention:**
1. Always recommend `block.json` as the canonical approach
2. Mark pre-5.8 patterns as "legacy" or "outdated"
3. Reference WordPress 5.8+ as minimum for modern block development
4. Include `"apiVersion": 3` in examples (Block API v3 is current)

**Detection:**
- Skill shows `registerBlockType` with all config inline
- Skill doesn't mention `block.json`
- Skill uses `apiVersion: 2` or omits it (current is 3)

**Which skills need this:**
- wp-gutenberg-blocks (CRITICAL)
- wp-theme-development (block theme patterns)

---

## Moderate Pitfalls

Mistakes that cause skills to be less useful or create minor issues.

### Pitfall 6: FSE Theme Advice That's Actually Classic Theme Advice

**What goes wrong:** Providing theme development guidance that only applies to classic themes, not block themes.

**Why it happens:** Most theme development content predates Full Site Editing (WordPress 5.9+, January 2022).

**Consequences:**
- Users build classic themes when they should build block themes
- Miss modern FSE features (template editing, global styles)
- Create themes that feel outdated on launch

**Classic theme patterns that DON'T apply to block themes:**

```php
// ❌ CLASSIC THEME ONLY: Template files with PHP loops
// wp-content/themes/mytheme/index.php
<?php get_header(); ?>
<?php if (have_posts()) : while (have_posts()) : the_post(); ?>
    <article><?php the_content(); ?></article>
<?php endwhile; endif; ?>
<?php get_footer(); ?>
```

```php
// ❌ CLASSIC THEME ONLY: add_theme_support() for features
add_theme_support('custom-logo');
add_theme_support('post-thumbnails');
add_theme_support('editor-styles');
```

**Block theme patterns:**

```json
// ✅ BLOCK THEME: theme.json replaces add_theme_support()
{
  "version": 2,
  "settings": {
    "appearanceTools": true,
    "layout": {
      "contentSize": "640px",
      "wideSize": "1200px"
    }
  }
}
```

```html
<!-- ✅ BLOCK THEME: HTML templates with block markup -->
<!-- wp-content/themes/mytheme/templates/index.html -->
<!-- wp:template-part {"slug":"header"} /-->
<!-- wp:query -->
    <!-- wp:post-template -->
        <!-- wp:post-title /-->
        <!-- wp:post-content /-->
    <!-- /wp:post-template -->
<!-- /wp:query -->
<!-- wp:template-part {"slug":"footer"} /-->
```

**Prevention:**
1. Default to block theme guidance for new themes
2. Label classic theme patterns as "Classic themes only"
3. Mention that block themes are the modern standard (WordPress 5.9+)
4. Include FSE-specific features: `theme.json`, HTML templates, template parts
5. Note: "Learning `theme.json` syntax is as critical in 2026 as knowing CSS was in 2015"

**Detection:**
- Skill recommends `add_theme_support()` without mentioning `theme.json`
- Skill shows PHP template files as primary pattern
- Skill doesn't mention Full Site Editing or block themes
- Skill treats classic and block themes as equals (block themes are modern standard)

**Which skills need this:**
- wp-theme-development (CRITICAL)
- wp-gutenberg-blocks (template editing context)

---

### Pitfall 7: Overly Aggressive Block Style Registration Warnings

**What goes wrong:** Flagging ANY use of `registerBlockStyle()` as a performance issue.

**Why it happens:** Each block style creates a preview iframe, causing editor slowdown with many styles.

**Consequences:**
- Users think block styles are always bad
- Miss legitimate use cases (2-3 styles per block is fine)
- Create overly complex alternatives for simple styling needs

**Real guidance:**

```javascript
// ✅ ACCEPTABLE: 2-3 block styles per block
registerBlockStyle('core/button', { name: 'outline', label: 'Outline' });
registerBlockStyle('core/button', { name: 'ghost', label: 'Ghost' });

// ❌ PERFORMANCE ISSUE: 10+ block styles
// Each style = separate iframe preview in editor
registerBlockStyle('core/group', { name: 'pattern-1', label: 'Pattern 1' });
registerBlockStyle('core/group', { name: 'pattern-2', label: 'Pattern 2' });
// ... 8 more ...
// Editor becomes unusable
```

**Prevention:**
1. Flag "too many block styles" not "any block styles"
2. Set threshold: "More than 5 block styles per block = performance issue"
3. Provide context: "Each block style creates a preview iframe in the editor"
4. Offer alternative: "For many style variations, use custom attributes with block filters"

**Detection:**
- Skill flags single `registerBlockStyle()` call as warning
- Skill doesn't provide quantity threshold
- Skill doesn't explain WHY it's a problem (iframe creation)

**Which skills need this:**
- wp-gutenberg-blocks
- wp-performance-review (editor performance)

---

### Pitfall 8: Plugin Architecture Advice That Ignores OOP Patterns

**What goes wrong:** Teaching procedural WordPress patterns for large plugins.

**Why it happens:** Most WordPress documentation shows simple procedural examples.

**Consequences:**
- Users build large plugins with global function soup
- Hard to test, maintain, and extend
- Miss modern PHP patterns (PSR-4 autoloading, dependency injection, etc.)

**Small plugin (procedural is fine):**

```php
// ✅ ACCEPTABLE for small plugins (~500 lines)
add_action('init', 'myplugin_register_post_type');

function myplugin_register_post_type() {
    register_post_type('event', [/* config */]);
}
```

**Large plugin (OOP is better):**

```php
// ✅ BETTER for large plugins (1000+ lines)
namespace MyPlugin;

class Plugin {
    public function __construct() {
        add_action('init', [$this, 'register_post_types']);
    }

    public function register_post_types() {
        register_post_type('event', [/* config */]);
    }
}

// Bootstrap
new Plugin();
```

**Modern patterns for 2026:**
- Factory pattern for object creation
- Strategy pattern for interchangeable algorithms
- Observer pattern for event-driven logic
- Service layer to separate business logic from WordPress coupling
- PSR-4 autoloading for class organization

**Prevention:**
1. Recommend OOP for plugins over 500-1000 lines
2. Include modern PHP patterns (not just WordPress-specific hooks)
3. Show PSR-4 autoloading for class organization
4. Reference: "Architecture depends on plugin size"
5. Provide examples of when to use each approach

**Detection:**
- Skill only shows procedural examples for plugin development
- Skill doesn't mention classes, namespaces, or autoloading
- Skill doesn't differentiate small vs large plugin patterns

**Which skills need this:**
- wp-plugin-development (CRITICAL)

---

### Pitfall 9: Yoda Conditions Applied to Non-Comparison Contexts

**What goes wrong:** Teaching Yoda conditions (`'value' === $var`) but not explaining when NOT to use them.

**Why it happens:** WordPress Coding Standards require Yoda conditions, but examples don't show exceptions.

**Consequences:**
- Code that's harder to read than necessary
- False flags on code that doesn't benefit from Yoda conditions

**Where Yoda conditions make sense:**

```php
// ✅ GOOD: Prevents accidental assignment
if ('publish' === $post->post_status) {
    // If you typo = instead of ==, you get error instead of assignment
}
```

**Where Yoda conditions are awkward:**

```php
// ❌ AWKWARD: Function return values
if (true === is_admin()) { // Verbose
if (null === get_post()) { // Awkward

// ✅ BETTER: Natural reading order for booleans/null checks
if (is_admin()) {
if (get_post() !== null) {
```

**Prevention:**
1. Explain WHY Yoda conditions exist (prevent accidental assignment)
2. Note they're less critical in PHP 7+ (assignment in conditionals shows warnings)
3. Show exceptions: boolean checks, null checks, function returns
4. Don't flag natural order for `is_*()` functions and boolean checks

**Detection:**
- Skill flags ALL non-Yoda comparisons
- Skill doesn't explain the reason behind Yoda conditions
- Skill shows `true === is_admin()` as the only correct pattern

**Which skills need this:**
- wp-plugin-development
- wp-theme-development
- All skills that reference WordPress coding standards

---

### Pitfall 10: InnerBlocks Sanitization That Breaks Functionality

**What goes wrong:** Recommending `wp_kses_post()` on InnerBlocks content.

**Why it happens:** Generic security advice: "Always sanitize output."

**Consequences:**
- Breaks embeds (YouTube, Twitter, etc.)
- Strips iframes and other allowed-but-not-in-kses content
- Users file bug reports about broken functionality

**The problem:**

```php
// ❌ BREAKS EMBEDS: InnerBlocks content is already sanitized
function render_my_block($attributes, $content) {
    return '<div class="my-block">' . wp_kses_post($content) . '</div>';
}
// Result: YouTube embeds stripped, iframes removed
```

**The fix:**

```php
// ✅ CORRECT: InnerBlocks content is pre-sanitized by WordPress
function render_my_block($attributes, $content) {
    return '<div class="my-block">' . $content . '</div>';
}

// ✅ SANITIZE ATTRIBUTES: But DO escape attributes
function render_my_block($attributes, $content) {
    $class = esc_attr($attributes['className'] ?? '');
    return '<div class="my-block ' . $class . '">' . $content . '</div>';
}
```

**Prevention:**
1. Note: "InnerBlocks content is already sanitized by WordPress"
2. Distinguish between attributes (need escaping) and InnerBlocks content (pre-sanitized)
3. Explain WHEN to sanitize: user input, not WordPress-generated content
4. Include example showing broken embeds from over-sanitization

**Detection:**
- Skill recommends `wp_kses_post()` on `$content` from dynamic block render callback
- Skill doesn't distinguish InnerBlocks from raw user input
- Skill treats ALL output as needing sanitization

**Which skills need this:**
- wp-security-review (CRITICAL)
- wp-gutenberg-blocks (block development context)

---

## Minor Pitfalls

Mistakes that are minor annoyances but don't cause major issues.

### Pitfall 11: Outdated Script/Style Enqueue Examples

**What goes wrong:** Showing old patterns for asset enqueuing without modern loading strategies.

**Why it happens:** Pre-WP 6.3 examples dominate search results.

**Consequences:**
- Users miss free performance wins from defer/async
- Code reviews don't catch missing version parameters

**Outdated pattern:**

```php
// ❌ OUTDATED: No loading strategy (pre-WP 6.3)
wp_enqueue_script('my-script', $url, [], '1.0', true); // true = footer only

// ❌ OUTDATED: No version parameter (cache busting issues)
wp_enqueue_script('my-script', $url);
```

**Modern pattern:**

```php
// ✅ MODERN (WP 6.3+): Loading strategy
wp_enqueue_script('my-script', $url, [], '1.0', [
    'strategy' => 'defer', // or 'async'
    'in_footer' => true
]);

// ✅ GOOD: Version parameter with constant
define('THEME_VERSION', '1.0.0');
wp_enqueue_script('my-script', $url, [], THEME_VERSION, [
    'strategy' => 'defer'
]);
```

**Prevention:**
1. Show WP 6.3+ loading strategies in examples
2. Always include version parameter in examples
3. Note backwards compatibility: "For WP < 6.3, use footer parameter only"
4. Flag missing versions as INFO level (not critical, but good practice)

**Detection:**
- Skill shows enqueue examples without strategy parameter
- Skill doesn't mention defer/async for script loading
- Examples omit version parameter

**Which skills need this:**
- wp-performance-review ✅ (already mentions this)
- wp-theme-development

---

### Pitfall 12: Confusing "No Object Cache" with "No Caching"

**What goes wrong:** Recommending against ALL caching on shared hosting.

**Why it happens:** Misunderstanding the difference between persistent object cache and non-persistent cache.

**Consequences:**
- Users avoid `wp_cache_*` functions entirely on shared hosting
- Miss performance wins from non-persistent cache (per-request)

**The reality:**

```php
// ✅ ALWAYS WORKS: wp_cache (non-persistent, per-request)
// This works on ALL hosting (with or without Redis/Memcached)
wp_cache_set('key', $data, 'group', $ttl);
$data = wp_cache_get('key', 'group');

// ⚠️ DEPENDS ON HOSTING: set_transient (persistent, stored in DB without object cache)
// On shared hosting without Redis: Creates rows in wp_options table
set_transient('key', $data, $ttl);
```

**Prevention:**
1. Clarify: "`wp_cache` works everywhere (non-persistent)"
2. Clarify: "`set_transient` requires object cache for user-specific data"
3. Show: "Check `wp_using_ext_object_cache()` before using transients for dynamic keys"
4. Don't recommend avoiding caching entirely on shared hosting

**Detection:**
- Skill says "don't use caching on shared hosting"
- Skill treats `wp_cache` and `set_transient` as requiring object cache
- Skill doesn't explain difference between persistent and non-persistent cache

**Which skills need this:**
- wp-performance-review
- wp-plugin-development

---

## Phase-Specific Warnings

Pitfalls that will surface during specific skill development phases.

| Skill | Likely Pitfall | When It Surfaces | Mitigation |
|-------|---------------|------------------|------------|
| **wp-security-review** | Nonce false positives (Pitfall 4) | Core pattern development | Check for REST API and read-only contexts before flagging |
| **wp-security-review** | InnerBlocks sanitization (Pitfall 10) | Block security checks | Document that InnerBlocks content is pre-sanitized |
| **wp-gutenberg-blocks** | Pre-block.json patterns (Pitfall 5) | Example creation | Always show block.json as canonical |
| **wp-gutenberg-blocks** | Block style over-flagging (Pitfall 7) | Performance checks | Set threshold (5+ styles = issue) |
| **wp-theme-development** | Classic theme advice for FSE (Pitfall 6) | Template guidance | Default to block themes, label classic patterns |
| **wp-theme-development** | Missing theme.json (Pitfall 6) | Configuration docs | Show theme.json as replacement for add_theme_support() |
| **wp-plugin-development** | Only procedural patterns (Pitfall 8) | Architecture guidance | Show OOP for large plugins (1000+ lines) |
| **wp-plugin-development** | Yoda condition zealotry (Pitfall 9) | Coding standards | Explain WHY and exceptions |

## Cross-Cutting Concerns

Pitfalls that affect multiple skills.

### Version Context in All Skills

**Problem:** Advice that was correct in WordPress 5.x but wrong in 6.x+.

**Solution for all skills:**
1. State WordPress version requirements: "WordPress 6.3+ introduces..."
2. Label outdated patterns: "Pre-5.8 pattern (use block.json instead)"
3. Check official docs before asserting capability
4. Flag "As of my training" statements as LOW confidence

### Platform Context in Performance/Security Skills

**Problem:** Advice that depends on hosting environment (VIP vs shared hosting).

**Solution:**
1. Always qualify recommendations with platform context
2. Provide platform-specific alternatives (VIP helpers, shared hosting fallbacks)
3. Check for `wp_using_ext_object_cache()` in caching examples
4. Don't assume all installs have Redis/Memcached

### Context Detection Patterns

**Key contexts to check:**
```php
// Admin context
is_admin()

// Cron context
wp_doing_cron()

// CLI context
defined('WP_CLI') && WP_CLI

// REST API context
did_action('rest_api_init')

// AJAX context
wp_doing_ajax()

// Frontend context
!is_admin() && !wp_doing_cron() && !wp_doing_ajax()
```

## Verification Checklist

Before releasing a skill, verify:

- [ ] Examples include WordPress version requirements
- [ ] Context-dependent advice qualifies WHERE it applies (admin vs frontend)
- [ ] Platform-dependent advice mentions hosting requirements
- [ ] Security advice doesn't flag REST API or read-only operations incorrectly
- [ ] Gutenberg advice uses block.json (not pre-5.8 patterns)
- [ ] Theme advice defaults to block themes (not classic themes)
- [ ] Caching advice checks for persistent object cache availability
- [ ] False positive scenarios documented in Common Mistakes section
- [ ] Examples use current WordPress APIs (not deprecated ones)

## Testing Scenarios

**Test each skill with code that SHOULD NOT be flagged:**

```php
// Scenario 1: Admin-only unbounded query
add_action('admin_post_export', function() {
    $posts = get_posts(['posts_per_page' => -1]); // Should NOT flag
});

// Scenario 2: REST API without explicit nonce
register_rest_route('app/v1', '/data', [
    'callback' => 'get_data',
    'permission_callback' => 'is_user_logged_in'
]); // Should NOT flag missing nonce

// Scenario 3: InnerBlocks without wp_kses_post
function render_block($attr, $content) {
    return '<div>' . $content . '</div>'; // Should NOT flag
}

// Scenario 4: WP 6.3+ script strategy
wp_enqueue_script('script', $url, [], '1.0', ['strategy' => 'defer']); // Should NOT flag as outdated

// Scenario 5: Block registration with block.json
registerBlockType(metadata); // Should NOT flag as missing configuration
```

## Documentation Requirements

Each skill's SKILL.md should include:

### Common Mistakes Section (existing pattern)

Document false positive scenarios like wp-performance-review does:

```markdown
## Common Mistakes

When performing [skill type] reviews, avoid these errors:

| Mistake | Why It's Wrong | Fix |
|---------|----------------|-----|
| Flagging X in Y context | Context makes it safe | Check for context before flagging |
```

### Version Requirements Section (new pattern)

```markdown
## WordPress Version Notes

This skill assumes WordPress [version]+. For older versions:

- Feature X: Introduced in WP [version]
- Pattern Y: Deprecated in WP [version], use Z instead
```

### Platform Considerations Section (new pattern)

```markdown
## Platform Considerations

Advice varies by hosting environment:

- **Managed WordPress (VIP, WP Engine, Pantheon):** [specific advice]
- **Self-hosted with object cache:** [specific advice]
- **Shared hosting:** [specific advice]
```

## Resources for Skill Authors

When creating skills, verify patterns against:

1. **Official WordPress Developer Handbook**
   - https://developer.wordpress.org/
   - Current as of WordPress 6.7+ (2026)

2. **Block Editor Handbook**
   - https://developer.wordpress.org/block-editor/
   - Authoritative for Gutenberg patterns

3. **WordPress Coding Standards**
   - https://developer.wordpress.org/coding-standards/
   - PHP, JavaScript, CSS standards

4. **WordPress Deprecated Functions**
   - https://github.com/WordPress/WordPress/blob/master/wp-includes/deprecated.php
   - Check before recommending functions

5. **Context7 (when available)**
   - Query for current library documentation
   - Verify API capabilities before asserting

## Sources

Research findings were verified against:

- [WordPress 6.7 Updates & 2026 Roadmap](https://www.elsner.com.au/latest-wordpress-version-updates-release-notes/)
- [Block Editor Handbook - Deprecation](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-deprecation/)
- [Gutenberg Blocks in 2026: WordPress Development in the AI Era](https://vapvarun.com/gutenberg-blocks-2026-wordpress-block-editor-ai-era/)
- [WordPress Full Site Editing - Troubleshooting Block Themes](https://fullsiteediting.com/lessons/troubleshooting-block-themes/)
- [WordPress Security Audit: Checklist & Best Practices](https://www.sentinelone.com/cybersecurity-101/cybersecurity/wordpress-security-audit/)
- [WordPress Plugin Architecture: OOP and Design Patterns](https://www.voxfor.com/wordpress-plugin-architecture-oop-design-patterns/)
- [PHP Coding Standards - WordPress Developer Handbook](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/)
- [WordPress Nonces - Common APIs Handbook](https://developer.wordpress.org/apis/security/nonces/)
- [REST API Authentication Handbook](https://developer.wordpress.org/rest-api/using-the-rest-api/authentication/)
- [Block Editor - block.json Metadata](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-metadata/)
- [Claude Code Skill for WordPress Performance Reviews](https://dev.to/n3rdh4ck3r/claude-code-skill-for-wordpress-performance-reviews-1560)
- [WordPress Plugin Check - AI Integration for False Positives](https://github.com/WordPress/plugin-check/issues/1106)
- [WordPress Block Editor (Complete Guide 2026)](https://cmsminds.com/blog/wordpress-block-editor/)

## Confidence Assessment

| Area | Level | Reason |
|------|-------|--------|
| Context-blind flags | HIGH | Existing skill already handles some cases; patterns well-documented |
| Version-specific advice | HIGH | Official WordPress docs verified; deprecations cataloged |
| Platform-specific patterns | HIGH | Existing skill mentions platform differences; VIP docs available |
| Security false positives | MEDIUM | REST API nonce behavior verified; some edge cases may exist |
| Gutenberg patterns | HIGH | Block Editor Handbook is authoritative; block.json is well-documented |
| FSE vs classic themes | HIGH | FSE is standard since WP 5.9; clear documentation |
| Plugin architecture | MEDIUM | OOP patterns are best practice but less standardized than core WP patterns |

## Summary

WordPress skill creation pitfalls cluster into three categories:

1. **Context blindness** - Flagging correct code because context wasn't checked (admin vs frontend, WP version, hosting platform)
2. **Outdated patterns** - Teaching pre-2022 WordPress patterns as current (pre-FSE themes, pre-block.json blocks)
3. **Over-zealous security** - Flagging secure code as insecure (REST API nonces, InnerBlocks sanitization)

**Prevention strategy:** For each pattern in a skill, ask:
- What WordPress version introduced/changed this?
- Does this depend on hosting environment?
- What execution context makes this acceptable/unacceptable?
- Could this create false positives in admin/CLI/cron?

The existing wp-performance-review skill demonstrates good awareness of some pitfalls (admin context checking in Common Mistakes section). New skills should follow this pattern and expand it.
