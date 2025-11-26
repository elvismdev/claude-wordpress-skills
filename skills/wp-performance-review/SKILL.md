---
name: wp-performance-review
description: WordPress performance code review and optimization analysis. Use when reviewing WordPress PHP code for performance issues, auditing themes/plugins for scalability, optimizing WP_Query usage, analyzing caching strategies, or when user mentions "performance review", "optimization audit", "slow WordPress", "scale WordPress", or "high-traffic WordPress". Detects anti-patterns in database queries, hooks, object caching, AJAX, and template loading.
---

# WordPress Performance Review Skill

Perform thorough performance code reviews for WordPress themes, plugins, and custom code.

## Code Review Workflow

1. **Identify file type** and apply relevant checks below
2. **Scan for critical patterns first** (OOM, unbounded queries, cache bypass)
3. **Check warnings** (inefficient but not catastrophic)
4. **Note optimizations** (nice-to-have improvements)
5. **Report with line numbers** using output format below

## File-Type Specific Checks

### Plugin/Theme PHP Files (`functions.php`, `plugin.php`, `*.php`)
Scan for:
- `query_posts()` → CRITICAL: Never use - breaks main query
- `posts_per_page.*-1` or `numberposts.*-1` → CRITICAL: Unbounded query
- `session_start()` → CRITICAL: Bypasses page cache
- `add_action.*init.*` or `add_action.*wp_loaded` → Check if expensive code runs every request
- `update_option` or `add_option` in non-admin context → WARNING: DB writes on page load
- `wp_remote_get` or `wp_remote_post` without caching → WARNING: Blocking HTTP

### WP_Query / Database Code
Scan for:
- Missing `posts_per_page` argument → WARNING: Defaults to blog setting
- `'meta_query'` with `'value'` comparisons → WARNING: Unindexed column scan
- `post__not_in` with large arrays → WARNING: Slow exclusion
- `LIKE '%term%'` (leading wildcard) → WARNING: Full table scan
- Missing `no_found_rows => true` when not paginating → INFO: Unnecessary count

### AJAX Handlers (`wp_ajax_*`, REST endpoints)
Scan for:
- `admin-ajax.php` usage → INFO: Consider REST API instead
- POST method for read operations → WARNING: Bypasses cache
- `setInterval` or polling patterns → CRITICAL: Self-DDoS risk
- Missing nonce verification → Security issue (not performance, but flag it)

### Template Files (`*.php` in theme)
Scan for:
- `get_template_part` in loops → WARNING: Consider caching output
- Database queries inside loops (N+1) → CRITICAL: Query multiplication
- `wp_remote_get` in templates → WARNING: Blocks rendering

### JavaScript Files
Scan for:
- `$.post(` for read operations → WARNING: Use GET for cacheability
- `setInterval.*fetch\|ajax` → CRITICAL: Polling pattern
- `import _ from 'lodash'` → WARNING: Full library import bloats bundle
- Inline `<script>` making AJAX calls on load → Check necessity

### Block Editor / Gutenberg Files (`block.json`, `*.js` in blocks/)
Scan for:
- Many `registerBlockStyle()` calls → WARNING: Each creates preview iframe
- `wp_kses_post($content)` in render callbacks → WARNING: Breaks InnerBlocks
- Static blocks without `render_callback` → INFO: Consider dynamic for maintainability

### Asset Registration (`functions.php`, `*.php`)
Scan for:
- `wp_enqueue_script` without version → INFO: Cache busting issues
- `wp_enqueue_script` without `defer`/`async` strategy → INFO: Blocks rendering
- Missing `THEME_VERSION` constant → INFO: Version management

## Search Patterns for Quick Detection

```bash
# Critical issues - scan these first
grep -rn "posts_per_page.*-1\|numberposts.*-1" .
grep -rn "query_posts\s*(" .
grep -rn "session_start\s*(" .
grep -rn "setInterval.*fetch\|setInterval.*ajax\|setInterval.*\\\$\." .

# Database writes on frontend
grep -rn "update_option\|add_option" . | grep -v "admin\|activate\|install"

# Uncached expensive functions
grep -rn "url_to_postid\|attachment_url_to_postid\|count_user_posts" .

# External HTTP without caching
grep -rn "wp_remote_get\|wp_remote_post\|file_get_contents.*http" .

# Cache bypass risks
grep -rn "setcookie\|session_start" .

# PHP code anti-patterns
grep -rn "in_array\s*(" . | grep -v "true\s*)" # Missing strict comparison
grep -rn "<<<" .  # Heredoc/nowdoc syntax
grep -rn "cache_results.*false" .

# JavaScript bundle issues
grep -rn "import.*from.*lodash['\"]" .  # Full lodash import
grep -rn "registerBlockStyle" .  # Many block styles = performance issue
```

## Platform Context

Different hosting environments require different approaches:

**Managed WordPress Hosts** (WP Engine, Pantheon, Pressable, WordPress VIP, etc.):
- Often provide object caching out of the box
- May have platform-specific helper functions (e.g., `wpcom_vip_*` on VIP)
- Check host documentation for recommended patterns

**Self-Hosted / Standard Hosting**:
- Implement object caching wrappers manually for expensive functions
- Consider Redis or Memcached plugins for persistent object cache
- More responsibility for caching layer configuration

**Shared Hosting**:
- Be extra cautious about unbounded queries and external HTTP
- Limited resources mean performance issues surface faster
- May lack persistent object cache entirely

## Quick Reference: Critical Anti-Patterns

### Database Queries
```php
// ❌ CRITICAL: Unbounded query.
'posts_per_page' => -1

// ❌ CRITICAL: Never use query_posts().
query_posts( 'cat=1' ); // Breaks pagination, conditionals.

// ❌ CRITICAL: Missing WHERE clause (falsy ID becomes 0).
$query = new WP_Query( array( 'p' => intval( $maybe_false_id ) ) );

// ❌ WARNING: LIKE with leading wildcard (full table scan).
$wpdb->get_results( "SELECT * FROM wp_posts WHERE post_title LIKE '%term%'" );

// ❌ WARNING: NOT IN queries (filter in PHP instead).
'post__not_in' => $excluded_ids
```

### Hooks & Actions
```php
// ❌ WARNING: Code runs on every request via init.
add_action( 'init', 'expensive_function' );

// ❌ CRITICAL: Database writes on every page load.
add_action( 'wp_head', 'prefix_bad_tracking' );
function prefix_bad_tracking() {
    update_option( 'last_visit', time() );
}

// ❌ WARNING: Using admin-ajax.php instead of REST API.
// Prefer: register_rest_route()
```

### PHP Code
```php
// ❌ WARNING: O(n) lookup - use isset() with associative array.
in_array( $value, $array ); // Also missing strict = true.

// ❌ WARNING: Heredoc prevents late escaping.
$html = <<<HTML
<div>$unescaped_content</div>
HTML;
```

### Caching Issues
```php
// ❌ WARNING: Uncached expensive function calls.
// These functions query the database on every call - wrap with caching:
url_to_postid( $url );
attachment_url_to_postid( $attachment_url );
count_user_posts( $user_id );
wp_oembed_get( $url );

// ✅ GOOD: Wrap with object cache (works on any host).
function prefix_cached_url_to_postid( $url ) {
    $cache_key = 'url_to_postid_' . md5( $url );
    $post_id   = wp_cache_get( $cache_key, 'url_lookups' );

    if ( false === $post_id ) {
        $post_id = url_to_postid( $url );
        wp_cache_set( $cache_key, $post_id, 'url_lookups', HOUR_IN_SECONDS );
    }

    return $post_id;
}

// ✅ GOOD: On WordPress VIP, use platform helpers instead.
// wpcom_vip_url_to_postid(), wpcom_vip_attachment_url_to_postid(), etc.

// ❌ WARNING: Large autoloaded options.
add_option( 'prefix_large_data', $data ); // Add: , '', 'no' for autoload.

// ❌ INFO: Missing wp_cache_get_multiple for batch lookups.
foreach ( $ids as $id ) {
    wp_cache_get( "key_{$id}" );
}
```

### AJAX & External Requests
```javascript
// ❌ WARNING: AJAX POST request (bypasses cache).
$.post( ajaxurl, data ); // Prefer: $.get() for read operations.

// ❌ CRITICAL: Polling pattern (self-DDoS).
setInterval( () => fetch( '/wp-json/...' ), 5000 );
```

```php
// ❌ WARNING: Synchronous external HTTP in page load.
wp_remote_get( $url ); // Cache result or move to cron.
```

### WP Cron
```php
// ❌ WARNING: WP Cron runs on page requests.
// Add to wp-config.php:
define( 'DISABLE_WP_CRON', true );
// Run via server cron: * * * * * wp cron event run --due-now
```

### Cache Bypass Issues
```php
// ❌ CRITICAL: Plugin starts PHP session on frontend (bypasses ALL page cache).
session_start(); // Check plugins for this - entire site becomes uncacheable!

// ❌ WARNING: Unique query params create cache misses.
// https://example.com/?utm_source=fb&utm_campaign=123&fbclid=abc
// Each unique URL = separate cache entry = cache miss.
// Solution: Strip marketing params at CDN/edge level.

// ❌ WARNING: Setting cookies on public pages.
setcookie( 'visitor_id', $id ); // Prevents caching for that user.
```

### External API Requests
```php
// ❌ WARNING: No timeout set (default is 5 seconds).
wp_remote_get( $url ); // Set timeout: array( 'timeout' => 2 ).

// ❌ WARNING: Missing error handling for API failures.
$response = wp_remote_get( $url );
echo $response['body']; // Check is_wp_error() first!
```

### Sitemaps & Redirects
```php
// ❌ WARNING: Generating sitemaps for deep archives (crawlers hammer these).
// Solution: Exclude old post types, cache generated sitemaps.

// ❌ CRITICAL: Redirect loops consuming CPU.
// Debug with: x-redirect-by header, wp_debug_backtrace_summary().
```

### Post Meta Queries
```php
// ❌ WARNING: Searching meta_value without index.
'meta_query' => array(
    array(
        'key'   => 'color',
        'value' => 'red',
    ),
)
// Better: Use taxonomy or encode value in meta_key name.

// ❌ WARNING: Binary meta values requiring value scan.
'meta_key'   => 'featured',
'meta_value' => 'true',
// Better: Presence of 'is_featured' key = true, absence = false.
```

## Severity Definitions

| Severity | Description |
|----------|-------------|
| **Critical** | Will cause failures at scale (OOM, 500 errors, DB locks) |
| **Warning** | Degrades performance under load |
| **Info** | Optimization opportunity |

## Output Format

Structure findings as:

```markdown
## Performance Review: [filename/component]

### Critical Issues
- **Line X**: [Issue] - [Explanation] - [Fix]

### Warnings  
- **Line X**: [Issue] - [Explanation] - [Fix]

### Recommendations
- [Optimization opportunities]

### Summary
- Total issues: X Critical, Y Warnings, Z Info
- Estimated impact: [High/Medium/Low]
```

## Deep-Dive References

Load these references based on the task:

| Task | Reference to Load |
|------|-------------------|
| Reviewing PHP code for issues | `references/anti-patterns.md` |
| Optimizing WP_Query calls | `references/wp-query-guide.md` |
| Implementing caching | `references/caching-guide.md` |
| High-traffic event prep | `references/measurement-guide.md` |

**Note**: For standard code reviews, `anti-patterns.md` contains all patterns needed. Other references provide deeper context when specifically optimizing queries, implementing caching strategies, or preparing for traffic events.
