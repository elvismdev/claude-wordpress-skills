# WooCommerce Performance Optimization Reference

This reference catalogs WooCommerce performance optimization patterns including cart fragments analysis, WC_Product_Query optimization, HPOS performance benefits, session handling, Action Scheduler for background processing, transient and object caching strategies, variation price caching, database optimization, and query monitoring patterns. Each pattern includes BAD/GOOD code pairs following WordPress PHP Coding Standards.

## Quick Reference Table

| Issue | Severity | Detection | Solution |
|-------|----------|-----------|----------|
| **Cart fragments site-wide** | CRITICAL | wc-cart-fragments.js loads on all pages | Conditional dequeuing or Mini-Cart Block |
| **WP_Query for products** | WARNING | `post_type=product` in WP_Query | Use `wc_get_products()` or `WC_Product_Query` |
| **Direct SQL for orders/products** | CRITICAL | `$wpdb->prepare()` against wp_posts for orders | Use `wc_get_orders()` / `wc_get_products()` |
| **wp_cron for bulk operations** | WARNING | `wp_schedule_event()` for order processing | Use Action Scheduler |
| **Session duration > 30 days** | WARNING | `wc_session_expiration` filter returning > 30 days | Respect 30-day cap (WC 10.1+) |
| **No product object caching** | INFO | Repeated `wc_get_product()` in loop | Use object cache or batch queries |
| **Transient bloat** | INFO | Large wp_options table with _transient_ entries | Implement transient cleanup cron |
| **Missing HPOS migration** | CRITICAL | `get_posts()` for orders on HPOS-enabled store | Migrate to wc_get_orders() |
| **N+1 order item queries** | WARNING | Individual order item loads in loop | Batch load with `get_items()` |
| **No variation price caching** | INFO | Many variations, slow price queries | Use WC 10.5+ variation price cache |

## Cart Fragments Analysis

**The most common WooCommerce performance complaint.** Cart fragments (wc-cart-fragments.js) can significantly impact site performance when loaded site-wide.

### What Are Cart Fragments?

Cart fragments are AJAX polling that updates mini-cart counts without full page reload:

1. **JavaScript:** `wc-cart-fragments.js` enqueued on page load
2. **AJAX request:** Sends request to `admin-ajax.php?action=wc_get_refreshed_fragments`
3. **Response:** Returns updated cart HTML fragments
4. **DOM update:** Updates mini-cart count/total without reload

**When it works well:**
- Cart/checkout pages where mini-cart updates are expected
- Product pages where add-to-cart is common

**When it causes problems:**
- Homepage/blog pages where cart rarely changes
- Static content pages with no commerce activity
- Loaded on every page view site-wide (pre-WC 7.8 default)

### The Performance Problem

```php
// Pre-WC 7.8 behavior (and still problematic in many themes):
// wc-cart-fragments.js loads on EVERY page
// Sends admin-ajax.php request on EVERY page view
// Even pages with no commerce functionality

// Impact:
// - Additional HTTP request per page view
// - admin-ajax.php processing for every visitor
// - Bypasses page cache (AJAX requests aren't cached)
// - Server load from constant polling
// - Slower TTFB (Time To First Byte)
```

**Severity:**
- **CRITICAL:** Cart fragments load site-wide with no conditional logic (homepage, blog, static pages all polling)
- **WARNING:** Cart fragments load on non-WC pages (blog, about, contact) without justification
- **INFO:** Cart fragments load only on WC pages (is_woocommerce(), is_cart(), is_checkout()) - acceptable

### Solution 1: Conditional Dequeuing

**Recommended for themes with traditional mini-cart:**

```php
// GOOD: Conditional dequeuing (only load where needed)
add_filter( 'woocommerce_get_script_data', 'conditional_cart_fragments', 10, 2 );
function conditional_cart_fragments( $data, $handle ) {
    if ( 'wc-cart-fragments' === $handle ) {
        // Only load on WooCommerce pages
        if ( ! is_woocommerce() && ! is_cart() && ! is_checkout() ) {
            return null; // Prevent loading
        }
    }
    return $data;
}

// BAD: Site-wide loading with no dequeue
// wc-cart-fragments.js loads on homepage, blog, contact page, etc.
// Causes performance issues on high-traffic sites
```

**Detection pattern:**

```bash
# Check if wc-cart-fragments.js loads on homepage
curl -s https://example.com | grep -c "wc-cart-fragments"
# If > 0 and homepage is not WC page, flag as CRITICAL/WARNING

# Check browser dev tools:
# Network tab → Filter JS → Look for wc-cart-fragments.js
# Network tab → Filter XHR → Look for wc_get_refreshed_fragments requests
```

### Solution 2: Mini-Cart Block (Recommended for New Themes)

**Best solution for block themes and new development:**

```php
// Mini-Cart Block has built-in performance optimizations:
// - No cart fragments JS needed
// - Server-side rendering
// - Updates via Store API (more efficient than admin-ajax.php)
// - Integrated with WooCommerce Blocks architecture

// Block markup (in block theme template):
<!-- wp:woocommerce/mini-cart /-->

// GOOD: Mini-Cart Block approach
// No fragments needed, better performance out of the box
```

**Benefits:**
- No wc-cart-fragments.js loading
- Server-side rendering (SEO friendly)
- Store API integration (more efficient)
- Block editor customization

**Migration path:**
1. Replace traditional mini-cart with Mini-Cart Block
2. Remove cart fragments handling code
3. Test cart count updates
4. Monitor performance improvement

### Solution 3: Complete Disable (No Mini-Cart)

**For sites without mini-cart functionality:**

```php
// GOOD: Complete disable when mini-cart not used
add_action( 'wp_enqueue_scripts', 'disable_cart_fragments', 100 );
function disable_cart_fragments() {
    if ( ! is_woocommerce() && ! is_cart() && ! is_checkout() ) {
        wp_dequeue_script( 'wc-cart-fragments' );
        wp_deregister_script( 'wc-cart-fragments' );
    }
}

// Or completely disable if no mini-cart exists anywhere:
add_action( 'wp_enqueue_scripts', 'completely_disable_cart_fragments', 100 );
function completely_disable_cart_fragments() {
    wp_dequeue_script( 'wc-cart-fragments' );
    wp_deregister_script( 'wc-cart-fragments' );
}
```

### Cart Fragments Benchmarking

**Measure impact:**

```php
// Before optimization:
// Homepage load time: 2.5s
// admin-ajax.php requests: 100/minute
// Server CPU: 45%

// After conditional dequeuing:
// Homepage load time: 1.8s (28% improvement)
// admin-ajax.php requests: 20/minute (80% reduction)
// Server CPU: 30% (33% reduction)

// Numbers will vary based on traffic, but pattern is consistent
```

**Cross-reference:** See wp-performance-review for general query optimization and caching strategies.

## WC_Product_Query Optimization

WooCommerce provides `wc_get_products()` and `WC_Product_Query` as optimized alternatives to `WP_Query` for product queries.

### Why Use WC_Product_Query

**Benefits:**
- Optimized product queries (uses product-specific indexes)
- Attribute lookup table support (WC 6.3+, indexed attributes for faster filtering)
- Variation price caching (WC 10.5+, cached variation prices)
- Future-proof for custom product tables (WC planning custom product tables like HPOS for orders)
- Consistent API with order queries (wc_get_orders pattern)

**Future-breaking:** WP_Query for products will break when WooCommerce migrates to custom product tables (timeline uncertain but planned).

### Basic WC_Product_Query Usage

```php
// GOOD: Use wc_get_products() for simple queries
$products = wc_get_products( array(
    'status' => 'publish',
    'limit' => 10,
    'orderby' => 'date',
    'order' => 'DESC',
    'return' => 'ids' // Return only IDs when you don't need full objects
) );

// GOOD: Use WC_Product_Query for complex queries
$query = new WC_Product_Query( array(
    'status' => 'publish',
    'limit' => 20,
    'category' => array( 'clothing' ),
    'tag' => array( 'sale' ),
    'orderby' => 'popularity',
    'return' => 'objects'
) );
$products = $query->get_products();

// WARNING: WP_Query for products (future-breaking)
$products = new WP_Query( array(
    'post_type' => 'product',
    'posts_per_page' => 10,
    'post_status' => 'publish'
) );
// Works today but will break when custom product tables migration happens
```

**Severity:** WARNING - WP_Query for products is future-breaking. Not yet critical but should migrate to WC_Product_Query.

### Performance-Optimized Parameters

```php
// GOOD: Return only IDs when you only need IDs
$product_ids = wc_get_products( array(
    'status' => 'publish',
    'limit' => 100,
    'return' => 'ids' // Much faster than loading full objects
) );

foreach ( $product_ids as $product_id ) {
    // Process ID
}

// GOOD: Limit results
$products = wc_get_products( array(
    'limit' => 10, // Always set a limit
    'orderby' => 'date',
    'order' => 'DESC'
) );

// BAD: No limit (retrieves all products)
$products = wc_get_products( array(
    'limit' => -1 // Loads ALL products, causes memory issues on large catalogs
) );

// GOOD: Use specific fields
$products = wc_get_products( array(
    'status' => 'publish',
    'type' => 'simple', // Limit to specific product type
    'category' => array( 'electronics' ), // Narrow by category
    'limit' => 20
) );
```

### Attribute Filtering (Uses Lookup Table)

```php
// GOOD: Use attribute parameter (uses indexed lookup table, WC 6.3+)
$products = wc_get_products( array(
    'attribute' => 'pa_color',
    'attribute_term' => 'blue',
    'limit' => 50
) );

// Less efficient: Meta query for attributes
$products = new WP_Query( array(
    'post_type' => 'product',
    'meta_query' => array(
        array(
            'key' => 'attribute_pa_color',
            'value' => 'blue',
            'compare' => 'LIKE'
        )
    )
) );
// Doesn't use lookup table, slower
```

### Object Caching for Repeated Loads

```php
// INFO: Missing object caching for repeated product loads
// BAD: Loading same product multiple times in loop
foreach ( $order_ids as $order_id ) {
    $order = wc_get_order( $order_id );
    foreach ( $order->get_items() as $item ) {
        $product = wc_get_product( $item->get_product_id() ); // Loads from DB each time
        // Process product
    }
}

// GOOD: Cache product objects
$product_cache = array();
foreach ( $order_ids as $order_id ) {
    $order = wc_get_order( $order_id );
    foreach ( $order->get_items() as $item ) {
        $product_id = $item->get_product_id();
        if ( ! isset( $product_cache[ $product_id ] ) ) {
            $product_cache[ $product_id ] = wc_get_product( $product_id );
        }
        $product = $product_cache[ $product_id ];
        // Process product
    }
}

// BETTER: Batch load products upfront
$product_ids = array();
foreach ( $order_ids as $order_id ) {
    $order = wc_get_order( $order_id );
    foreach ( $order->get_items() as $item ) {
        $product_ids[] = $item->get_product_id();
    }
}
$product_ids = array_unique( $product_ids );
$products = wc_get_products( array( 'include' => $product_ids, 'return' => 'objects' ) );
// Now process orders with pre-loaded products
```

### Direct SQL Anti-Pattern

```php
// CRITICAL: Direct SQL queries for products
global $wpdb;
$results = $wpdb->get_results(
    "SELECT ID FROM {$wpdb->posts} WHERE post_type = 'product' AND post_status = 'publish'"
);
// Breaks when custom product tables migration happens
// Bypasses WooCommerce CRUD layer

// GOOD: Use WC_Product_Query
$product_ids = wc_get_products( array(
    'status' => 'publish',
    'return' => 'ids',
    'limit' => -1 // Only if truly needed
) );
```

**Severity:** CRITICAL - Direct SQL for products bypasses CRUD layer and will break on custom product tables migration.

## HPOS Performance Benefits

HPOS (High-Performance Order Storage) provides significant performance improvements over post-based storage.

### Performance Benchmarks

**From WooCommerce documentation:**
- **5x faster order creation** (compared to post-based storage)
- **40x faster order filtering** (dedicated indexes vs wp_posts filtering)
- **Dedicated tables:** `wp_wc_orders` and `wp_wc_orders_meta` instead of shared `wp_posts` and `wp_postmeta`

### Why HPOS Is Faster

```php
// Legacy post-based storage:
// - Orders stored in wp_posts (shared with posts, pages, CPTs)
// - post_type = 'shop_order' filtering on every query
// - wp_postmeta table shared (no order-specific indexes)
// - JOIN operations between posts and postmeta

// HPOS storage:
// - Dedicated wp_wc_orders table (no post_type filtering needed)
// - Purpose-built indexes for common order queries
// - wp_wc_orders_meta table with order-specific indexes
// - Optimized for order operations (no post baggage)
```

### Order Query Performance

```php
// With HPOS enabled:
$orders = wc_get_orders( array(
    'status' => 'completed',
    'date_created' => '>=' . ( time() - MONTH_IN_SECONDS ),
    'limit' => 100
) );
// Fast - uses dedicated wp_wc_orders table with indexes

// Legacy (post-based):
// Same query but against wp_posts + wp_postmeta
// Slower due to post_type filtering and shared table
```

### Migration Path

```php
// Enable HPOS:
// WooCommerce → Settings → Advanced → Features
// Enable "High-Performance Order Storage"

// Compatibility mode (both post and HPOS storage):
// Allows gradual migration, data synced between both systems
// Slightly slower but ensures compatibility during migration

// Full HPOS mode:
// Disable compatibility mode after testing
// Maximum performance improvement
```

### Extension Impact

**Extensions using CRUD APIs automatically benefit:**

```php
// GOOD: CRUD-based code benefits from HPOS performance
$orders = wc_get_orders( array(
    'status' => 'processing',
    'limit' => 50
) );
// Fast with HPOS, works with legacy storage

// BAD: Direct post access doesn't benefit (and breaks on HPOS)
$orders = new WP_Query( array(
    'post_type' => 'shop_order',
    'post_status' => 'wc-processing',
    'posts_per_page' => 50
) );
// Doesn't work with HPOS at all
```

**Cross-reference:** See wc-extension-guide.md for complete HPOS compatibility patterns and migration checklist.

## Session Handling Performance

WooCommerce uses `WC_Session_Handler` to store session data in `wp_woocommerce_sessions` table.

### 30-Day Session Cap (WC 10.1+)

**WooCommerce 10.1 enforces 30-day maximum session duration:**

```php
// WARNING: Session duration exceeding 30 days
add_filter( 'wc_session_expiration', 'extend_session_duration' );
function extend_session_duration( $expiration ) {
    return 60 * DAY_IN_SECONDS; // 60 days
    // WC 10.1+ caps this at 30 days and generates warning
}

// GOOD: Respect 30-day cap
add_filter( 'wc_session_expiration', 'custom_session_duration' );
function custom_session_duration( $expiration ) {
    return 14 * DAY_IN_SECONDS; // 14 days (within 30-day limit)
}
```

**Severity:** WARNING - Session duration > 30 days is capped and generates warning in WC 10.1+.

### Session Cleanup Performance

**WooCommerce runs cleanup cron to delete expired sessions:**

```php
// Automatic cleanup via WC cron
// Deletes expired sessions from wp_woocommerce_sessions

// Performance impact:
// - Large session tables slow cleanup queries
// - Abandoned sessions accumulate over time
// - Table bloat affects all session operations

// Monitor session table size:
SELECT COUNT(*) FROM wp_woocommerce_sessions;
SELECT COUNT(*) FROM wp_woocommerce_sessions WHERE session_expiry < UNIX_TIMESTAMP();
// If many expired sessions, cleanup may be slow
```

### Long-Term Persistence Pattern

```php
// BAD: Using sessions for long-term data
add_filter( 'wc_session_expiration', 'very_long_session' );
function very_long_session( $expiration ) {
    return 365 * DAY_IN_SECONDS; // 1 year (capped at 30 days)
}
// Sessions are for temporary data only

// GOOD: Use custom database table for long-term persistence
// Example: Wishlist, saved quotes, abandoned carts for recovery
global $wpdb;
$wpdb->insert(
    $wpdb->prefix . 'custom_saved_carts',
    array(
        'user_id' => get_current_user_id(),
        'cart_data' => maybe_serialize( WC()->cart->get_cart() ),
        'created' => current_time( 'mysql' )
    )
);
// Not subject to session expiration
```

**Severity:** INFO - Custom session implementations bypassing WC_Session_Handler may not benefit from cleanup and optimizations.

## Action Scheduler for Background Processing

Action Scheduler ships with WooCommerce and provides reliable background processing superior to wp_cron().

### Why Action Scheduler Over wp_cron()

**Comparison:**

| Feature | Action Scheduler | wp_cron() |
|---------|-----------------|-----------|
| **Reliability** | Runs regardless of traffic | Traffic-dependent (no traffic = no execution) |
| **Retry logic** | Automatic retry on failure | No retry, failed cron lost |
| **Admin UI** | WooCommerce → Status → Scheduled Actions | No UI, must use code/plugins |
| **Queue management** | View pending, in-progress, complete | No visibility |
| **Traceability** | Logs all executions | No built-in logging |
| **Scalability** | Queue processing, throttling | All crons at once |

### Common WooCommerce Use Cases

```php
// Use Action Scheduler for:
// - Order processing (bulk status changes, exports)
// - Inventory sync (third-party systems)
// - Report generation (large datasets)
// - Bulk email sending
// - Webhook processing (async after verification)
// - Data cleanup (old sessions, transients)

// WARNING: wp_cron for bulk WC operations
wp_schedule_event( time(), 'hourly', 'bulk_order_export' );
// Unreliable on low-traffic sites
// No retry if fails
// No way to monitor progress

// GOOD: Action Scheduler for bulk operations
as_schedule_recurring_action( time(), HOUR_IN_SECONDS, 'bulk_order_export', array(), 'wc-exports' );
// Reliable execution
// Automatic retry
// Admin UI for monitoring
```

### Immediate Background Processing

```php
// GOOD: Async processing for webhooks
function handle_webhook( $payload ) {
    // Verify webhook signature first
    // ...

    // Process async via Action Scheduler
    as_enqueue_async_action(
        'process_webhook_data',
        array( 'payload' => $payload ),
        'wc-webhooks'
    );

    // Respond immediately (< 5 seconds)
    http_response_code( 200 );
    exit;
}

add_action( 'process_webhook_data', 'do_process_webhook', 10, 1 );
function do_process_webhook( $payload ) {
    // Heavy processing here (can take minutes)
    // Update orders, sync inventory, etc.
}

// BAD: Synchronous webhook processing
function handle_webhook( $payload ) {
    // Heavy processing (may timeout)
    foreach ( $orders as $order ) {
        // Process each order
    }
    // Webhook sender times out, retries, causes duplicates
}
```

### Batch Processing Pattern

```php
// GOOD: Batch processing with recursion
function start_bulk_export() {
    as_enqueue_async_action(
        'process_export_batch',
        array( 'offset' => 0 ),
        'wc-exports'
    );
}

add_action( 'process_export_batch', 'process_batch', 10, 1 );
function process_batch( $offset ) {
    $batch_size = 100;
    $orders = wc_get_orders( array(
        'limit' => $batch_size,
        'offset' => $offset,
        'return' => 'ids'
    ) );

    foreach ( $orders as $order_id ) {
        // Export order
    }

    // Schedule next batch if more orders exist
    if ( count( $orders ) === $batch_size ) {
        as_enqueue_async_action(
            'process_export_batch',
            array( 'offset' => $offset + $batch_size ),
            'wc-exports'
        );
    }
}

// BAD: Process all orders at once
add_action( 'bulk_export', 'export_all_orders' );
function export_all_orders() {
    $orders = wc_get_orders( array( 'limit' => -1 ) ); // All orders
    foreach ( $orders as $order ) {
        // Export order
    }
    // Timeout on large datasets
    // High memory usage
    // No progress tracking
}
```

### Monitoring Actions

**Admin UI:**
```
WooCommerce → Status → Scheduled Actions
```

**View:**
- Pending actions (queued, not yet processed)
- In-progress actions (currently running)
- Complete actions (successfully finished)
- Failed actions (errors, with retry status)

**Query actions programmatically:**

```php
// Check if action is scheduled
if ( as_next_scheduled_action( 'my_action' ) ) {
    // Already scheduled
}

// Get all pending actions for group
$actions = as_get_scheduled_actions( array(
    'group' => 'wc-exports',
    'status' => 'pending'
) );
```

## Transient and Object Caching Strategies

Caching reduces repeated queries for computed or external data.

### When to Cache

**Good candidates for caching:**
- Repeated product loads in loops
- API responses (shipping rates, payment gateway tokens)
- Computed values (cart totals, shipping calculations)
- External service calls (exchange rates, geocoding)

**Bad candidates for caching:**
- Data that changes frequently (stock levels)
- User-specific data (unless keyed by user)
- Large datasets (cache storage limits)

### Transient API (DB-Backed Cache)

```php
// GOOD: Cache shipping rates
function get_shipping_rates( $package ) {
    $cache_key = 'shipping_rates_' . md5( serialize( $package ) );
    $rates = get_transient( $cache_key );

    if ( false === $rates ) {
        // Calculate rates (expensive API call)
        $rates = calculate_shipping_rates( $package );

        // Cache for 1 hour
        set_transient( $cache_key, $rates, HOUR_IN_SECONDS );
    }

    return $rates;
}

// BAD: Computing shipping rate on every page load
function get_shipping_rates( $package ) {
    // Expensive API call every time
    return calculate_shipping_rates( $package );
}
```

### Object Cache (Persistent Cache if Available)

```php
// GOOD: Object cache for repeated product loads
function get_products_by_category( $category_id ) {
    $cache_key = 'products_cat_' . $category_id;
    $products = wp_cache_get( $cache_key, 'custom_products' );

    if ( false === $products ) {
        $products = wc_get_products( array(
            'category' => array( $category_id ),
            'limit' => 50,
            'return' => 'ids'
        ) );

        // Cache for current request (or persistent if object cache available)
        wp_cache_set( $cache_key, $products, 'custom_products', HOUR_IN_SECONDS );
    }

    return $products;
}
```

### WooCommerce-Specific Caching

```php
// Product transients (invalidated on product save)
function get_product_data( $product_id ) {
    $cache_key = 'product_data_' . $product_id;
    $data = get_transient( $cache_key );

    if ( false === $data ) {
        $product = wc_get_product( $product_id );
        $data = array(
            'price' => $product->get_price(),
            'stock' => $product->get_stock_quantity(),
            'categories' => $product->get_category_ids()
        );

        set_transient( $cache_key, $data, DAY_IN_SECONDS );
    }

    return $data;
}

// Invalidate cache on product save
add_action( 'woocommerce_update_product', 'clear_product_cache', 10, 1 );
function clear_product_cache( $product_id ) {
    $cache_key = 'product_data_' . $product_id;
    delete_transient( $cache_key );
}
```

### Transient Cleanup

```php
// INFO: Transient bloat
// wp_options table accumulates _transient_ entries
// Expired transients may not be cleaned up automatically

// Monitor transient count:
SELECT COUNT(*) FROM wp_options WHERE option_name LIKE '_transient_%';

// Implement cleanup cron:
add_action( 'custom_cleanup_transients', 'cleanup_expired_transients' );
function cleanup_expired_transients() {
    global $wpdb;
    $wpdb->query(
        "DELETE FROM {$wpdb->options}
         WHERE option_name LIKE '_transient_timeout_%'
         AND option_value < UNIX_TIMESTAMP()"
    );
}

// Schedule cleanup (use Action Scheduler)
if ( ! as_next_scheduled_action( 'custom_cleanup_transients' ) ) {
    as_schedule_recurring_action( time(), DAY_IN_SECONDS, 'custom_cleanup_transients', array(), 'wc-cleanup' );
}
```

## Variation Price Caching (WC 10.5+)

WooCommerce 10.5 introduces variation price caching for products with many variations.

### Problem: Variable Product Price Queries

```php
// Variable products with many variations cause slow price queries:
// - get_variation_price( 'min' ) queries all variation prices
// - get_variation_price( 'max' ) queries all variation prices
// - Each query loads all variations to calculate min/max

// Example: Product with 100 variations
$product = wc_get_product( $variable_product_id );
$min_price = $product->get_variation_price( 'min' ); // Loads 100 variations
$max_price = $product->get_variation_price( 'max' ); // Loads 100 variations again
```

### WC 10.5 Improvement

```php
// WC 10.5+ caches variation prices per product
// Cached data includes:
// - Minimum variation price
// - Maximum variation price
// - Invalidated on variation save

// No code changes needed - automatic optimization
$product = wc_get_product( $variable_product_id );
$min_price = $product->get_variation_price( 'min' ); // Uses cache
$max_price = $product->get_variation_price( 'max' ); // Uses cache

// Cache invalidation automatic on variation update
```

### Product Attribute Lookup Table (WC 6.3+)

```php
// Product attribute lookup table provides indexed attributes
// Faster product filtering by attributes

// GOOD: Filter products by attribute (uses lookup table)
$products = wc_get_products( array(
    'attribute' => 'pa_color',
    'attribute_term' => 'blue',
    'limit' => 50
) );
// Fast - uses indexed lookup table

// Less efficient: Meta query approach
$products = new WP_Query( array(
    'post_type' => 'product',
    'meta_query' => array(
        array(
            'key' => 'attribute_pa_color',
            'value' => 'blue'
        )
    )
) );
// Doesn't use lookup table
```

**Severity:** INFO - Awareness of variation price caching in WC 10.5+. No action needed unless building custom variation price queries.

## Database Optimization

### wp_postmeta Index Recommendations

```php
// For WooCommerce stores, wp_postmeta gets heavy use
// Product meta, order meta (if not using HPOS)

// Recommended indexes:
// - meta_key (most queries filter by key)
// - meta_value (for exact value lookups)
// - post_id + meta_key (common JOIN pattern)

// Check existing indexes:
SHOW INDEXES FROM wp_postmeta;

// Add index if missing (via migration/setup plugin):
ALTER TABLE wp_postmeta ADD INDEX meta_key_value (meta_key, meta_value(20));
```

### Autoloaded Options Audit

```php
// WooCommerce adds many autoloaded options
// wp_options with autoload='yes' are loaded on EVERY request

// Audit autoloaded options:
SELECT option_name, LENGTH(option_value) AS value_length
FROM wp_options
WHERE autoload = 'yes'
ORDER BY value_length DESC;

// Common WooCommerce autoloaded options:
// - woocommerce_* settings
// - _transient_* (should be cleaned up)
// - action_scheduler_* (consider disabling autoload for large datasets)

// Disable autoload for specific options if not needed on every request:
update_option( 'large_option_name', $value, false ); // false = don't autoload
```

**Severity:** INFO - Large autoloaded options slow every request. Audit and disable autoload where possible.

### HPOS-Compatible Order Count

```php
// BAD: wp_count_posts() for orders (doesn't work with HPOS)
$count = wp_count_posts( 'shop_order' );
$completed = $count->{'wc-completed'};

// GOOD: wc_orders_count() (HPOS-compatible)
$count = wc_orders_count( 'completed' );
// Works with both HPOS and legacy storage

// For custom order counts:
$orders = wc_get_orders( array(
    'status' => 'processing',
    'limit' => -1,
    'return' => 'ids'
) );
$count = count( $orders );
// Or use COUNT query if available in WC API
```

### Direct Database Query Anti-Pattern

```php
// CRITICAL: COUNT(*) on wp_posts for orders
global $wpdb;
$count = $wpdb->get_var(
    "SELECT COUNT(*) FROM {$wpdb->posts}
     WHERE post_type = 'shop_order'
     AND post_status = 'wc-completed'"
);
// Breaks with HPOS
// Bypasses WooCommerce CRUD

// GOOD: Use wc_orders_count()
$count = wc_orders_count( 'completed' );
```

**Severity:** CRITICAL - Direct database queries for orders break on HPOS.

## Query Monitoring Patterns

### Tools for Monitoring

**Query Monitor plugin:**
- Shows all database queries per request
- Highlights slow queries (> 0.05s)
- Shows duplicate queries (N+1 patterns)
- Displays query backtrace (where query originated)

**WooCommerce logging:**

```php
// Enable WC logging
$logger = wc_get_logger();
$logger->debug( 'Query executed: ' . $query, array( 'source' => 'custom-extension' ) );

// View logs: WooCommerce → Status → Logs
```

### Common Slow Queries in WC

**Order listing:**
```php
// Slow: Loading all orders with full objects
$orders = wc_get_orders( array( 'limit' => -1 ) );

// Fast: Limit + pagination
$orders = wc_get_orders( array(
    'limit' => 50,
    'page' => 1
) );
```

**Product filtering:**
```php
// Slow: Meta query on meta_value
$products = new WP_Query( array(
    'post_type' => 'product',
    'meta_query' => array(
        array(
            'key' => 'custom_field',
            'value' => 'some_value'
        )
    )
) );

// Faster: Taxonomy or indexed field
$products = wc_get_products( array(
    'category' => array( 'electronics' )
) );
```

**Session table scans:**
```php
// Monitor session table size
SELECT COUNT(*) FROM wp_woocommerce_sessions;

// If large, cleanup expired sessions:
DELETE FROM wp_woocommerce_sessions WHERE session_expiry < UNIX_TIMESTAMP();
```

### N+1 Pattern in WooCommerce

```php
// WARNING: N+1 order items loading
foreach ( $order_ids as $order_id ) {
    $order = wc_get_order( $order_id );
    foreach ( $order->get_items() as $item ) {
        // Individual item load (N queries)
    }
}

// GOOD: Batch load items
$orders = wc_get_orders( array( 'include' => $order_ids ) );
foreach ( $orders as $order ) {
    $items = $order->get_items(); // Loaded with order (no extra queries)
}
```

**Cross-reference:** See wp-performance-review for general query optimization patterns, N+1 detection, and caching strategies.

## Cross-References

This reference cross-references other skills for related patterns:

- **wp-performance-review:** General query optimization, N+1 pattern detection, WP_Query optimization, caching strategies, transient usage
- **wp-plugin-development:** Action Scheduler vs wp_cron, background processing patterns, Settings API for performance options
- **wp-theme-development:** Template performance, conditional asset loading
- **wc-extension-guide.md:** HPOS compatibility patterns, CRUD API usage, Action Scheduler implementation

---

*WooCommerce 8.2+ with HPOS | WordPress 6.x+ | PHP 7.4+*
