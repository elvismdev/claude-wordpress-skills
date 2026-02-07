# WooCommerce Template Hierarchy and Customization Reference

This reference catalogs WooCommerce template hierarchy, override procedures, hooks-first philosophy, shop/product/cart/checkout customization, email templates, WooCommerce Blocks compatibility, and template anti-patterns. Each pattern includes hook examples and BAD/GOOD override pairs following WordPress PHP Coding Standards.

## Quick Reference Table

| Template File | Location | Purpose | Key Hooks |
|---------------|----------|---------|-----------|
| **archive-product.php** | `woocommerce/` | Shop page, product archives | `woocommerce_before_shop_loop`, `woocommerce_after_shop_loop` |
| **content-product.php** | `woocommerce/` | Individual product in loop | `woocommerce_before_shop_loop_item`, `woocommerce_shop_loop_item_title`, `woocommerce_after_shop_loop_item` |
| **single-product.php** | `woocommerce/` | Single product wrapper | `woocommerce_before_single_product`, `woocommerce_after_single_product` |
| **content-single-product.php** | `woocommerce/` | Single product content | `woocommerce_before_single_product_summary`, `woocommerce_single_product_summary`, `woocommerce_after_single_product_summary` |
| **cart/cart.php** | `woocommerce/cart/` | Cart page | `woocommerce_before_cart`, `woocommerce_cart_contents`, `woocommerce_after_cart_table`, `woocommerce_cart_collaterals` |
| **checkout/form-checkout.php** | `woocommerce/checkout/` | Checkout page | `woocommerce_before_checkout_form`, `woocommerce_checkout_before_customer_details`, `woocommerce_review_order_before_payment` |
| **myaccount/my-account.php** | `woocommerce/myaccount/` | My Account dashboard | `woocommerce_account_navigation`, `woocommerce_account_content` |
| **myaccount/view-order.php** | `woocommerce/myaccount/` | Order details | `woocommerce_order_details_before_order_table`, `woocommerce_order_details_after_order_table` |
| **emails/*.php** | `woocommerce/emails/` | Email templates | `woocommerce_email_header`, `woocommerce_email_order_details`, `woocommerce_email_footer` |

## Template Override Procedure

### Step-by-Step Override Process

**1. Locate the template in WooCommerce plugin:**

```
wp-content/plugins/woocommerce/templates/
├── archive-product.php
├── single-product.php
├── cart/
│   └── cart.php
├── checkout/
│   └── form-checkout.php
└── emails/
    └── customer-completed-order.php
```

**2. Copy to theme's woocommerce directory:**

```php
// GOOD: Copy to theme (preserves directory structure)
wp-content/themes/my-theme/woocommerce/cart/cart.php

// GOOD: Copy to child theme (overrides parent theme's override)
wp-content/themes/child-theme/woocommerce/cart/cart.php

// BAD: Editing plugin files directly
wp-content/plugins/woocommerce/templates/cart/cart.php
// Lost on WooCommerce updates
```

**3. Preserve directory structure:**

```
my-theme/
└── woocommerce/
    ├── archive-product.php
    ├── single-product.php
    ├── cart/
    │   ├── cart.php
    │   └── cart-empty.php
    ├── checkout/
    │   └── form-checkout.php
    ├── myaccount/
    │   ├── my-account.php
    │   └── view-order.php
    └── emails/
        └── customer-completed-order.php
```

### Version Tracking

**Every WooCommerce template has a @version comment:**

```php
<?php
/**
 * Cart Page
 *
 * @package WooCommerce\Templates
 * @version 8.5.0
 */

if ( ! defined( 'ABSPATH' ) ) {
    exit;
}
```

**Check for outdated templates:**
- Go to WooCommerce → Status → System Status
- Scroll to "Templates" section
- Lists all overridden templates with version mismatch warnings

**Severity:**
- **CRITICAL:** Template override missing do_action() calls
- **WARNING:** Template version 2+ versions behind current WC version
- **INFO:** Template version 1 version behind (minor update needed)

### Child Theme Override Priority

```php
// Template load order (WordPress checks in this order):
// 1. child-theme/woocommerce/cart/cart.php
// 2. parent-theme/woocommerce/cart/cart.php
// 3. plugins/woocommerce/templates/cart/cart.php

// BAD: Override in parent theme instead of child theme
// parent-theme/woocommerce/cart/cart.php
// Lost when parent theme updates

// GOOD: Override in child theme
// child-theme/woocommerce/cart/cart.php
// Survives parent theme updates
```

## Hooks-First Philosophy

**Key teaching point:** Hooks are strongly preferred over template overrides for maintainability, compatibility, and update safety.

### When Hooks Work (Use These Instead)

**Adding content before/after elements:**

```php
// GOOD: Hook into existing template
add_action( 'woocommerce_before_shop_loop_item_title', 'add_product_badge', 15 );
function add_product_badge() {
    global $product;
    if ( $product->is_on_sale() ) {
        echo '<span class="sale-badge">' . __( 'Sale!', 'text-domain' ) . '</span>';
    }
}

// BAD: Override content-product.php just to add badge
// Creates maintenance burden, breaks on WC updates
```

**Modifying displayed data:**

```php
// GOOD: Filter product title
add_filter( 'the_title', 'modify_product_title', 10, 2 );
function modify_product_title( $title, $id ) {
    if ( is_product() && in_the_loop() ) {
        $product = wc_get_product( $id );
        if ( $product && $product->is_on_sale() ) {
            $title .= ' - ' . __( 'On Sale!', 'text-domain' );
        }
    }
    return $title;
}

// BAD: Override content-single-product.php to change title
```

**Conditional display:**

```php
// GOOD: Remove related products via hook
remove_action( 'woocommerce_after_single_product_summary', 'woocommerce_output_related_products', 20 );

// BAD: Override single-product.php and delete related products section
```

### When Overrides Are Necessary

**Fundamentally restructuring layout:**
- Moving product image below product description
- Reordering checkout fields (billing before shipping)
- Changing cart table column order

**Removing/reordering sections not achievable via remove_action():**
- Template has hardcoded HTML structure
- No hooks exist at desired insertion point

**Rule:** If you must override, preserve ALL `do_action()` and `apply_filters()` calls.

### Hook Preservation in Overrides

```php
// GOOD: Preserve all action hooks from original template
// In woocommerce/content-product.php

do_action( 'woocommerce_before_shop_loop_item' );

echo '<div class="custom-wrapper">'; // Custom markup

do_action( 'woocommerce_before_shop_loop_item_title' );
do_action( 'woocommerce_shop_loop_item_title' );
do_action( 'woocommerce_after_shop_loop_item_title' );

echo '</div>'; // Custom markup

do_action( 'woocommerce_after_shop_loop_item' );

// CRITICAL: Deleted hooks in template override
// BAD: No do_action() calls
echo '<div class="product">';
echo '<h3>' . get_the_title() . '</h3>';
echo '<span class="price">' . $product->get_price_html() . '</span>';
echo '</div>';
// Breaks other plugins that hook into template
// Product add-ons, wishlist, reviews won't display
```

**Severity:** CRITICAL - Removing hooks breaks extensibility. Other plugins can't integrate with template.

**Cross-reference:** See wp-theme-development for general template patterns and template hierarchy.

## Shop Page Customization

### Archive Template (Shop Page)

**Template:** `archive-product.php`

**Key hooks and default priorities:**

```php
// Before shop loop
do_action( 'woocommerce_before_shop_loop' );
// Priority 10: woocommerce_result_count
// Priority 20: woocommerce_catalog_ordering

// Shop loop start
woocommerce_product_loop_start();

// Each product in loop (content-product.php)
do_action( 'woocommerce_before_shop_loop_item' );        // Priority 10: woocommerce_template_loop_product_link_open
do_action( 'woocommerce_before_shop_loop_item_title' );  // Priority 10: woocommerce_show_product_loop_sale_flash
                                                         // Priority 10: woocommerce_template_loop_product_thumbnail
do_action( 'woocommerce_shop_loop_item_title' );         // Priority 10: woocommerce_template_loop_product_title
do_action( 'woocommerce_after_shop_loop_item_title' );   // Priority 10: woocommerce_template_loop_rating
                                                         // Priority 10: woocommerce_template_loop_price
do_action( 'woocommerce_after_shop_loop_item' );         // Priority 10: woocommerce_template_loop_product_link_close
                                                         // Priority 10: woocommerce_template_loop_add_to_cart

// Shop loop end
woocommerce_product_loop_end();

// After shop loop
do_action( 'woocommerce_after_shop_loop' );
// Priority 10: woocommerce_pagination
```

### Hook Examples

```php
// Add custom badge before product title
add_action( 'woocommerce_before_shop_loop_item_title', 'add_featured_badge', 5 );
function add_featured_badge() {
    global $product;
    if ( $product->is_featured() ) {
        echo '<span class="featured-badge">' . __( 'Featured', 'text-domain' ) . '</span>';
    }
}

// Add custom content after price
add_action( 'woocommerce_after_shop_loop_item_title', 'add_stock_status', 15 );
function add_stock_status() {
    global $product;
    if ( ! $product->is_in_stock() ) {
        echo '<span class="out-of-stock">' . __( 'Out of Stock', 'text-domain' ) . '</span>';
    }
}

// Remove result count
remove_action( 'woocommerce_before_shop_loop', 'woocommerce_result_count', 20 );

// Reorder catalog ordering (move to after loop)
remove_action( 'woocommerce_before_shop_loop', 'woocommerce_catalog_ordering', 30 );
add_action( 'woocommerce_after_shop_loop', 'woocommerce_catalog_ordering', 5 );
```

**Bad pattern:**

```php
// BAD: Override content-product.php to add price badge
// Creates maintenance burden

// GOOD: Use hook instead
add_action( 'woocommerce_after_shop_loop_item_title', 'add_discount_percentage', 11 );
function add_discount_percentage() {
    global $product;
    if ( $product->is_on_sale() ) {
        $regular = $product->get_regular_price();
        $sale = $product->get_sale_price();
        $percentage = round( ( ( $regular - $sale ) / $regular ) * 100 );
        echo '<span class="discount">' . sprintf( __( '%s%% off', 'text-domain' ), $percentage ) . '</span>';
    }
}
```

## Product Display Customization

### Single Product Template

**Template:** `single-product.php` (wrapper) and `content-single-product.php` (content)

**Key hooks and default priorities:**

```php
do_action( 'woocommerce_before_single_product' );

echo '<div id="product-' . esc_attr( $post->ID ) . '" class="' . esc_attr( implode( ' ', $classes ) ) . '">';

    do_action( 'woocommerce_before_single_product_summary' );
    // Priority 10: woocommerce_show_product_sale_flash
    // Priority 20: woocommerce_show_product_images

    echo '<div class="summary entry-summary">';

        do_action( 'woocommerce_single_product_summary' );
        // Priority 5:  woocommerce_template_single_title
        // Priority 10: woocommerce_template_single_rating
        // Priority 10: woocommerce_template_single_price
        // Priority 20: woocommerce_template_single_excerpt
        // Priority 30: woocommerce_template_single_add_to_cart
        // Priority 40: woocommerce_template_single_meta

    echo '</div>';

    do_action( 'woocommerce_after_single_product_summary' );
    // Priority 10: woocommerce_output_product_data_tabs
    // Priority 15: woocommerce_upsell_display
    // Priority 20: woocommerce_output_related_products

echo '</div>';

do_action( 'woocommerce_after_single_product' );
```

### Product Tabs Customization

```php
// Add custom tab
add_filter( 'woocommerce_product_tabs', 'add_shipping_tab' );
function add_shipping_tab( $tabs ) {
    $tabs['shipping'] = array(
        'title' => __( 'Shipping Info', 'text-domain' ),
        'priority' => 50,
        'callback' => 'shipping_tab_content'
    );
    return $tabs;
}

function shipping_tab_content() {
    echo '<h2>' . __( 'Shipping Information', 'text-domain' ) . '</h2>';
    echo '<p>' . __( 'Free shipping on orders over $50.', 'text-domain' ) . '</p>';
}

// Remove tab
add_filter( 'woocommerce_product_tabs', 'remove_description_tab', 98 );
function remove_description_tab( $tabs ) {
    unset( $tabs['description'] );
    return $tabs;
}

// Reorder tabs
add_filter( 'woocommerce_product_tabs', 'reorder_product_tabs', 98 );
function reorder_product_tabs( $tabs ) {
    $tabs['reviews']['priority'] = 10;           // Move reviews first
    $tabs['description']['priority'] = 20;       // Description second
    $tabs['additional_information']['priority'] = 30; // Additional info third
    return $tabs;
}
```

### Related Products Customization

```php
// Remove related products
remove_action( 'woocommerce_after_single_product_summary', 'woocommerce_output_related_products', 20 );

// Customize related products count
add_filter( 'woocommerce_output_related_products_args', 'custom_related_products_args' );
function custom_related_products_args( $args ) {
    $args['posts_per_page'] = 6; // Show 6 instead of 4
    $args['columns'] = 3;        // 3 columns instead of 4
    return $args;
}
```

### Product Variations

```php
// Modify variation data
add_filter( 'woocommerce_available_variation', 'customize_variation_data', 10, 3 );
function customize_variation_data( $data, $product, $variation ) {
    // Add custom data to variation
    $data['custom_field'] = $variation->get_meta( 'custom_field' );
    return $data;
}

// Modify variation price HTML
add_filter( 'woocommerce_format_sale_price', 'custom_variation_price_html', 10, 3 );
function custom_variation_price_html( $price, $regular_price, $sale_price ) {
    $price = '<del>' . ( is_numeric( $regular_price ) ? wc_price( $regular_price ) : $regular_price ) . '</del> ';
    $price .= '<ins>' . ( is_numeric( $sale_price ) ? wc_price( $sale_price ) : $sale_price ) . '</ins>';
    $price .= ' <span class="savings">' . __( 'Save!', 'text-domain' ) . '</span>';
    return $price;
}
```

## Cart/Checkout Customization

### Cart Template

**Template:** `cart/cart.php`

**Key hooks:**

```php
do_action( 'woocommerce_before_cart' );

echo '<form class="woocommerce-cart-form" action="' . esc_url( wc_get_cart_url() ) . '" method="post">';

    do_action( 'woocommerce_before_cart_table' );

    // Cart table
    do_action( 'woocommerce_before_cart_contents' );
    // Loop through cart items
    do_action( 'woocommerce_cart_contents' );
    do_action( 'woocommerce_cart_coupon' );
    do_action( 'woocommerce_after_cart_contents' );

    do_action( 'woocommerce_after_cart_table' );

echo '</form>';

do_action( 'woocommerce_before_cart_collaterals' );

echo '<div class="cart-collaterals">';
    do_action( 'woocommerce_cart_collaterals' );
    // Priority 10: woocommerce_cross_sell_display
    // Priority 20: woocommerce_cart_totals
echo '</div>';

do_action( 'woocommerce_after_cart' );
```

### Checkout Template

**Template:** `checkout/form-checkout.php`

**Key hooks:**

```php
do_action( 'woocommerce_before_checkout_form', $checkout );

echo '<form name="checkout" method="post" class="checkout woocommerce-checkout" action="' . esc_url( wc_get_checkout_url() ) . '" enctype="multipart/form-data">';

    do_action( 'woocommerce_checkout_before_customer_details' );

    echo '<div class="col2-set" id="customer_details">';
        do_action( 'woocommerce_checkout_billing' );
        do_action( 'woocommerce_checkout_shipping' );
    echo '</div>';

    do_action( 'woocommerce_checkout_after_customer_details' );

    echo '<h3 id="order_review_heading">' . esc_html__( 'Your order', 'woocommerce' ) . '</h3>';

    do_action( 'woocommerce_checkout_before_order_review' );

    echo '<div id="order_review" class="woocommerce-checkout-review-order">';
        do_action( 'woocommerce_checkout_order_review' );
    echo '</div>';

    do_action( 'woocommerce_checkout_after_order_review' );

echo '</form>';

do_action( 'woocommerce_after_checkout_form', $checkout );
```

### Checkout Field Customization

```php
// Add custom checkout field
add_filter( 'woocommerce_checkout_fields', 'add_custom_checkout_field' );
function add_custom_checkout_field( $fields ) {
    $fields['billing']['billing_phone_alt'] = array(
        'type' => 'text',
        'label' => __( 'Alternate Phone', 'text-domain' ),
        'required' => false,
        'class' => array( 'form-row-wide' ),
        'priority' => 25 // After main phone field
    );
    return $fields;
}

// Remove checkout field
add_filter( 'woocommerce_checkout_fields', 'remove_company_field' );
function remove_company_field( $fields ) {
    unset( $fields['billing']['billing_company'] );
    unset( $fields['shipping']['shipping_company'] );
    return $fields;
}

// Validate custom field
add_action( 'woocommerce_checkout_process', 'validate_custom_checkout_field' );
function validate_custom_checkout_field() {
    if ( isset( $_POST['billing_phone_alt'] ) && strlen( $_POST['billing_phone_alt'] ) < 10 ) {
        wc_add_notice( __( 'Alternate phone must be at least 10 digits.', 'text-domain' ), 'error' );
    }
}

// Save custom field to order meta
add_action( 'woocommerce_checkout_update_order_meta', 'save_custom_checkout_field' );
function save_custom_checkout_field( $order_id ) {
    if ( isset( $_POST['billing_phone_alt'] ) ) {
        $order = wc_get_order( $order_id );
        $order->update_meta_data( 'alternate_phone', sanitize_text_field( $_POST['billing_phone_alt'] ) );
        $order->save();
    }
}
```

**Bad pattern:**

```php
// BAD: Override form-checkout.php for field changes
// Creates maintenance burden, breaks on WC updates

// GOOD: Use woocommerce_checkout_fields filter
add_filter( 'woocommerce_checkout_fields', 'customize_checkout_fields' );
function customize_checkout_fields( $fields ) {
    // Modify fields array
    return $fields;
}
```

## My Account Templates

### My Account Dashboard

**Template:** `myaccount/my-account.php`

**Key hooks:**

```php
do_action( 'woocommerce_before_account_navigation' );

woocommerce_account_navigation(); // Left sidebar navigation

do_action( 'woocommerce_after_account_navigation' );

do_action( 'woocommerce_account_content' );
// Content area based on current endpoint
```

### Custom My Account Endpoints

```php
// Add custom endpoint
add_action( 'init', 'add_custom_endpoint' );
function add_custom_endpoint() {
    add_rewrite_endpoint( 'custom-tab', EP_ROOT | EP_PAGES );
}

// Add endpoint to menu
add_filter( 'woocommerce_account_menu_items', 'add_custom_menu_item' );
function add_custom_menu_item( $items ) {
    $items['custom-tab'] = __( 'Custom Tab', 'text-domain' );
    return $items;
}

// Display endpoint content
add_action( 'woocommerce_account_custom-tab_endpoint', 'custom_tab_content' );
function custom_tab_content() {
    echo '<h2>' . __( 'Custom Tab Content', 'text-domain' ) . '</h2>';
    echo '<p>' . __( 'This is custom content.', 'text-domain' ) . '</p>';
}

// Flush rewrite rules after adding endpoint (activation hook)
register_activation_hook( __FILE__, 'flush_rewrite_rules' );
```

### Order Details Customization

```php
// Add content before order details
add_action( 'woocommerce_order_details_before_order_table', 'add_custom_order_info' );
function add_custom_order_info( $order ) {
    $custom_field = $order->get_meta( 'custom_field' );
    if ( $custom_field ) {
        echo '<p><strong>' . __( 'Custom Info:', 'text-domain' ) . '</strong> ' . esc_html( $custom_field ) . '</p>';
    }
}
```

## Email Template Customization

### Email Template Override

**Templates:** `emails/customer-completed-order.php`, `emails/admin-new-order.php`, etc.

**Override location:**

```
theme/woocommerce/emails/customer-completed-order.php
```

### Custom Email Class

```php
// Register custom email
add_filter( 'woocommerce_email_classes', 'add_custom_email' );
function add_custom_email( $emails ) {
    $emails['WC_Email_Custom'] = new WC_Email_Custom();
    return $emails;
}

// Custom email class
class WC_Email_Custom extends WC_Email {

    public function __construct() {
        $this->id = 'custom_email';
        $this->customer_email = true;
        $this->title = __( 'Custom Email', 'text-domain' );
        $this->description = __( 'Custom email sent to customers.', 'text-domain' );

        $this->template_html = 'emails/custom-email.php';
        $this->template_plain = 'emails/plain/custom-email.php';
        $this->template_base = plugin_dir_path( __FILE__ ) . 'templates/';

        parent::__construct();
    }

    public function trigger( $order_id ) {
        $this->object = wc_get_order( $order_id );
        $this->recipient = $this->object->get_billing_email();

        if ( ! $this->is_enabled() || ! $this->get_recipient() ) {
            return;
        }

        $this->send( $this->get_recipient(), $this->get_subject(), $this->get_content(), $this->get_headers(), $this->get_attachments() );
    }

    public function get_content_html() {
        return wc_get_template_html( $this->template_html, array(
            'order' => $this->object,
            'email_heading' => $this->get_heading(),
            'sent_to_admin' => false,
            'plain_text' => false,
            'email' => $this
        ), '', $this->template_base );
    }

    public function get_content_plain() {
        return wc_get_template_html( $this->template_plain, array(
            'order' => $this->object,
            'email_heading' => $this->get_heading(),
            'sent_to_admin' => false,
            'plain_text' => true,
            'email' => $this
        ), '', $this->template_base );
    }
}

// Trigger custom email
do_action( 'woocommerce_email_custom', $order_id );
```

**Bad pattern:**

```php
// BAD: Use wp_mail() directly for order emails
wp_mail( $to, $subject, $message );

// GOOD: WC_Email subclass with template
// Provides template system, filters, settings UI
```

### Email Hook Customization

```php
// Add content before order details in emails
add_action( 'woocommerce_email_before_order_table', 'add_email_custom_content', 10, 4 );
function add_email_custom_content( $order, $sent_to_admin, $plain_text, $email ) {
    if ( $plain_text ) {
        echo __( 'Custom text content', 'text-domain' ) . "\n\n";
    } else {
        echo '<p>' . __( 'Custom HTML content', 'text-domain' ) . '</p>';
    }
}
```

## WooCommerce Blocks Compatibility

**Critical awareness:** Template overrides in `woocommerce/cart/` and `woocommerce/checkout/` do NOT affect block-based cart and checkout.

### Block Cart vs Classic Cart

```php
// Classic cart uses templates:
// - woocommerce/cart/cart.php
// - woocommerce/cart/cart-empty.php
// Template overrides work

// Block cart uses Store API:
// - Rendered via React components
// - Template overrides have NO effect
// - Use Store API extension points instead
```

**Severity:**
- **CRITICAL:** Extension relies on checkout template override but store uses block checkout
- **WARNING:** Template overrides present but block checkout is active
- **INFO:** Store could migrate to block checkout for better performance

### Detecting Block Checkout

```php
// Check if block checkout is in use
function is_block_checkout_active() {
    $checkout_page_id = wc_get_page_id( 'checkout' );
    if ( $checkout_page_id ) {
        $content = get_post_field( 'post_content', $checkout_page_id );
        return has_block( 'woocommerce/checkout', $content );
    }
    return false;
}
```

### Block Checkout Extension

For block-based checkout customization, use:
- **Additional Checkout Fields API** (WC 8.6+) - See wc-extension-guide.md
- **Store API extension points**
- **Block filters and slots**

**Not covered in depth here** - see wp-block-development skill for general block patterns.

### Migration Recommendation

```php
// For new stores: Use block checkout
// - Better performance (no cart fragments needed)
// - Modern UI (Site Editor customization)
// - Built-in validation and error handling

// For existing stores with template overrides:
// - Continue using classic checkout
// - Or migrate overrides to Additional Checkout Fields API / Store API
// - Test thoroughly before switching
```

## Template Anti-Patterns

### Pattern 1: Deleted Action Hooks

**Severity:** CRITICAL

```php
// BAD: Template override with removed do_action() calls
// In woocommerce/content-product.php

echo '<div class="product">';
echo '<img src="' . esc_url( $product->get_image_id() ) . '">';
echo '<h3>' . esc_html( get_the_title() ) . '</h3>';
echo '<span class="price">' . $product->get_price_html() . '</span>';
echo '<a href="' . esc_url( get_permalink() ) . '" class="button">View</a>';
echo '</div>';

// No do_action() calls = no extension points
// Product add-ons won't display
// Wishlist plugins can't add buttons
// Review stars missing
// Other plugins broken

// GOOD: Preserve all action hooks
echo '<div class="product">';

do_action( 'woocommerce_before_shop_loop_item' );
do_action( 'woocommerce_before_shop_loop_item_title' );

echo '<img src="' . esc_url( $product->get_image_id() ) . '">'; // Custom markup

do_action( 'woocommerce_shop_loop_item_title' );
do_action( 'woocommerce_after_shop_loop_item_title' );
do_action( 'woocommerce_after_shop_loop_item' );

echo '</div>';
```

**Detection:** Compare overridden template against WooCommerce original. Count `do_action()` calls. If override has fewer, flag as CRITICAL.

### Pattern 2: Deleted Apply_Filters() Calls

**Severity:** CRITICAL

```php
// BAD: Template override with removed apply_filters()
// In woocommerce/single-product/product-image.php

echo '<img src="' . esc_url( wp_get_attachment_url( $image_id ) ) . '" />';

// Missing: apply_filters( 'woocommerce_single_product_image_html', ... )
// Gallery plugins can't modify image output
// Zoom/lightbox plugins broken

// GOOD: Preserve filters
$image_html = sprintf( '<img src="%s" />', esc_url( wp_get_attachment_url( $image_id ) ) );
echo apply_filters( 'woocommerce_single_product_image_html', $image_html, $image_id );
```

### Pattern 3: Outdated Template Versions

**Severity:** WARNING (2+ versions behind), INFO (1 version behind)

```php
// BAD: Template override is 2+ versions behind
/**
 * Cart Page
 *
 * @version 3.8.0
 */
// Current WooCommerce version: 8.5.0
// Template is 4+ major versions behind
// Missing security patches, bug fixes, new features

// Detection: Compare @version comment against current WC version
// WooCommerce Status page shows this automatically
```

### Pattern 4: Override in Parent Theme

**Severity:** WARNING

```php
// BAD: Override in parent theme instead of child theme
// parent-theme/woocommerce/cart/cart.php
// Lost when parent theme updates

// GOOD: Override in child theme
// child-theme/woocommerce/cart/cart.php
// Survives parent theme updates
```

### Pattern 5: Override Where Hook Exists

**Severity:** INFO

```php
// INFO: Template override to add badge before title
// woocommerce/content-product.php overridden

// BETTER: Use existing hook instead
add_action( 'woocommerce_before_shop_loop_item_title', 'add_badge' );
function add_badge() {
    // Badge markup
}

// More maintainable, survives WC updates
```

### Detection Patterns

```bash
# Find template overrides
grep -r "@version" wp-content/themes/*/woocommerce/*.php

# Count do_action calls in override vs original
grep -c "do_action" wp-content/themes/my-theme/woocommerce/cart/cart.php
grep -c "do_action" wp-content/plugins/woocommerce/templates/cart/cart.php
# If override has fewer, flag as CRITICAL

# Check for outdated versions via WooCommerce Status
# WooCommerce → Status → System Status → Templates section
```

**Cross-reference:** See wp-theme-development for general template override patterns and template hierarchy.

## Cross-References

This reference cross-references other skills for related patterns:

- **wp-theme-development:** General template patterns, template hierarchy (block vs classic themes), hooks-first philosophy, child theme development
- **wp-block-development:** WooCommerce Blocks (product grid, cart, checkout), Store API integration, block-based templates
- **wp-plugin-development:** Hook architecture, action/filter priorities, Settings API for WC extensions
- **wp-performance-review:** Template performance, conditional asset loading, caching strategies

---

*WooCommerce 8.2+ | WordPress 6.x+ | Block themes and classic themes*
