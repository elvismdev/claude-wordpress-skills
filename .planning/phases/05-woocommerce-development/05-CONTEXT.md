# Phase 5: WooCommerce Development - Context

**Gathered:** 2026-02-06
**Status:** Ready for planning

<domain>
## Phase Boundary

Deliver the `wp-woocommerce-dev` skill for WooCommerce extension and theme integration code review -- extension architecture (HPOS, CRUD, data stores), payment gateway patterns, WooCommerce hooks, template override system, cart/checkout block integration, and performance optimization. Also deliver infrastructure updates (README.md, marketplace.json, plugin.json version, CHANGELOG.md) to complete the plugin build-out. Targets WooCommerce 8.2+/10.x with HPOS as default. Code-level review only -- no store setup, hosting, or PCI compliance auditing.

</domain>

<decisions>
## Implementation Decisions

### Detection scope & depth
- **Deep coverage (dedicated sections in SKILL.md):**
  - HPOS compatibility -- declare_compatibility() calls, CRUD-only order access, wc_get_orders()/wc_get_order() vs get_posts(), meta operations via WC_Order methods + save(), compatibility mode awareness
  - WooCommerce CRUD pattern -- WC_Data base class, data stores (WC_Object_Data_Store_Interface), wc_get_products()/WC_Product_Query vs WP_Query for products, custom data store registration via woocommerce_data_stores filter
  - Payment gateway development -- WC_Payment_Gateway class, process_payment() flow, refund handling (process_refund()), saved payment methods (WC_Payment_Token), card data anti-patterns (never store/log raw card data, token-based storage only)
  - WooCommerce hooks catalog -- lifecycle hooks (woocommerce_before_cart, woocommerce_checkout_process, woocommerce_thankyou, woocommerce_order_status_changed), product data hooks (woocommerce_product_options_*, woocommerce_process_product_meta), checkout field hooks, email hooks
  - Order handling -- WC_Order CRUD, status transitions (wc_get_order_statuses()), custom order statuses, order item manipulation, order notes
  - Cart operations -- WC()->cart methods, cart item data (woocommerce_add_cart_item_data), cart fees, cart validation hooks
  - Custom product types -- register_product_type(), extending WC_Product, product data stores, product type class hierarchy
  - Shipping methods -- WC_Shipping_Method class, calculate_shipping(), shipping zones, flat rate/free shipping customization
  - Cart fragments performance -- detect site-wide wc-cart-fragments.js loading, recommend Mini-Cart Block or conditional dequeuing, flag as WARNING/CRITICAL based on scope
  - Action Scheduler patterns -- detect wp_cron() for heavy WC tasks, recommend as_schedule_single_action()/as_enqueue_async_action(), flag synchronous bulk processing
  - Webhook security -- signature verification (X-WC-Webhook-Signature), async processing, duplicate delivery handling (X-WC-Webhook-Delivery-ID)
  - Template override anti-patterns -- detect overrides with deleted action hooks (CRITICAL), outdated template versions (WARNING), recommend hooks-first approach
- **Surface-level coverage (scan patterns + brief guidance):**
  - WooCommerce Blocks checkout integration -- Store API, block checkout extension points, compatibility declarations for block cart/checkout, Additional Checkout Fields API
  - WooCommerce REST API extensions -- v3 endpoints, custom endpoint registration, authentication patterns (Basic Auth, OAuth 1.0a), HTTPS enforcement
  - Coupon/discount validation -- custom coupon types, validation hooks (woocommerce_coupon_is_valid)
  - Session handling -- WC_Session_Handler, 30-day cap (WC 10.1+), avoid custom session implementations
  - Email customization -- WC_Email class extension, template overrides for emails
  - Product variations performance -- attribute lookup table, variation price caching, dynamic dropdown limits
  - WooCommerce Blocks compatibility with template overrides -- note incompatibility, recommend block integration for new stores
  - Experimental product object caching (WC 10.5+) -- awareness only
- **Context-aware detection:** Distinguish between:
  - WooCommerce extension/plugin (standalone plugin extending WC functionality)
  - WooCommerce theme integration (theme with woocommerce/ template overrides)
  - Payment gateway (WC_Payment_Gateway subclass -- heightened security review)
  - Shipping method (WC_Shipping_Method subclass)
  - Custom product type (WC_Product subclass with data store)
  - WooCommerce Blocks integration (block-based checkout/cart extension)

### HPOS handling strategy
- **HPOS is default for new stores since WC 8.2** -- all new extension code MUST use CRUD APIs
- **Missing HPOS compatibility declaration = CRITICAL** for new extensions (declare_compatibility() in before_woocommerce_init hook)
- **Direct post access for orders (get_posts with shop_order) = CRITICAL** -- will break on HPOS-enabled stores
- **WP_Query for products = WARNING** (custom product tables migration planned, not yet enforced but future-breaking)
- **Direct database queries for orders/products = CRITICAL** -- bypasses both HPOS and CRUD
- Same future-proofing pattern: flag code that works today but will break in upcoming WC releases

### Payment security boundary
- **Code-level anti-patterns only** -- this is NOT a PCI compliance audit
- **CRITICAL violations:** Storing raw card numbers/CVV in database, logging card data (including in error_log/debug.log), transmitting card data over HTTP, storing full card data in sessions
- **WARNING violations:** Hardcoded API credentials, missing HTTPS check before payment processing, no webhook signature verification
- **Explicit non-scope:** We do not assess PCI SAQ level, hosting compliance, or network security -- note that PCI compliance is the store owner's responsibility and recommend professional PCI assessment
- Pattern: detect what code does wrong, not what infrastructure is missing

### Performance detection patterns
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
  - Recommend: Action Scheduler (as_schedule_single_action, as_enqueue_async_action) -- ships with WC
  - INFO: Synchronous processing of operations that should be async
- **Session handling:**
  - WARNING: Custom session filters exceeding 30-day cap (WC 10.1+ enforces limit)
  - INFO: Custom session implementations bypassing WC_Session_Handler

### Template override vs hooks guidance
- **Hooks are strongly preferred over template overrides** -- this is a key teaching point
- **Template overrides with deleted action hooks = CRITICAL** -- breaks plugin integration, other extensions can't hook in
- **Outdated template overrides (version mismatch) = WARNING** -- may miss WC updates, cause display issues
- **Template override where a hook exists = INFO** -- suggest using hook instead
- **Template version tracking:** Detect @version comments in overridden templates, compare against WC version expectations
- WooCommerce Blocks pages bypass template overrides entirely -- note this as awareness item

### Cross-references to existing skills
- **wp-security-review:** Payment data handling (never store raw cards), REST API authentication, AJAX nonce verification for add-to-cart, capability checks in order management
- **wp-plugin-development:** WC as a plugin extension (proper dependency checking, activation/deactivation), hooks architecture, REST API endpoint registration, Settings API for WC settings pages
- **wp-block-development:** WooCommerce Blocks (product grid block, cart block, checkout block), block-based checkout extension points, Store API integration
- **wp-theme-development:** WooCommerce template overrides in themes (woocommerce/ directory), shop page customization, product display templates, WC Blocks compatibility with block themes
- **wp-performance-review:** Cart fragments optimization, WC_Product_Query vs WP_Query, session table performance, Action Scheduler vs wp_cron, transient usage for product data caching
- Same cross-reference pattern as prior skills: brief mention + pointer, don't duplicate content

### Reference doc structure
- Follow established cookbook format: quick reference table + detailed patterns + edge cases + WC nuances
- **Three reference docs:**
  - `wc-extension-guide.md` -- Complete extension development patterns: HPOS compatibility (declare_compatibility, CRUD-only access, migration checklist), payment gateway development (WC_Payment_Gateway lifecycle, process_payment flow, refund handling, saved payment methods, security anti-patterns), shipping method development (WC_Shipping_Method, calculate_shipping, zones), custom product types (register_product_type, data stores), order handling (WC_Order CRUD, status transitions, custom statuses), cart operations (cart item data, fees, validation), REST API extensions, WooCommerce Blocks checkout integration, Action Scheduler patterns, webhook security
  - `wc-template-guide.md` -- Template override patterns: complete template hierarchy (archive-product, content-product, single-product, cart, checkout, myaccount), proper override procedure (child theme woocommerce/ directory, version tracking), hooks-first philosophy (when to use hooks vs overrides, preserving action hooks in overrides), shop page customization, product display patterns, cart/checkout theming, WooCommerce Blocks compatibility notes, email template customization
  - `wc-performance-guide.md` -- Performance optimization patterns: cart fragments analysis (problem, solutions, Mini-Cart Block alternative), WC_Product_Query optimization, HPOS performance benefits, session handling (30-day cap, cleanup), Action Scheduler for background processing, transient and object caching strategies, variation price caching, product attribute lookup table, database optimization (wp_postmeta index recommendations), query monitoring patterns
- BAD/GOOD pairs throughout in PHP, WordPress PHP Coding Standards

### Finding output style
- Mirror all prior skills exactly:
  - Group findings by FILE (PHP files organized by actual file path)
  - Each finding: line number, severity, issue description, explanation, BAD/GOOD code pair
  - Summary at end with total counts by severity
- **Severity mapping for WooCommerce:**
  - CRITICAL: Extension will break on modern WC stores or has security vulnerabilities (missing HPOS compatibility, direct DB access for orders/products, raw card data storage, deleted hooks in template overrides, no webhook signature verification)
  - WARNING: Extension works but has quality/compatibility/performance issues (WP_Query for products, cart fragments site-wide, outdated template overrides, hardcoded API credentials, wp_cron for bulk operations, session duration exceeding 30 days)
  - INFO: Best practice improvements (hooks instead of template overrides, Action Scheduler instead of wp_cron for light tasks, object caching for product loads, block checkout integration opportunity)

### Quick scan vs full review boundary
- `/wp-woo` (quick scan): Grep-based pattern detection -- scan for get_posts with shop_order, direct $wpdb queries against WC tables, missing declare_compatibility, raw card data patterns, cart-fragments loading without dequeue, template overrides without do_action, WP_Query with post_type=product
- `/wp-woo-review` (full review): Complete skill workflow -- WC context detection (extension/theme/gateway/shipping), HPOS compatibility audit, payment security review, hook usage analysis, template override assessment, performance pattern detection, produces structured report
- Same boundary pattern as all prior skills

### Infrastructure updates (Plan 3)
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

</decisions>

<specifics>
## Specific Ideas

- Use the existing skills (wp-security-review, wp-plugin-development, wp-block-development, wp-theme-development) as structural templates -- same 12-section layout
- The description field in YAML frontmatter should include trigger phrases: "WooCommerce review", "WooCommerce extension", "WooCommerce plugin", "payment gateway", "shipping method", "HPOS", "High-Performance Order Storage", "custom product type", "WC hooks", "cart fragments", "checkout block", "WooCommerce template", "wc_get_orders", "WC_Payment_Gateway", "woocommerce_checkout", "shop_order", "WooCommerce performance", "cart optimization", "WooCommerce REST API", "Action Scheduler", "WooCommerce blocks"
- HPOS is the single most important architectural shift in WooCommerce -- skill should make this front and center, analogous to how theme.json v3 was front and center in Phase 4
- Payment gateway section should be security-conscious without becoming a PCI compliance checklist -- focus on what a code reviewer can detect in source code
- Cart fragments is the most common WooCommerce performance complaint -- give it prominent coverage similar to how N+1 queries get prominent coverage in wp-performance-review
- WooCommerce 10.x is current (10.5 as of Feb 2026) -- skill should target 8.2+ (HPOS default) through current
- This is the final skill and the most cross-referential -- it touches security (payment), plugin architecture (extension), blocks (WC Blocks), themes (template overrides), and performance (cart fragments, queries)

</specifics>

<deferred>
## Deferred Ideas

None -- all WooCommerce development concerns fit within phase scope. Infrastructure updates (README, marketplace.json, plugin.json, CHANGELOG) are explicitly scoped in this phase per the roadmap.

</deferred>

---

*Phase: 05-woocommerce-development*
*Context gathered: 2026-02-06*
