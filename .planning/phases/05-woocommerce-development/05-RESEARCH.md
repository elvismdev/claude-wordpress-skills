# Phase 5: WooCommerce Development - Research

**Researched:** 2026-02-06
**Domain:** WooCommerce extension development, HPOS compatibility, payment gateway development
**Confidence:** HIGH

## Summary

WooCommerce extension development in 2026 centers on **High-Performance Order Storage (HPOS)** compatibility as the critical architectural shift. Since WooCommerce 8.2 (October 2023), HPOS is the default for new stores, replacing WordPress post tables with dedicated order tables. Extensions must use CRUD APIs (wc_get_orders(), WC_Order methods) instead of direct post access to work on modern WooCommerce installations.

The WooCommerce ecosystem provides mature APIs for extension development: **WC_Payment_Gateway** for payment processing, **WC_Shipping_Method** for shipping calculations, **WC_Product** hierarchy for custom product types, and extensive hooks for cart/checkout customization. The standard stack emphasizes CRUD-first data access, Action Scheduler for background processing, and hooks-over-template-overrides for maintainability.

Key performance concerns remain **cart fragments** (wc-cart-fragments.js loading site-wide is still a major issue in 2026, despite improvements in WC 7.8) and query optimization (WP_Query for products is deprecated in favor of wc_get_products()/WC_Product_Query). Payment gateway development requires code-level security patterns (never store/log raw card data, use tokenization) but is explicitly NOT a PCI compliance audit—that's infrastructure-level.

**Primary recommendation:** Build HPOS-first extensions using WooCommerce CRUD APIs exclusively, declare compatibility with FeaturesUtil::declare_compatibility(), optimize cart fragments via conditional dequeuing or Mini-Cart Block migration, and follow hooks-first philosophy for template customization.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Detection scope & depth:**
- **Deep coverage (dedicated sections in SKILL.md):**
  - HPOS compatibility — declare_compatibility() calls, CRUD-only order access, wc_get_orders()/wc_get_order() vs get_posts(), meta operations via WC_Order methods + save(), compatibility mode awareness
  - WooCommerce CRUD pattern — WC_Data base class, data stores (WC_Object_Data_Store_Interface), wc_get_products()/WC_Product_Query vs WP_Query for products, custom data store registration via woocommerce_data_stores filter
  - Payment gateway development — WC_Payment_Gateway class, process_payment() flow, refund handling (process_refund()), saved payment methods (WC_Payment_Token), card data anti-patterns (never store/log raw card data, token-based storage only)
  - WooCommerce hooks catalog — lifecycle hooks (woocommerce_before_cart, woocommerce_checkout_process, woocommerce_thankyou, woocommerce_order_status_changed), product data hooks (woocommerce_product_options_*, woocommerce_process_product_meta), checkout field hooks, email hooks
  - Order handling — WC_Order CRUD, status transitions (wc_get_order_statuses()), custom order statuses, order item manipulation, order notes
  - Cart operations — WC()->cart methods, cart item data (woocommerce_add_cart_item_data), cart fees, cart validation hooks
  - Custom product types — register_product_type(), extending WC_Product, product data stores, product type class hierarchy
  - Shipping methods — WC_Shipping_Method class, calculate_shipping(), shipping zones, flat rate/free shipping customization
  - Cart fragments performance — detect site-wide wc-cart-fragments.js loading, recommend Mini-Cart Block or conditional dequeuing, flag as WARNING/CRITICAL based on scope
  - Action Scheduler patterns — detect wp_cron() for heavy WC tasks, recommend as_schedule_single_action()/as_enqueue_async_action(), flag synchronous bulk processing
  - Webhook security — signature verification (X-WC-Webhook-Signature), async processing, duplicate delivery handling (X-WC-Webhook-Delivery-ID)
  - Template override anti-patterns — detect overrides with deleted action hooks (CRITICAL), outdated template versions (WARNING), recommend hooks-first approach
- **Surface-level coverage (scan patterns + brief guidance):**
  - WooCommerce Blocks checkout integration — Store API, block checkout extension points, compatibility declarations for block cart/checkout, Additional Checkout Fields API
  - WooCommerce REST API extensions — v3 endpoints, custom endpoint registration, authentication patterns (Basic Auth, OAuth 1.0a), HTTPS enforcement
  - Coupon/discount validation — custom coupon types, validation hooks (woocommerce_coupon_is_valid)
  - Session handling — WC_Session_Handler, 30-day cap (WC 10.1+), avoid custom session implementations
  - Email customization — WC_Email class extension, template overrides for emails
  - Product variations performance — attribute lookup table, variation price caching, dynamic dropdown limits
  - WooCommerce Blocks compatibility with template overrides — note incompatibility, recommend block integration for new stores
  - Experimental product object caching (WC 10.5+) — awareness only
- **Context-aware detection:** Distinguish between:
  - WooCommerce extension/plugin (standalone plugin extending WC functionality)
  - WooCommerce theme integration (theme with woocommerce/ template overrides)
  - Payment gateway (WC_Payment_Gateway subclass — heightened security review)
  - Shipping method (WC_Shipping_Method subclass)
  - Custom product type (WC_Product subclass with data store)
  - WooCommerce Blocks integration (block-based checkout/cart extension)

**HPOS handling strategy:**
- **HPOS is default for new stores since WC 8.2** — all new extension code MUST use CRUD APIs
- **Missing HPOS compatibility declaration = CRITICAL** for new extensions (declare_compatibility() in before_woocommerce_init hook)
- **Direct post access for orders (get_posts with shop_order) = CRITICAL** — will break on HPOS-enabled stores
- **WP_Query for products = WARNING** (custom product tables migration planned, not yet enforced but future-breaking)
- **Direct database queries for orders/products = CRITICAL** — bypasses both HPOS and CRUD
- Same future-proofing pattern: flag code that works today but will break in upcoming WC releases

**Payment security boundary:**
- **Code-level anti-patterns only** — this is NOT a PCI compliance audit
- **CRITICAL violations:** Storing raw card numbers/CVV in database, logging card data (including in error_log/debug.log), transmitting card data over HTTP, storing full card data in sessions
- **WARNING violations:** Hardcoded API credentials, missing HTTPS check before payment processing, no webhook signature verification
- **Explicit non-scope:** We do not assess PCI SAQ level, hosting compliance, or network security — note that PCI compliance is the store owner's responsibility and recommend professional PCI assessment
- Pattern: detect what code does wrong, not what infrastructure is missing

**Performance detection patterns:**
- **Cart fragments (wc-cart-fragments.js):**
  - Still a major performance problem in 2026
  - CRITICAL when loaded site-wide with no conditional dequeuing
  - WARNING when loaded on non-WC pages without justification
  - Recommend: Mini-Cart Block (built-in optimization), or conditional dequeuing to cart/checkout/product pages only
- **Product queries:**
  - CRITICAL: Direct SQL queries against wp_posts for products
  - WARNING: WP_Query with post_type=product (use wc_get_products()/WC_Product_Query)
  - INFO: Missing object caching for repeated product loads
- **Action Scheduler:**
  - WARNING: wp_cron() for bulk WC operations (order processing, inventory sync, report generation)
  - Recommend: Action Scheduler (as_schedule_single_action, as_enqueue_async_action) — ships with WC
  - INFO: Synchronous processing of operations that should be async
- **Session handling:**
  - WARNING: Custom session filters exceeding 30-day cap (WC 10.1+ enforces limit)
  - INFO: Custom session implementations bypassing WC_Session_Handler

**Template override vs hooks guidance:**
- **Hooks are strongly preferred over template overrides** — this is a key teaching point
- **Template overrides with deleted action hooks = CRITICAL** — breaks plugin integration, other extensions can't hook in
- **Outdated template overrides (version mismatch) = WARNING** — may miss WC updates, cause display issues
- **Template override where a hook exists = INFO** — suggest using hook instead
- **Template version tracking:** Detect @version comments in overridden templates, compare against WC version expectations
- WooCommerce Blocks pages bypass template overrides entirely — note this as awareness item

**Cross-references to existing skills:**
- **wp-security-review:** Payment data handling (never store raw cards), REST API authentication, AJAX nonce verification for add-to-cart, capability checks in order management
- **wp-plugin-development:** WC as a plugin extension (proper dependency checking, activation/deactivation), hooks architecture, REST API endpoint registration, Settings API for WC settings pages
- **wp-block-development:** WooCommerce Blocks (product grid block, cart block, checkout block), block-based checkout extension points, Store API integration
- **wp-theme-development:** WooCommerce template overrides in themes (woocommerce/ directory), shop page customization, product display templates, WC Blocks compatibility with block themes
- **wp-performance-review:** Cart fragments optimization, WC_Product_Query vs WP_Query, session table performance, Action Scheduler vs wp_cron, transient usage for product data caching
- Same cross-reference pattern as prior skills: brief mention + pointer, don't duplicate content

**Reference doc structure:**
- Follow established cookbook format: quick reference table + detailed patterns + edge cases + WC nuances
- **Three reference docs:**
  - `wc-extension-guide.md` — Complete extension development patterns: HPOS compatibility (declare_compatibility, CRUD-only access, migration checklist), payment gateway development (WC_Payment_Gateway lifecycle, process_payment flow, refund handling, saved payment methods, security anti-patterns), shipping method development (WC_Shipping_Method, calculate_shipping, zones), custom product types (register_product_type, data stores), order handling (WC_Order CRUD, status transitions, custom statuses), cart operations (cart item data, fees, validation), REST API extensions, WooCommerce Blocks checkout integration, Action Scheduler patterns, webhook security
  - `wc-template-guide.md` — Template override patterns: complete template hierarchy (archive-product, content-product, single-product, cart, checkout, myaccount), proper override procedure (child theme woocommerce/ directory, version tracking), hooks-first philosophy (when to use hooks vs overrides, preserving action hooks in overrides), shop page customization, product display patterns, cart/checkout theming, WooCommerce Blocks compatibility notes, email template customization
  - `wc-performance-guide.md` — Performance optimization patterns: cart fragments analysis (problem, solutions, Mini-Cart Block alternative), WC_Product_Query optimization, HPOS performance benefits, session handling (30-day cap, cleanup), Action Scheduler for background processing, transient and object caching strategies, variation price caching, product attribute lookup table, database optimization (wp_postmeta index recommendations), query monitoring patterns
- BAD/GOOD pairs throughout in PHP, WordPress PHP Coding Standards

**Finding output style:**
- Mirror all prior skills exactly:
  - Group findings by FILE (PHP files organized by actual file path)
  - Each finding: line number, severity, issue description, explanation, BAD/GOOD code pair
  - Summary at end with total counts by severity
- **Severity mapping for WooCommerce:**
  - CRITICAL: Extension will break on modern WC stores or has security vulnerabilities (missing HPOS compatibility, direct DB access for orders/products, raw card data storage, deleted hooks in template overrides, no webhook signature verification)
  - WARNING: Extension works but has quality/compatibility/performance issues (WP_Query for products, cart fragments site-wide, outdated template overrides, hardcoded API credentials, wp_cron for bulk operations, session duration exceeding 30 days)
  - INFO: Best practice improvements (hooks instead of template overrides, Action Scheduler instead of wp_cron for light tasks, object caching for product loads, block checkout integration opportunity)

**Quick scan vs full review boundary:**
- `/wp-woo` (quick scan): Grep-based pattern detection — scan for get_posts with shop_order, direct $wpdb queries against WC tables, missing declare_compatibility, raw card data patterns, cart-fragments loading without dequeue, template overrides without do_action, WP_Query with post_type=product
- `/wp-woo-review` (full review): Complete skill workflow — WC context detection (extension/theme/gateway/shipping), HPOS compatibility audit, payment security review, hook usage analysis, template override assessment, performance pattern detection, produces structured report
- Same boundary pattern as all prior skills

**Infrastructure updates (Plan 3):**
- README.md: Document all 6 skills with status, triggers, commands (INF-01)
- marketplace.json: Register all new skills (INF-02)
- plugin.json: Version bump (INF-03)
- CHANGELOG.md: Document all new skills added (INF-04)
- These complete the plugin build-out to v1.0

### Claude's Discretion

- Exact grep patterns for quick detection (WC patterns need different regex than generic WP)
- Depth of WooCommerce Blocks checkout integration coverage (enough to detect missing compatibility, not a Blocks development tutorial)
- How to handle WooCommerce-specific false positives (e.g., get_posts() for non-order post types that happen to be in WC extensions)
- Whether to include WooCommerce Admin/Analytics patterns or limit to storefront extension development
- Coupon validation depth (enough to detect missing validation hooks, not a coupon system tutorial)
- Email template customization depth (surface-level pattern detection only)

### Deferred Ideas (OUT OF SCOPE)

None — all WooCommerce development concerns fit within phase scope. Infrastructure updates (README, marketplace.json, plugin.json, CHANGELOG) are explicitly scoped in this phase per the roadmap.

</user_constraints>

## Standard Stack

The established libraries/tools for WooCommerce extension development:

### Core (Shipped with WooCommerce)
| Library/API | Version | Purpose | Why Standard |
|-------------|---------|---------|--------------|
| WooCommerce | 8.2+ (10.5 current) | E-commerce platform | Target version with HPOS default |
| HPOS (Custom Order Tables) | 8.2+ | Order storage | Default since Oct 2023, replaces post tables |
| Action Scheduler | Built-in | Background processing | Ships with WC, replaces wp_cron for heavy tasks |
| WooCommerce CRUD | 3.0+ | Data access layer | WC_Data, WC_Order, WC_Product classes for all data operations |
| WooCommerce Blocks | 8.0+ | Block-based cart/checkout | Modern store interface, bypasses template overrides |
| Store API | Built-in | REST API for Blocks | Extension point for block checkout customization |

### Extension APIs
| API | Version | Purpose | When to Use |
|-----|---------|---------|-------------|
| WC_Payment_Gateway | Core API | Payment processing | Custom payment methods, gateway integrations |
| WC_Shipping_Method | Core API | Shipping calculations | Custom shipping methods, rate tables |
| WC_Product hierarchy | Core API | Custom product types | Subscriptions, bookings, memberships |
| WC_Email | Core API | Email notifications | Custom order emails, triggers |
| WooCommerce REST API v3 | Built-in | External integrations | Headless commerce, mobile apps |

### Performance Tools
| Tool | Purpose | When to Use |
|------|---------|-------------|
| Mini-Cart Block | Cart fragment replacement | New themes, block-based checkout |
| Product Object Caching (10.5+) | Cache product instances | Experimental opt-in for high-traffic stores |
| Variation Price Caching (10.5+) | Cache variation prices | Products with many variations |
| Attribute Lookup Table (6.3+) | Indexed product attributes | Product filtering by attributes |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| HPOS | WordPress post tables | Legacy only—breaks on new stores since WC 8.2 |
| Action Scheduler | wp_cron() | wp_cron unreliable, traffic-dependent, no retry logic |
| wc_get_orders() | WP_Query/get_posts() | Direct post access breaks HPOS compatibility |
| Mini-Cart Block | Cart fragments | Fragments cause performance issues, load site-wide |
| WooCommerce Blocks | Classic templates | Blocks are future direction, better performance |

**Installation:**
WooCommerce ships with all core APIs. Extensions install as WordPress plugins:
```bash
# WooCommerce itself
composer require woocommerce/woocommerce

# Extensions are WordPress plugins
# No separate installation - use WooCommerce APIs
```

## Architecture Patterns

### Recommended Extension Structure
```
my-wc-extension/
├── my-wc-extension.php          # Main plugin file with HPOS declaration
├── includes/
│   ├── class-gateway.php        # WC_Payment_Gateway extension
│   ├── class-shipping.php       # WC_Shipping_Method extension
│   ├── class-product-type.php   # WC_Product extension
│   └── class-order-handler.php  # WC_Order CRUD operations
├── templates/                   # Template overrides (if needed)
│   └── emails/
│       └── custom-email.php
└── assets/
    ├── js/
    └── css/
```

### Pattern 1: HPOS Compatibility Declaration (MANDATORY)
**What:** All extensions must declare HPOS compatibility status in main plugin file
**When to use:** Every WooCommerce extension created since WC 8.2 (Oct 2023)
**Example:**
```php
// Source: https://developer.woocommerce.com/docs/features/high-performance-order-storage/recipe-book/

// GOOD: Declare compatibility in before_woocommerce_init hook
add_action( 'before_woocommerce_init', function() {
    if ( class_exists( \Automattic\WooCommerce\Utilities\FeaturesUtil::class ) ) {
        \Automattic\WooCommerce\Utilities\FeaturesUtil::declare_compatibility(
            'custom_order_tables',
            __FILE__,
            true // true = compatible, false = incompatible
        );
    }
} );

// BAD: Missing HPOS declaration
// Extension silently breaks on HPOS-enabled stores
```

### Pattern 2: Order Data Access via CRUD (HPOS-Required)
**What:** Use wc_get_order() and WC_Order methods, never get_posts() or $wpdb
**When to use:** All order data access in extensions
**Example:**
```php
// Source: https://developer.woocommerce.com/docs/features/high-performance-order-storage/recipe-book/

// GOOD: HPOS-compatible order access
$order = wc_get_order( $order_id );
$order->update_meta_data( 'custom_field', $value );
$order->save(); // Single save after all changes

// BAD: Direct post access (breaks HPOS)
$order_post = get_post( $order_id );
update_post_meta( $order_id, 'custom_field', $value );

// BAD: WP_Query for orders (breaks HPOS)
$orders = new WP_Query( array(
    'post_type' => 'shop_order',
    'post_status' => 'wc-completed'
) );

// GOOD: Use wc_get_orders() instead
$orders = wc_get_orders( array(
    'status' => 'completed',
    'limit' => 10
) );
```

### Pattern 3: Product Data Access via CRUD
**What:** Use wc_get_product() and WC_Product_Query, not WP_Query
**When to use:** All product data access in extensions
**Example:**
```php
// Source: https://developer.woocommerce.com/docs/best-practices/data-management/crud-objects/

// GOOD: WooCommerce CRUD API
$product = wc_get_product( $product_id );
$product->set_price( 19.99 );
$product->save();

// GOOD: WC_Product_Query for product lists
$query = new WC_Product_Query( array(
    'status' => 'publish',
    'limit' => 10,
    'category' => array( 'clothing' ),
    'orderby' => 'date',
    'order' => 'DESC'
) );
$products = $query->get_products();

// WARNING: WP_Query for products (future-breaking)
// Custom product tables migration planned
$products = new WP_Query( array(
    'post_type' => 'product',
    'posts_per_page' => 10
) );
```

### Pattern 4: Payment Gateway Implementation
**What:** Extend WC_Payment_Gateway with security-first patterns
**When to use:** Custom payment methods, third-party gateway integrations
**Example:**
```php
// Source: https://developer.woocommerce.com/docs/features/payments/payment-gateway-api

class WC_Gateway_Custom extends WC_Payment_Gateway {

    public function __construct() {
        $this->id = 'custom_gateway';
        $this->has_fields = true;
        $this->method_title = __( 'Custom Gateway', 'text-domain' );
        $this->method_description = __( 'Description', 'text-domain' );

        $this->init_form_fields();
        $this->init_settings();

        add_action( 'woocommerce_update_options_payment_gateways_' . $this->id,
            array( $this, 'process_admin_options' ) );
    }

    public function process_payment( $order_id ) {
        $order = wc_get_order( $order_id );

        // GOOD: Never store raw card data
        // Use tokenization or redirect to hosted payment page

        // CRITICAL: Always check HTTPS in production
        if ( ! is_ssl() && 'yes' !== $this->testmode ) {
            wc_add_notice( __( 'SSL required', 'text-domain' ), 'error' );
            return array( 'result' => 'failure' );
        }

        // Process payment via API (pseudocode)
        $result = $this->api_charge( $order );

        if ( $result->success ) {
            // GOOD: Use payment_complete() not manual status change
            $order->payment_complete( $result->transaction_id );

            return array(
                'result' => 'success',
                'redirect' => $this->get_return_url( $order )
            );
        }

        return array( 'result' => 'failure' );
    }

    // CRITICAL: Never log card data
    // BAD: error_log( 'Card: ' . $_POST['card_number'] );
}
```

### Pattern 5: Action Scheduler for Background Processing
**What:** Use Action Scheduler (ships with WC) instead of wp_cron() for heavy tasks
**When to use:** Bulk order processing, inventory sync, report generation
**Example:**
```php
// Source: https://actionscheduler.org/

// GOOD: Action Scheduler for bulk operations
function schedule_order_export() {
    as_enqueue_async_action(
        'process_order_export',
        array( 'batch_id' => 123 ),
        'wc-exports'
    );
}

add_action( 'process_order_export', 'do_order_export', 10, 1 );
function do_order_export( $batch_id ) {
    // Process in batches to avoid timeout
    $orders = wc_get_orders( array(
        'limit' => 100,
        'batch' => $batch_id
    ) );

    foreach ( $orders as $order ) {
        // Export logic
    }

    // Schedule next batch if needed
    if ( count( $orders ) === 100 ) {
        as_enqueue_async_action(
            'process_order_export',
            array( 'batch_id' => $batch_id + 1 ),
            'wc-exports'
        );
    }
}

// WARNING: wp_cron for heavy WC tasks
// Unreliable, traffic-dependent, no retry on failure
wp_schedule_event( time(), 'hourly', 'process_order_export' );
```

### Pattern 6: Cart Fragments Optimization
**What:** Conditionally dequeue wc-cart-fragments.js or migrate to Mini-Cart Block
**When to use:** Themes with mini-cart, performance optimization
**Example:**
```php
// Source: https://developer.woocommerce.com/2023/06/16/best-practices-for-the-use-of-the-cart-fragments-api/

// GOOD: Conditional dequeuing (only load where needed)
add_filter( 'woocommerce_get_script_data', function( $data, $handle ) {
    if ( 'wc-cart-fragments' === $handle ) {
        // Only load on WC pages
        if ( ! is_woocommerce() && ! is_cart() && ! is_checkout() ) {
            return null;
        }
    }
    return $data;
}, 10, 2 );

// BETTER: Migrate to Mini-Cart Block (no fragments needed)
// Mini-Cart Block has built-in performance optimizations

// CRITICAL: Site-wide cart fragments loading
// Default behavior pre-WC 7.8, causes performance issues
```

### Pattern 7: Template Overrides with Hook Preservation
**What:** When overriding templates, preserve all action hooks
**When to use:** Template customization that can't be done via hooks alone
**Example:**
```php
// Source: https://developer.woocommerce.com/docs/theming/theme-development/template-structure/

// In child-theme/woocommerce/content-product.php

// GOOD: Preserve all action hooks from original template
do_action( 'woocommerce_before_shop_loop_item' );
do_action( 'woocommerce_before_shop_loop_item_title' );
// Custom content here
do_action( 'woocommerce_shop_loop_item_title' );
do_action( 'woocommerce_after_shop_loop_item_title' );
do_action( 'woocommerce_after_shop_loop_item' );

// CRITICAL: Deleted hooks in template override
// Breaks other plugins that hook into template
// No do_action() calls = no extension points

// BETTER: Use hooks instead of template override
add_action( 'woocommerce_shop_loop_item_title', 'add_custom_content', 15 );
function add_custom_content() {
    // Custom content via hook, no template override needed
}
```

### Pattern 8: Webhook Security
**What:** Verify webhook signatures, process async, handle duplicates
**When to use:** Receiving webhooks from WooCommerce or third-party services
**Example:**
```php
// Source: https://hookdeck.com/webhooks/platforms/how-to-secure-and-verify-woocommerce-webhooks-with-hookdeck

// GOOD: Verify webhook signature
function verify_wc_webhook( $payload, $signature, $secret ) {
    $expected = base64_encode( hash_hmac( 'sha256', $payload, $secret, true ) );

    // CRITICAL: Use hash_equals to prevent timing attacks
    return hash_equals( $expected, $signature );
}

// In webhook handler
$payload = file_get_contents( 'php://input' );
$signature = $_SERVER['HTTP_X_WC_WEBHOOK_SIGNATURE'] ?? '';
$delivery_id = $_SERVER['HTTP_X_WC_WEBHOOK_DELIVERY_ID'] ?? '';

// WARNING: Missing signature verification
// Allows forged webhooks

if ( ! verify_wc_webhook( $payload, $signature, $secret ) ) {
    http_response_code( 401 );
    exit;
}

// GOOD: Check for duplicate delivery
if ( get_transient( 'wc_webhook_' . $delivery_id ) ) {
    http_response_code( 200 ); // Already processed
    exit;
}
set_transient( 'wc_webhook_' . $delivery_id, true, DAY_IN_SECONDS );

// GOOD: Process async via Action Scheduler
as_enqueue_async_action( 'process_wc_webhook', array( 'payload' => $payload ) );

http_response_code( 200 );
exit;
```

### Anti-Patterns to Avoid
- **Direct database queries for orders/products:** Bypasses CRUD layer, breaks HPOS
- **get_posts() or WP_Query for shop_order post type:** Breaks on HPOS-enabled stores
- **Storing raw card data in database/sessions/logs:** PCI violation, security risk
- **wp_cron() for bulk WooCommerce operations:** Unreliable, traffic-dependent
- **Template overrides with deleted action hooks:** Breaks plugin extensibility
- **Site-wide cart fragments loading:** Major performance issue
- **Custom session implementations bypassing WC_Session_Handler:** Breaks session management
- **Missing HPOS compatibility declaration:** Extension silently incompatible

## Don't Hand-Roll

Problems that look simple but have existing WooCommerce solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Background processing | Custom cron system | Action Scheduler | Built into WC, reliable, traceable, retry logic, admin UI |
| Order data storage | Custom DB tables | WC_Order CRUD + meta | HPOS-compatible, WC APIs work, automatic migration |
| Product variations | Custom variation system | WC_Product_Variable | Price caching, attribute lookup table, variation picker UI |
| Cart updates | Custom AJAX handlers | Cart fragments or Mini-Cart Block | Session handling, nonce verification, cache compatibility |
| Payment tokenization | Custom token storage | WC_Payment_Token API | PCI-scoped, secure storage, customer UI for saved methods |
| Email notifications | Custom mailer | WC_Email class | Template system, filters, settings UI, customer preferences |
| Shipping calculations | Manual rate tables | WC_Shipping_Method | Zones, classes, tax integration, multi-currency |
| Session management | PHP sessions or custom tables | WC_Session_Handler | Cart persistence, 30-day cap (WC 10.1+), cleanup cron |
| REST API endpoints | Custom API | WooCommerce REST API v3 | Authentication, pagination, filtering, documentation |
| Product search | Custom SQL queries | WC_Product_Query | Indexed attributes, variation search, stock filtering |

**Key insight:** WooCommerce's architecture is mature (15+ years). Custom solutions miss edge cases (HPOS migration, session cleanup, webhook retries, variation price caching) that are complex to implement correctly. Use WooCommerce APIs for all data operations—they handle migration paths, performance optimizations, and backward compatibility.

## Common Pitfalls

### Pitfall 1: Missing HPOS Compatibility Declaration
**What goes wrong:** Extension appears to work in development but silently fails on HPOS-enabled production stores (default since WC 8.2). Orders aren't found, meta doesn't save, queries return empty results.
**Why it happens:** Developers test with legacy post-based orders or compatibility mode enabled, missing that HPOS is now the default for new stores.
**How to avoid:** Always declare compatibility status in main plugin file via `before_woocommerce_init` hook. Test with HPOS enabled AND disabled.
**Warning signs:**
- `wc_get_order( $id )` returns null on stores known to have orders
- Order meta operations silently fail
- WooCommerce Status page shows extension as incompatible

### Pitfall 2: Direct Post Access for Orders
**What goes wrong:** get_posts(), WP_Query, or direct $wpdb queries against wp_posts/wp_postmeta for orders break completely on HPOS-enabled stores.
**Why it happens:** Developers copy legacy WordPress patterns without understanding WooCommerce's CRUD layer.
**How to avoid:** Use wc_get_orders() and WC_Order methods exclusively. Never query wp_posts for post_type='shop_order'.
**Warning signs:**
- Queries using post_type='shop_order'
- get_posts() or WP_Query for orders
- Direct $wpdb->prepare() against wp_posts with order data
- update_post_meta() for order fields

### Pitfall 3: Cart Fragments Loading Site-Wide
**What goes wrong:** wc-cart-fragments.js loads on every page (even non-WC pages), making AJAX requests to admin-ajax.php on page load, causing performance issues especially on high-traffic sites.
**Why it happens:** WooCommerce loaded fragments globally pre-7.8. Extensions or themes may re-enable this behavior via filters.
**How to avoid:** Conditionally dequeue on non-WC pages or migrate to Mini-Cart Block (no fragments needed).
**Warning signs:**
- wc-cart-fragments.js loads on homepage/blog pages
- admin-ajax.php?action=wc_get_refreshed_fragments requests on every page
- No conditional is_woocommerce()/is_cart()/is_checkout() checks before enqueuing

### Pitfall 4: Template Overrides with Deleted Hooks
**What goes wrong:** Template override removes do_action() calls from original template, breaking other plugins that hook into those actions. Product add-ons, wishlist plugins, review systems fail silently.
**Why it happens:** Developers copy template markup but don't understand hook architecture. They remove "unnecessary" do_action() lines.
**How to avoid:** Compare overridden template against original, preserve all do_action() and apply_filters() calls. Use hooks instead of overrides when possible.
**Warning signs:**
- Template file with no do_action() calls
- Template version comment (@version) significantly older than current WC version
- Other plugins' features missing on product/cart pages

### Pitfall 5: Storing Raw Card Data
**What goes wrong:** Extension logs credit card numbers to error_log, stores them in database, or puts them in sessions. This is a critical PCI violation and security breach.
**Why it happens:** Developers add debugging or try to implement retry logic without understanding PCI scope.
**How to avoid:** Never access $_POST['card_number'] or similar fields. Use payment gateway APIs with tokenization. Never log payment data.
**Warning signs:**
- error_log() calls in payment processing code
- Database columns for card_number, cvv, expiry
- Session data containing payment information
- Direct access to $_POST in process_payment()

### Pitfall 6: WP_Query for Products (Future-Breaking)
**What goes wrong:** Code works today but WooCommerce is planning custom product tables migration (like HPOS for orders). WP_Query will break when that happens.
**Why it happens:** Developers use familiar WordPress patterns without checking WooCommerce documentation.
**How to avoid:** Use wc_get_products() or WC_Product_Query for all product queries. Mark WP_Query usage as WARNING (not yet broken but future-incompatible).
**Warning signs:**
- new WP_Query( array( 'post_type' => 'product' ) )
- get_posts() with product post type
- Direct $wpdb queries against wp_posts for products

### Pitfall 7: wp_cron() for Bulk WooCommerce Operations
**What goes wrong:** wp_cron() is traffic-dependent, has no retry logic, and can fail silently. Bulk order exports, inventory syncs, or report generation may never run or timeout.
**Why it happens:** Developers use WordPress's built-in cron without understanding its limitations for heavy tasks.
**How to avoid:** Use Action Scheduler (ships with WC) for all background processing. It has queue management, retry logic, and admin UI.
**Warning signs:**
- wp_schedule_event() for order processing
- Cron callbacks with foreach loops over all orders/products
- No batch processing or timeout handling
- Operations that should run regardless of site traffic

### Pitfall 8: Custom Session Filters Exceeding 30-Day Cap
**What goes wrong:** WooCommerce 10.1+ enforces 30-day maximum session duration. Extensions using wc_session_expiration filter to set longer durations trigger warnings and are capped at 30 days.
**Why it happens:** Extensions try to persist cart data indefinitely for abandoned cart recovery or long-term wishlists.
**How to avoid:** Respect 30-day cap. Use custom database tables for long-term persistence, not sessions.
**Warning signs:**
- wc_session_expiration filter returning > 30 days
- wc_session_expiring filter with long durations
- Warnings in WooCommerce Status Logs about session duration
- Custom session implementations bypassing WC_Session_Handler

### Pitfall 9: Missing Webhook Signature Verification
**What goes wrong:** Extension processes webhooks without verifying X-WC-Webhook-Signature header, allowing forged requests to trigger actions (order status changes, refunds, inventory updates).
**Why it happens:** Developers trust webhook source without implementing signature verification.
**How to avoid:** Always verify HMAC-SHA256 signature using webhook secret. Use hash_equals() to prevent timing attacks.
**Warning signs:**
- Webhook handler with no signature check
- Direct processing of $_POST data from webhooks
- No verification of X-WC-Webhook-Signature header
- Missing duplicate delivery checks (X-WC-Webhook-Delivery-ID)

### Pitfall 10: Outdated Template Overrides
**What goes wrong:** Template override is 2+ versions behind current WooCommerce version. Missing security patches, new features, or bug fixes. Checkout may break, prices display incorrectly, or AJAX fails.
**Why it happens:** Developers override template once and forget to update it when WooCommerce updates.
**How to avoid:** Monitor WooCommerce System Status for outdated templates. Compare @version comment against current WC version. Subscribe to WooCommerce developer blog for breaking changes.
**Warning signs:**
- System Status showing outdated template warnings
- @version 3.x.x in template when WC is 10.x
- Template in child theme hasn't been updated in 2+ years
- Features missing that exist in default WooCommerce

## Code Examples

Verified patterns from official WooCommerce documentation:

### HPOS Compatibility Detection
```php
// Source: https://developer.woocommerce.com/docs/features/high-performance-order-storage/recipe-book/

use Automattic\WooCommerce\Utilities\OrderUtil;

if ( OrderUtil::custom_orders_table_usage_is_enabled() ) {
    // HPOS tables are being used
    // Order data in wp_wc_orders, not wp_posts
} else {
    // Legacy post-based storage
    // Order data in wp_posts with post_type=shop_order
}

// GOOD: Write code that works in both modes
$order = wc_get_order( $order_id );
// Works regardless of storage mode
```

### Order Meta Operations (HPOS-Compatible)
```php
// Source: https://developer.woocommerce.com/docs/features/high-performance-order-storage/recipe-book/

// GOOD: CRUD methods for order meta
$order = wc_get_order( $order_id );
$order->update_meta_data( 'delivery_date', '2026-02-15' );
$order->add_meta_data( 'gift_message', 'Happy Birthday!', true ); // true = unique
$order->delete_meta_data( 'old_field' );
$order->save(); // Single save after all changes

// Get meta
$delivery_date = $order->get_meta( 'delivery_date', true );

// BAD: Direct meta functions (breaks HPOS)
update_post_meta( $order_id, 'delivery_date', '2026-02-15' );
$delivery_date = get_post_meta( $order_id, 'delivery_date', true );
```

### Custom Order Status Registration
```php
// Source: https://rudrastyh.com/woocommerce/order-statuses.html

// GOOD: Register custom order status
add_action( 'init', 'register_awaiting_shipment_status' );
function register_awaiting_shipment_status() {
    register_post_status( 'wc-awaiting-shipment', array(
        'label' => __( 'Awaiting Shipment', 'text-domain' ),
        'public' => true,
        'show_in_admin_status_list' => true,
        'label_count' => _n_noop(
            'Awaiting shipment <span class="count">(%s)</span>',
            'Awaiting shipment <span class="count">(%s)</span>',
            'text-domain'
        )
    ) );
}

// Add to WooCommerce status list
add_filter( 'wc_order_statuses', 'add_awaiting_shipment_to_order_statuses' );
function add_awaiting_shipment_to_order_statuses( $order_statuses ) {
    $order_statuses['wc-awaiting-shipment'] = __( 'Awaiting Shipment', 'text-domain' );
    return $order_statuses;
}

// Set order to custom status
$order = wc_get_order( $order_id );
$order->set_status( 'awaiting-shipment', 'Order is ready for shipment' );
```

### Shipping Method Implementation
```php
// Source: https://developer.woocommerce.com/docs/features/shipping/shipping-method-api/

class WC_Shipping_Custom extends WC_Shipping_Method {

    public function __construct( $instance_id = 0 ) {
        $this->id = 'custom_shipping';
        $this->instance_id = absint( $instance_id );
        $this->method_title = __( 'Custom Shipping', 'text-domain' );
        $this->method_description = __( 'Description', 'text-domain' );
        $this->supports = array( 'shipping-zones', 'instance-settings' );

        $this->init();
    }

    public function calculate_shipping( $package = array() ) {
        // Calculate rate based on package
        $rate = array(
            'id' => $this->id . $this->instance_id,
            'label' => $this->title,
            'cost' => 10.00,
            'calc_tax' => 'per_order'
        );

        // Add the rate
        $this->add_rate( $rate );
    }
}
```

### WooCommerce Blocks Checkout Integration
```php
// Source: https://developer.woocommerce.com/docs/block-development/extensible-blocks/cart-and-checkout-blocks/additional-checkout-fields/

// GOOD: Add checkout field via Additional Checkout Fields API (WC 8.6+)
add_action( 'woocommerce_init', 'register_custom_checkout_field' );
function register_custom_checkout_field() {
    woocommerce_register_additional_checkout_field( array(
        'id' => 'namespace/gift-message',
        'label' => __( 'Gift message', 'text-domain' ),
        'location' => 'order',
        'type' => 'text',
        'attributes' => array(
            'maxLength' => 200
        ),
        'show_in_confirmation_email' => true
    ) );
}

// Access field value after order
$order = wc_get_order( $order_id );
$gift_message = $order->get_meta( 'namespace/gift-message' );
```

### Coupon Validation Hook
```php
// Source: https://github.com/woocommerce/woocommerce/issues/39

// GOOD: Custom coupon validation
add_filter( 'woocommerce_coupon_is_valid', 'validate_custom_coupon_logic', 10, 3 );
function validate_custom_coupon_logic( $valid, $coupon, $discount ) {

    // Example: Require minimum 3 items in cart
    if ( WC()->cart->get_cart_contents_count() < 3 ) {
        throw new Exception(
            __( 'This coupon requires at least 3 items in cart.', 'text-domain' ),
            109 // Error code
        );
    }

    return $valid;
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| WordPress post tables for orders | HPOS (Custom Order Tables) | WC 8.2 (Oct 2023) | Default for new stores, 5x faster order creation, 40x faster filtering |
| WP_Query for orders | wc_get_orders() | WC 3.0+ (2017) | CRUD layer, HPOS-compatible |
| Site-wide cart fragments | Conditional loading or Mini-Cart Block | WC 7.8 (June 2023) | No longer loads globally by default |
| wp_cron() for background tasks | Action Scheduler | Shipped with WC 3.0+ | Reliable processing, retry logic, admin UI |
| direct product meta queries | WC_Product_Query | WC 3.0+ (2017) | Attribute lookup table, variation price caching |
| Custom order emails | WC_Email class | WC 2.0+ | Template system, settings integration |
| wc_enqueue_js() | WordPress core wp_add_inline_script() | Deprecated WC 10.4 (Nov 2025) | Standard WP patterns |
| Unlimited session duration | 30-day maximum | WC 10.1 (Aug 2025) | Performance improvement, prevents bloat |
| Manual analytics imports | Batch processing (100/12hrs) | WC 10.5 (Feb 2026) | Scalability for high-volume stores |
| Product instance creation every call | Product Object Caching (experimental) | WC 10.5 (Feb 2026) | Prevents duplicate DB loads |

**Deprecated/outdated:**
- **query_posts()**: Never use, breaks main query (deprecated WordPress-wide)
- **get_posts() for orders**: Breaks HPOS, use wc_get_orders()
- **WP_Query for products**: Future-breaking, custom product tables planned
- **Direct database queries for orders/products**: Bypasses CRUD, breaks HPOS
- **wc_enqueue_js()**: Deprecated WC 10.4, use wp_add_inline_script()
- **AccessiblePrivateMethods trait**: Removed WC 10.5, was in Internal namespace
- **Site-wide cart fragments**: Performance anti-pattern, conditionally dequeue

## Open Questions

Things that couldn't be fully resolved:

1. **Custom product tables migration timeline**
   - What we know: WooCommerce is planning custom product tables similar to HPOS for orders. WP_Query for products will break when this happens.
   - What's unclear: Exact release version and timeline for product tables migration
   - Recommendation: Flag WP_Query for products as WARNING (future-breaking), recommend WC_Product_Query now for future-proofing

2. **WooCommerce Admin/Analytics patterns**
   - What we know: WC 10.5 introduces batch analytics imports, experimental REST API cache. Admin UI is React-based with extensions via @woocommerce/components.
   - What's unclear: Whether to include WC Admin patterns (admin UI development) or focus only on storefront extensions
   - Recommendation: Surface-level coverage only—detect admin code loading on frontend (performance issue), note Admin API exists, but don't deep-dive into React admin development (out of scope for PHP code review)

3. **Exact WooCommerce version for testing**
   - What we know: WC 10.5 is current (Feb 2026), 8.2+ has HPOS default
   - What's unclear: Whether to target 8.2 minimum (HPOS default) or go back to 7.x (HPOS optional)
   - Recommendation: Target 8.2+ minimum (HPOS as requirement not optional feature). Extensions not HPOS-compatible break on modern stores.

4. **WooCommerce Blocks depth**
   - What we know: Block-based cart/checkout is the future, has Store API extension points, Additional Checkout Fields API (WC 8.6+)
   - What's unclear: How deep to cover Blocks development (React/JavaScript patterns vs PHP integration only)
   - Recommendation: Surface-level—detect missing block compatibility declarations, note Store API exists, recommend Additional Checkout Fields API, but don't become a Blocks development tutorial. Focus on PHP backend integration points.

5. **False positive patterns**
   - What we know: get_posts() is valid for non-order post types, WP_Query is valid for standard posts/pages
   - What's unclear: How to distinguish between legitimate post queries and WooCommerce anti-patterns
   - Recommendation: Context-aware detection—flag get_posts() with post_type='shop_order' (CRITICAL), WP_Query with post_type='product' (WARNING), but ignore get_posts() for custom post types in WC extensions

## Sources

### Primary (HIGH confidence)
- [High-Performance Order Storage (HPOS) | WooCommerce developer docs](https://developer.woocommerce.com/docs/features/high-performance-order-storage/) - HPOS architecture, default status
- [HPOS extension recipe book | WooCommerce developer docs](https://developer.woocommerce.com/docs/features/high-performance-order-storage/recipe-book/) - declare_compatibility code, CRUD patterns, migration checklist
- [Developing using WooCommerce CRUD objects | WooCommerce developer docs](https://developer.woocommerce.com/docs/best-practices/data-management/crud-objects/) - WC_Data architecture, CRUD methods
- [WooCommerce Payment Gateway API | WooCommerce developer docs](https://developer.woocommerce.com/docs/features/payments/payment-gateway-api) - WC_Payment_Gateway class, process_payment lifecycle
- [Best practices for the use of the "cart fragments" API | WooCommerce Developer Blog](https://developer.woocommerce.com/2023/06/16/best-practices-for-the-use-of-the-cart-fragments-api/) - Cart fragments performance, conditional dequeuing, Mini-Cart Block
- [Template structure & Overriding templates via a theme | WooCommerce developer docs](https://developer.woocommerce.com/docs/theming/theme-development/template-structure/) - Template hierarchy, version tracking, hooks-first philosophy
- [WooCommerce 10.5: What's coming for developers | WooCommerce Developer Blog](https://developer.woocommerce.com/2026/01/20/woocommerce-10-5-whats-coming-for-developers/) - Product object caching, variation price caching, deprecations
- [Developer Advisory: Changes to session management in WooCommerce 10.1 | WooCommerce Developer Blog](https://developer.woocommerce.com/2025/08/08/developer-advisory-changes-to-session-management-and-cron-jobs-in-woocommerce-10-1/) - 30-day session cap, enforcement details
- [Action Scheduler - Background Processing Job Queue](https://actionscheduler.org/) - Action Scheduler architecture, vs wp_cron
- [WooCommerce Code Reference](https://woocommerce.github.io/code-reference/) - Class documentation, method signatures

### Secondary (MEDIUM confidence)
- [How to Secure and Verify WooCommerce Webhooks with Hookdeck](https://hookdeck.com/webhooks/platforms/how-to-secure-and-verify-woocommerce-webhooks-with-hookdeck) - Webhook signature verification, X-WC-Webhook-Signature
- [Shipping method API | WooCommerce developer docs](https://developer.woocommerce.com/docs/features/shipping/shipping-method-api/) - WC_Shipping_Method implementation
- [Getting started with Cart and Checkout extensibility | WooCommerce developer docs](https://developer.woocommerce.com/docs/block-development/extensible-blocks/cart-and-checkout-blocks/) - WooCommerce Blocks integration
- [Additional checkout fields | WooCommerce developer docs](https://developer.woocommerce.com/docs/block-development/extensible-blocks/cart-and-checkout-blocks/additional-checkout-fields/) - Additional Checkout Fields API (WC 8.6+)
- [WooCommerce Order Status Explained: The Complete 2026 Guide](https://funnelkit.com/woocommerce-order-status/) - Order status transitions, custom statuses
- [Variation prices caching improvements in WooCommerce 10.5 | WooCommerce Developer Blog](https://developer.woocommerce.com/2026/01/08/variation-prices-caching-improvements-in-woocommerce-10-5/) - Variation price caching details
- [PCI-DSS compliance and WooCommerce Documentation](https://woocommerce.com/document/pci-dss-compliance-and-woocommerce/) - Payment security scope, PCI levels

### Tertiary (LOW confidence - community sources)
- [WooCommerce: Add Custom Order Status](https://rudrastyh.com/woocommerce/order-statuses.html) - Community guide, custom status registration patterns
- [WooCommerce Order Lifecycle Hooks](https://rudrastyh.com/woocommerce/order-lifecycle-hooks.html) - Hook catalog (verify against official docs)
- Community blog posts on cart fragments optimization, template overrides, coupon validation - useful for patterns but verify against official docs

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Official WooCommerce documentation, current version information (WC 10.5 released Feb 2026)
- Architecture: HIGH - HPOS patterns from official recipe book, CRUD API from official docs, payment gateway from official API docs
- Pitfalls: MEDIUM - Combination of official advisories (session cap, HPOS migration) and community-reported issues (cart fragments, template hooks)
- Performance: HIGH - Official developer advisories for cart fragments, Action Scheduler, variation caching, batch analytics
- Security: MEDIUM - Official PCI scope documentation, webhook security from third-party but verified patterns

**Research date:** 2026-02-06
**Valid until:** 30 days (Apr 2026) for stable APIs, 7 days for fast-moving WooCommerce features (experimental caching, Blocks APIs)

**Notes:**
- WooCommerce 10.5 is current as of research date (Feb 3, 2026 release)
- HPOS has been default since WC 8.2 (Oct 2023) - mature, not experimental
- Cart fragments improvements in WC 7.8 but still a performance concern in 2026
- Custom product tables migration timeline uncertain—flag as future-breaking but not immediate
- Product object caching is experimental in 10.5, may change before stable release
