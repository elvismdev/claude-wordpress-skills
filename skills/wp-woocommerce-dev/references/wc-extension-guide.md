# WooCommerce Extension Development Reference

This reference catalogs WooCommerce extension development patterns including HPOS compatibility, payment gateway development, shipping methods, custom product types, order handling, cart operations, REST API extensions, WooCommerce Blocks integration, Action Scheduler, webhook security, and coupon validation. Each pattern includes HPOS-compatible code examples following WordPress PHP Coding Standards.

## Quick Reference Table

| Pattern | API/Class | Severity if Missing | Example Use Case |
|---------|-----------|---------------------|------------------|
| **HPOS declaration** | `FeaturesUtil::declare_compatibility()` | CRITICAL | Every extension for WC 8.2+ stores |
| **Order CRUD** | `wc_get_order()`, `WC_Order` methods | CRITICAL | All order data access |
| **Product CRUD** | `wc_get_product()`, `WC_Product_Query` | WARNING | Product queries, custom fields |
| **Payment gateway** | `WC_Payment_Gateway` | CRITICAL (security) | Custom payment methods |
| **Shipping method** | `WC_Shipping_Method` | — | Custom shipping calculations |
| **Custom product type** | `WC_Product` subclass | — | Subscriptions, bookings, memberships |
| **Cart operations** | `WC()->cart` methods | — | Cart item data, fees, validation |
| **REST API endpoint** | `register_rest_route()` | WARNING (auth) | External integrations |
| **WC Blocks integration** | `woocommerce_register_additional_checkout_field()` | — | Block checkout customization |
| **Action Scheduler** | `as_schedule_single_action()` | WARNING | Background processing |
| **Webhook security** | Signature verification | CRITICAL | Webhook handlers |
| **Coupon validation** | `woocommerce_coupon_is_valid` filter | — | Custom coupon logic |

## HPOS Compatibility

**The most critical section for modern WooCommerce development.** HPOS (High-Performance Order Storage) has been the default for new stores since WooCommerce 8.2 (October 2023). Extensions not using CRUD APIs break on HPOS-enabled stores.

### Why HPOS Matters

**Performance benefits:**
- 5x faster order creation
- 40x faster order filtering
- Dedicated order tables (`wp_wc_orders`, `wp_wc_orders_meta`) instead of shared `wp_posts`
- Purpose-built indexes for order queries

**Breaking change:**
- Direct post access (`get_posts()`, `WP_Query`, `update_post_meta()`) no longer works for orders
- Extensions must use WooCommerce CRUD APIs exclusively

### Compatibility Declaration Pattern

**Required in main plugin file:**

```php
// GOOD: Declare HPOS compatibility in before_woocommerce_init hook
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
// WooCommerce Status page shows extension as incompatible
```

**Severity:** CRITICAL - Missing declaration causes silent failures on modern WC stores.

### CRUD-Only Order Access

**All order operations must use WooCommerce CRUD APIs:**

```php
// GOOD: HPOS-compatible order access
$order = wc_get_order( $order_id );
if ( ! $order ) {
    return; // Order doesn't exist
}

// Read data
$order_total = $order->get_total();
$billing_email = $order->get_billing_email();
$order_status = $order->get_status();

// BAD: Direct post access (breaks HPOS)
$order_post = get_post( $order_id );
$order_total = get_post_meta( $order_id, '_order_total', true );
```

### Order Query Pattern

```php
// GOOD: Use wc_get_orders() for order queries
$orders = wc_get_orders( array(
    'status' => 'completed',
    'limit' => 100,
    'orderby' => 'date',
    'order' => 'DESC',
    'date_created' => '>=' . ( time() - MONTH_IN_SECONDS )
) );

foreach ( $orders as $order ) {
    // Process order
}

// BAD: WP_Query for orders (breaks HPOS)
$orders = new WP_Query( array(
    'post_type' => 'shop_order',
    'post_status' => 'wc-completed',
    'posts_per_page' => 100
) );

// BAD: get_posts() for orders (breaks HPOS)
$orders = get_posts( array(
    'post_type' => 'shop_order',
    'post_status' => 'wc-completed',
    'numberposts' => 100
) );
```

**Severity:** CRITICAL - Direct post queries return empty results on HPOS stores.

### Meta Operations

**Use WC_Order methods for all meta operations:**

```php
$order = wc_get_order( $order_id );

// GOOD: CRUD meta operations
$order->update_meta_data( 'delivery_date', '2026-02-15' );
$order->add_meta_data( 'gift_message', 'Happy Birthday!', true ); // true = unique
$order->delete_meta_data( 'old_field' );
$order->save(); // Single save after all changes

// Get meta
$delivery_date = $order->get_meta( 'delivery_date', true );

// BAD: Direct meta functions (breaks HPOS)
update_post_meta( $order_id, 'delivery_date', '2026-02-15' );
$delivery_date = get_post_meta( $order_id, 'delivery_date', true );
delete_post_meta( $order_id, 'old_field' );
```

**Performance tip:** Batch all changes before calling `save()` to minimize database writes.

### Compatibility Mode Detection

**Check if HPOS is enabled for conditional logic:**

```php
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

### HPOS Migration Checklist

When migrating legacy code to HPOS compatibility:

- [ ] **Replace `get_posts()` with `wc_get_orders()`**
  - `post_type=shop_order` → `wc_get_orders()`
  - Map post status to order status (`wc-completed` → `completed`)

- [ ] **Replace `WP_Query` with `wc_get_orders()`**
  - All order queries must use CRUD API
  - Convert `WP_Query` args to `wc_get_orders()` args

- [ ] **Replace `get_post_meta()` / `update_post_meta()`**
  - `get_post_meta( $order_id, 'key' )` → `$order->get_meta( 'key' )`
  - `update_post_meta( $order_id, 'key', $value )` → `$order->update_meta_data( 'key', $value ); $order->save();`

- [ ] **Replace direct `$wpdb` queries**
  - No direct queries against `wp_posts` or `wp_postmeta` for orders
  - Use `wc_get_orders()` with appropriate filters

- [ ] **Add `declare_compatibility()` call**
  - In `before_woocommerce_init` hook
  - Check `class_exists( FeaturesUtil::class )` first

- [ ] **Test with HPOS enabled AND disabled**
  - WooCommerce Settings → Advanced → Features
  - Enable "High-Performance Order Storage"
  - Test all order operations

## Payment Gateway Development

WooCommerce payment gateways extend `WC_Payment_Gateway` class. Security is paramount—never store/log raw card data.

### Gateway Lifecycle

```php
class WC_Gateway_Custom extends WC_Payment_Gateway {

    public function __construct() {
        $this->id = 'custom_gateway'; // Unique ID
        $this->has_fields = true; // Show payment fields on checkout
        $this->method_title = __( 'Custom Gateway', 'text-domain' );
        $this->method_description = __( 'Process payments via Custom Gateway', 'text-domain' );

        // Supported features
        $this->supports = array(
            'products',
            'refunds',
            'tokenization' // Saved payment methods
        );

        $this->init_form_fields(); // Admin settings
        $this->init_settings();    // Load settings

        $this->title = $this->get_option( 'title' );
        $this->description = $this->get_option( 'description' );

        add_action( 'woocommerce_update_options_payment_gateways_' . $this->id,
            array( $this, 'process_admin_options' ) );
    }

    public function init_form_fields() {
        $this->form_fields = array(
            'enabled' => array(
                'title' => __( 'Enable/Disable', 'text-domain' ),
                'type' => 'checkbox',
                'label' => __( 'Enable Custom Gateway', 'text-domain' ),
                'default' => 'no'
            ),
            'title' => array(
                'title' => __( 'Title', 'text-domain' ),
                'type' => 'text',
                'description' => __( 'Payment method title on checkout', 'text-domain' ),
                'default' => __( 'Custom Payment', 'text-domain' ),
                'desc_tip' => true
            ),
            'api_key' => array(
                'title' => __( 'API Key', 'text-domain' ),
                'type' => 'password', // Use password type for secrets
                'description' => __( 'Get this from your gateway dashboard', 'text-domain' )
            )
        );
    }
}
```

### process_payment() Flow

**Required method for processing payments:**

```php
public function process_payment( $order_id ) {
    $order = wc_get_order( $order_id );

    // CRITICAL: Always check HTTPS in production
    if ( ! is_ssl() && 'yes' !== $this->get_option( 'testmode' ) ) {
        wc_add_notice( __( 'Payment gateway requires SSL', 'text-domain' ), 'error' );
        return array( 'result' => 'failure' );
    }

    // GOOD: Never access raw card data
    // Use tokenization or redirect to hosted payment page

    // Process payment via API
    $result = $this->api_charge( array(
        'amount' => $order->get_total(),
        'currency' => $order->get_currency(),
        'order_id' => $order->get_id()
    ) );

    if ( $result->success ) {
        // GOOD: Use payment_complete() not manual status change
        $order->payment_complete( $result->transaction_id );

        // Add order note
        $order->add_order_note(
            sprintf( __( 'Payment completed. Transaction ID: %s', 'text-domain' ), $result->transaction_id )
        );

        // Empty cart
        WC()->cart->empty_cart();

        return array(
            'result' => 'success',
            'redirect' => $this->get_return_url( $order )
        );
    }

    // Payment failed
    $order->add_order_note(
        sprintf( __( 'Payment failed: %s', 'text-domain' ), $result->error_message )
    );

    wc_add_notice( $result->error_message, 'error' );

    return array( 'result' => 'failure' );
}

// BAD: Manual status change
// $order->update_status( 'processing' ); // Don't do this
// Use payment_complete() instead - it handles status transitions correctly
```

### Refund Handling

```php
public function process_refund( $order_id, $amount = null, $reason = '' ) {
    $order = wc_get_order( $order_id );

    if ( ! $order ) {
        return new WP_Error( 'invalid_order', __( 'Invalid order', 'text-domain' ) );
    }

    $transaction_id = $order->get_transaction_id();

    if ( ! $transaction_id ) {
        return new WP_Error( 'no_transaction', __( 'No transaction ID', 'text-domain' ) );
    }

    // Process refund via API
    $result = $this->api_refund( array(
        'transaction_id' => $transaction_id,
        'amount' => $amount,
        'reason' => $reason
    ) );

    if ( $result->success ) {
        $order->add_order_note(
            sprintf( __( 'Refund of %s processed. Refund ID: %s', 'text-domain' ),
                wc_price( $amount ),
                $result->refund_id
            )
        );

        return true;
    }

    return new WP_Error( 'refund_failed', $result->error_message );
}
```

### Saved Payment Methods (Tokenization)

```php
public function __construct() {
    // Enable tokenization in constructor
    $this->supports = array( 'products', 'tokenization' );

    // Hook to save payment method
    add_action( 'woocommerce_payment_token_added_to_order', array( $this, 'add_token_to_order' ), 10, 4 );
}

public function process_payment( $order_id ) {
    $order = wc_get_order( $order_id );

    // Check if customer wants to save payment method
    $save_method = isset( $_POST['wc-' . $this->id . '-new-payment-method'] ) && ! empty( $_POST['wc-' . $this->id . '-new-payment-method'] );

    if ( $save_method && is_user_logged_in() ) {
        // Create token from gateway response
        $token = new WC_Payment_Token_CC();
        $token->set_token( $gateway_token_id ); // From gateway API
        $token->set_gateway_id( $this->id );
        $token->set_card_type( 'visa' ); // From gateway API
        $token->set_last4( '1234' ); // From gateway API
        $token->set_expiry_month( '12' );
        $token->set_expiry_year( '2030' );
        $token->set_user_id( get_current_user_id() );
        $token->save();

        // Attach token to order
        $order->add_payment_token( $token );
    }

    // Continue payment processing...
}
```

### Payment Security Anti-Patterns

**CRITICAL violations:**

```php
// CRITICAL: NEVER store raw card data
// BAD: Storing card number in database
update_post_meta( $order_id, 'card_number', $_POST['card_number'] );
$order->update_meta_data( 'card_number', $_POST['card_number'] );

// CRITICAL: NEVER log card data
// BAD: Logging payment data
error_log( 'Card number: ' . $_POST['card_number'] );
wc_get_logger()->debug( 'CVV: ' . $_POST['cvv'] );

// CRITICAL: NEVER store card data in sessions
// BAD: Session storage of card data
WC()->session->set( 'card_details', $_POST );

// CRITICAL: NEVER transmit card data over HTTP
// BAD: No SSL check before processing
public function process_payment( $order_id ) {
    // Processing payment without is_ssl() check
}
```

**WARNING violations:**

```php
// WARNING: Hardcoded API credentials
// BAD: Credentials in code
$api_key = 'sk_live_hardcoded_key';

// GOOD: Load from settings
$api_key = $this->get_option( 'api_key' );

// WARNING: Missing HTTPS check
// BAD: No SSL verification
public function process_payment( $order_id ) {
    // No is_ssl() check
}

// GOOD: Always check SSL in production
if ( ! is_ssl() && 'yes' !== $this->get_option( 'testmode' ) ) {
    wc_add_notice( __( 'SSL required', 'text-domain' ), 'error' );
    return array( 'result' => 'failure' );
}
```

**Best practices:**
- Use tokenization via gateway API (store token, not card data)
- Use hosted payment pages when possible
- Check `is_ssl()` before processing
- Load API keys from `get_option()` (not hardcoded)
- Use webhook signature verification (see Webhook Security section)

**Cross-reference:** See wp-security-review skill for general payment security patterns, capability checks, and nonce verification.

## Shipping Method Development

Custom shipping methods extend `WC_Shipping_Method` and integrate with shipping zones.

### Basic Shipping Method

```php
class WC_Shipping_Custom extends WC_Shipping_Method {

    public function __construct( $instance_id = 0 ) {
        $this->id = 'custom_shipping';
        $this->instance_id = absint( $instance_id );
        $this->method_title = __( 'Custom Shipping', 'text-domain' );
        $this->method_description = __( 'Calculate shipping based on custom rules', 'text-domain' );

        // Enable shipping zones and instance settings
        $this->supports = array(
            'shipping-zones',
            'instance-settings',
            'instance-settings-modal'
        );

        $this->init();
    }

    public function init() {
        $this->init_form_fields();
        $this->init_settings();

        $this->enabled = $this->get_option( 'enabled' );
        $this->title = $this->get_option( 'title' );

        add_action( 'woocommerce_update_options_shipping_' . $this->id, array( $this, 'process_admin_options' ) );
    }

    public function init_form_fields() {
        $this->instance_form_fields = array(
            'enabled' => array(
                'title' => __( 'Enable/Disable', 'text-domain' ),
                'type' => 'checkbox',
                'label' => __( 'Enable this shipping method', 'text-domain' ),
                'default' => 'yes'
            ),
            'title' => array(
                'title' => __( 'Method Title', 'text-domain' ),
                'type' => 'text',
                'description' => __( 'Title shown to customers', 'text-domain' ),
                'default' => __( 'Custom Shipping', 'text-domain' ),
                'desc_tip' => true
            ),
            'cost' => array(
                'title' => __( 'Cost', 'text-domain' ),
                'type' => 'text',
                'description' => __( 'Base shipping cost', 'text-domain' ),
                'default' => '10',
                'desc_tip' => true
            )
        );
    }
}
```

### calculate_shipping() Implementation

```php
public function calculate_shipping( $package = array() ) {
    // Access package data
    $destination = $package['destination'];
    $contents = $package['contents'];
    $contents_cost = $package['contents_cost'];

    // Calculate cost based on package
    $cost = $this->get_option( 'cost' );

    // Add per-item cost
    foreach ( $contents as $item_id => $values ) {
        $product = $values['data'];
        $cost += $product->get_weight() * 2; // Example: $2 per kg
    }

    // Create rate
    $rate = array(
        'id' => $this->get_rate_id(),
        'label' => $this->title,
        'cost' => $cost,
        'calc_tax' => 'per_order', // or 'per_item'
        'meta_data' => array(
            'estimated_delivery' => '3-5 business days'
        )
    );

    // Add the rate
    $this->add_rate( $rate );
}

// BAD: Hardcoded shipping cost without zone support
public function calculate_shipping( $package = array() ) {
    $this->add_rate( array(
        'id' => $this->id,
        'label' => 'Shipping',
        'cost' => 10 // No calculation, no zone awareness
    ) );
}
```

### Shipping Class Support

```php
public function calculate_shipping( $package = array() ) {
    $cost = 0;

    foreach ( $package['contents'] as $item_id => $values ) {
        $product = $values['data'];
        $shipping_class_id = $product->get_shipping_class_id();

        // Different rates per shipping class
        if ( $shipping_class_id === 123 ) {
            $cost += 5 * $values['quantity'];
        } elseif ( $shipping_class_id === 456 ) {
            $cost += 10 * $values['quantity'];
        } else {
            $cost += 2 * $values['quantity'];
        }
    }

    $this->add_rate( array(
        'id' => $this->get_rate_id(),
        'label' => $this->title,
        'cost' => $cost
    ) );
}
```

## Custom Product Types

Custom product types extend the `WC_Product` hierarchy and integrate with WooCommerce product management.

### Product Type Registration

```php
// Register custom product type
add_filter( 'product_type_selector', 'add_custom_product_type' );
function add_custom_product_type( $types ) {
    $types['custom'] = __( 'Custom Product', 'text-domain' );
    return $types;
}

// Register product class
add_filter( 'woocommerce_product_class', 'load_custom_product_class', 10, 2 );
function load_custom_product_class( $classname, $product_type ) {
    if ( 'custom' === $product_type ) {
        $classname = 'WC_Product_Custom';
    }
    return $classname;
}
```

### Custom Product Class

```php
class WC_Product_Custom extends WC_Product {

    public function __construct( $product = 0 ) {
        $this->product_type = 'custom';
        parent::__construct( $product );
    }

    public function get_type() {
        return 'custom';
    }

    // Override methods as needed
    public function is_purchasable() {
        // Custom purchasable logic
        return true;
    }

    public function add_to_cart_url() {
        // Custom add-to-cart URL
        return apply_filters( 'woocommerce_product_add_to_cart_url', get_permalink( $this->get_id() ), $this );
    }
}
```

### Custom Product Data Tab

```php
// Add custom tab to product data metabox
add_filter( 'woocommerce_product_data_tabs', 'add_custom_product_data_tab' );
function add_custom_product_data_tab( $tabs ) {
    $tabs['custom'] = array(
        'label' => __( 'Custom Data', 'text-domain' ),
        'target' => 'custom_product_data',
        'class' => array( 'show_if_custom' )
    );
    return $tabs;
}

// Display custom fields
add_action( 'woocommerce_product_data_panels', 'custom_product_data_fields' );
function custom_product_data_fields() {
    global $post;
    ?>
    <div id="custom_product_data" class="panel woocommerce_options_panel hidden">
        <?php
        woocommerce_wp_text_input( array(
            'id' => '_custom_field',
            'label' => __( 'Custom Field', 'text-domain' ),
            'desc_tip' => true,
            'description' => __( 'Enter custom data', 'text-domain' )
        ) );
        ?>
    </div>
    <?php
}

// Save custom fields
add_action( 'woocommerce_process_product_meta', 'save_custom_product_data' );
function save_custom_product_data( $post_id ) {
    $product = wc_get_product( $post_id );

    if ( isset( $_POST['_custom_field'] ) ) {
        $product->update_meta_data( '_custom_field', sanitize_text_field( $_POST['_custom_field'] ) );
        $product->save();
    }
}
```

**Anti-pattern:**

```php
// BAD: Custom post type for products
register_post_type( 'custom_product', array( /* ... */ ) );

// GOOD: Extend WC_Product with proper data store
class WC_Product_Custom extends WC_Product {
    // Custom product logic
}
```

## Order Handling

All order operations must use WooCommerce CRUD APIs for HPOS compatibility.

### Order CRUD Operations

```php
// Create new order
$order = wc_create_order();
$order->add_product( wc_get_product( $product_id ), $quantity );
$order->set_address( $billing_address, 'billing' );
$order->set_address( $shipping_address, 'shipping' );
$order->calculate_totals();
$order->save();

// Read order data
$order = wc_get_order( $order_id );
$order_total = $order->get_total();
$billing_email = $order->get_billing_email();
$items = $order->get_items();

// Update order
$order->set_billing_email( 'new@example.com' );
$order->update_meta_data( 'custom_field', 'value' );
$order->save();

// Delete order (use with caution)
$order->delete( true ); // true = force delete
```

### Status Transitions

```php
// Get available statuses
$statuses = wc_get_order_statuses();
// Returns: array( 'wc-pending' => 'Pending payment', 'wc-processing' => 'Processing', ... )

// Change order status
$order = wc_get_order( $order_id );
$order->set_status( 'processing', 'Order is being prepared' ); // Status + note
// Don't call save() - set_status() saves automatically

// Hook into status changes
add_action( 'woocommerce_order_status_changed', 'handle_order_status_change', 10, 4 );
function handle_order_status_change( $order_id, $old_status, $new_status, $order ) {
    if ( 'processing' === $new_status ) {
        // Handle new processing orders
    }
}
```

### Custom Order Statuses

```php
// Register custom order status
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
```

### Order Items Manipulation

```php
$order = wc_get_order( $order_id );

// Add product to order
$product = wc_get_product( $product_id );
$item_id = $order->add_product( $product, $quantity, array(
    'subtotal' => $product->get_price() * $quantity,
    'total' => $product->get_price() * $quantity
) );

// Add fee
$item_id = $order->add_fee( array(
    'name' => 'Custom Fee',
    'amount' => 10,
    'taxable' => true,
    'tax_class' => ''
) );

// Add shipping
$shipping = new WC_Order_Item_Shipping();
$shipping->set_method_title( 'Custom Shipping' );
$shipping->set_method_id( 'custom_shipping' );
$shipping->set_total( 15 );
$order->add_item( $shipping );

// Update order items
foreach ( $order->get_items() as $item_id => $item ) {
    $item->update_meta_data( 'custom_data', 'value' );
    $item->save();
}

// Recalculate totals after modifications
$order->calculate_totals();
$order->save();
```

### Order Notes

```php
$order = wc_get_order( $order_id );

// Add customer note (visible to customer)
$order->add_order_note(
    __( 'Your order is being prepared for shipment.', 'text-domain' ),
    true // true = customer note, false = internal note
);

// Add internal note (admin only)
$order->add_order_note(
    __( 'Payment verified manually.', 'text-domain' ),
    false // Internal note
);

// Add system note (generated by extension)
$order->add_order_note(
    sprintf( __( 'Inventory synced. SKU: %s', 'text-domain' ), $sku ),
    false,
    true // true = added by system/extension
);
```

### Batch Operations Performance

```php
// GOOD: Batch operations with single save
$order = wc_get_order( $order_id );
$order->set_billing_email( 'new@example.com' );
$order->update_meta_data( 'field1', 'value1' );
$order->update_meta_data( 'field2', 'value2' );
$order->update_meta_data( 'field3', 'value3' );
$order->save(); // Single save for all changes

// BAD: Multiple save calls
$order = wc_get_order( $order_id );
$order->update_meta_data( 'field1', 'value1' );
$order->save(); // Unnecessary save
$order->update_meta_data( 'field2', 'value2' );
$order->save(); // Unnecessary save
$order->update_meta_data( 'field3', 'value3' );
$order->save();

// BAD: Individual update_post_meta() calls (also breaks HPOS)
update_post_meta( $order_id, 'field1', 'value1' );
update_post_meta( $order_id, 'field2', 'value2' );
update_post_meta( $order_id, 'field3', 'value3' );
```

## Cart Operations

Cart operations use the global `WC()->cart` object.

### Cart Methods

```php
// Add product to cart
WC()->cart->add_to_cart( $product_id, $quantity, $variation_id, $variation, $cart_item_data );

// Get cart contents
$cart_items = WC()->cart->get_cart();
foreach ( $cart_items as $cart_item_key => $cart_item ) {
    $product = $cart_item['data'];
    $quantity = $cart_item['quantity'];
}

// Get cart totals
$subtotal = WC()->cart->get_cart_subtotal();
$total = WC()->cart->get_total();
$item_count = WC()->cart->get_cart_contents_count();

// Update cart item quantity
WC()->cart->set_quantity( $cart_item_key, $new_quantity );

// Remove cart item
WC()->cart->remove_cart_item( $cart_item_key );

// Empty cart
WC()->cart->empty_cart();

// Calculate totals (call after modifying cart)
WC()->cart->calculate_totals();
```

### Custom Cart Item Data

```php
// Add custom data to cart items
add_filter( 'woocommerce_add_cart_item_data', 'add_custom_cart_item_data', 10, 3 );
function add_custom_cart_item_data( $cart_item_data, $product_id, $variation_id ) {
    if ( isset( $_POST['custom_field'] ) ) {
        $cart_item_data['custom_field'] = sanitize_text_field( $_POST['custom_field'] );
    }
    return $cart_item_data;
}

// Display custom data in cart
add_filter( 'woocommerce_get_item_data', 'display_custom_cart_item_data', 10, 2 );
function display_custom_cart_item_data( $item_data, $cart_item ) {
    if ( isset( $cart_item['custom_field'] ) ) {
        $item_data[] = array(
            'key' => __( 'Custom Field', 'text-domain' ),
            'value' => $cart_item['custom_field'],
            'display' => ''
        );
    }
    return $item_data;
}

// Save custom data to order item meta
add_action( 'woocommerce_checkout_create_order_line_item', 'save_custom_order_item_meta', 10, 4 );
function save_custom_order_item_meta( $item, $cart_item_key, $values, $order ) {
    if ( isset( $values['custom_field'] ) ) {
        $item->update_meta_data( 'Custom Field', $values['custom_field'] );
    }
}
```

### Cart Fees

```php
// Add fees to cart
add_action( 'woocommerce_cart_calculate_fees', 'add_custom_cart_fee' );
function add_custom_cart_fee() {
    if ( is_admin() && ! defined( 'DOING_AJAX' ) ) {
        return;
    }

    $percentage = 0.05; // 5% fee
    $fee = WC()->cart->get_subtotal() * $percentage;

    WC()->cart->add_fee( __( 'Service Fee', 'text-domain' ), $fee, true ); // true = taxable
}
```

### Cart Validation

```php
// Validate cart items
add_action( 'woocommerce_check_cart_items', 'validate_cart_items' );
function validate_cart_items() {
    foreach ( WC()->cart->get_cart() as $cart_item_key => $cart_item ) {
        $product = $cart_item['data'];

        if ( $cart_item['quantity'] > 5 ) {
            wc_add_notice(
                sprintf( __( 'Maximum 5 units of %s allowed', 'text-domain' ), $product->get_name() ),
                'error'
            );
        }
    }
}

// Validate add-to-cart
add_filter( 'woocommerce_add_to_cart_validation', 'validate_add_to_cart', 10, 3 );
function validate_add_to_cart( $passed, $product_id, $quantity ) {
    if ( $quantity > 10 ) {
        wc_add_notice( __( 'Maximum 10 units allowed per order', 'text-domain' ), 'error' );
        $passed = false;
    }
    return $passed;
}
```

### AJAX Add-to-Cart Security

```php
// Custom AJAX add-to-cart handler
add_action( 'wp_ajax_custom_add_to_cart', 'handle_custom_add_to_cart' );
add_action( 'wp_ajax_nopriv_custom_add_to_cart', 'handle_custom_add_to_cart' );
function handle_custom_add_to_cart() {
    // CRITICAL: Verify nonce
    check_ajax_referer( 'custom-add-to-cart-nonce', 'security' );

    $product_id = absint( $_POST['product_id'] );
    $quantity = absint( $_POST['quantity'] );

    if ( $product_id && $quantity ) {
        WC()->cart->add_to_cart( $product_id, $quantity );
        wp_send_json_success();
    }

    wp_send_json_error();
}

// BAD: No nonce verification
function handle_custom_add_to_cart() {
    // Direct processing without nonce check
    $product_id = $_POST['product_id'];
    WC()->cart->add_to_cart( $product_id );
}
```

**Cross-reference:** See wp-security-review for nonce verification patterns and AJAX security.

## WooCommerce REST API Extensions

WooCommerce provides REST API v3 at `/wp-json/wc/v3/` endpoints. Surface-level coverage only.

### Custom Endpoint Registration

```php
add_filter( 'woocommerce_rest_api_get_rest_namespaces', 'add_custom_rest_endpoints' );
function add_custom_rest_endpoints( $namespaces ) {
    $namespaces['wc/v3']['custom'] = 'My_Custom_REST_Controller';
    return $namespaces;
}

class My_Custom_REST_Controller extends WC_REST_Controller {
    protected $namespace = 'wc/v3';
    protected $rest_base = 'custom';

    public function register_routes() {
        register_rest_route( $this->namespace, '/' . $this->rest_base, array(
            array(
                'methods' => WP_REST_Server::READABLE,
                'callback' => array( $this, 'get_items' ),
                'permission_callback' => array( $this, 'get_items_permissions_check' )
            )
        ) );
    }

    public function get_items_permissions_check( $request ) {
        // CRITICAL: Always implement permission checks
        if ( ! current_user_can( 'manage_woocommerce' ) ) {
            return new WP_Error( 'woocommerce_rest_cannot_view', __( 'Unauthorized', 'text-domain' ), array( 'status' => 403 ) );
        }
        return true;
    }

    public function get_items( $request ) {
        // Return custom data
        return rest_ensure_response( array( 'data' => 'Custom endpoint data' ) );
    }
}
```

### Authentication

WooCommerce REST API supports:
- **Basic Auth** (development only - requires username/password)
- **OAuth 1.0a** (production - consumer key/secret)
- **HTTPS enforcement** (required for Basic Auth)

**Cross-reference:** See wp-plugin-development for REST API patterns and wp-security-review for permission_callback requirements.

## WooCommerce Blocks Integration

Block-based cart and checkout bypass traditional template overrides. Surface-level coverage only.

### Additional Checkout Fields API (WC 8.6+)

```php
add_action( 'woocommerce_init', 'register_custom_checkout_field' );
function register_custom_checkout_field() {
    woocommerce_register_additional_checkout_field( array(
        'id' => 'namespace/gift-message',
        'label' => __( 'Gift message', 'text-domain' ),
        'location' => 'order',
        'type' => 'text',
        'attributes' => array(
            'maxLength' => 200,
            'pattern' => '[a-zA-Z0-9 ]+' // Optional validation pattern
        ),
        'show_in_confirmation_email' => true
    ) );
}

// Access field value after order
$order = wc_get_order( $order_id );
$gift_message = $order->get_meta( 'namespace/gift-message' );
```

### Block Compatibility Declaration

```php
add_action( 'before_woocommerce_init', function() {
    if ( class_exists( '\Automattic\WooCommerce\Utilities\FeaturesUtil' ) ) {
        \Automattic\WooCommerce\Utilities\FeaturesUtil::declare_compatibility(
            'cart_checkout_blocks',
            __FILE__,
            true
        );
    }
} );
```

**Note:** Template overrides in `woocommerce/cart/` and `woocommerce/checkout/` do NOT affect block cart/checkout. Use Store API and Additional Checkout Fields API instead.

**Cross-reference:** See wp-block-development for general block patterns and Store API integration.

## Action Scheduler Patterns

Action Scheduler ships with WooCommerce and provides reliable background processing.

### Why Action Scheduler Over wp_cron()

**Action Scheduler benefits:**
- Reliable (not traffic-dependent)
- Retry logic (failed actions retry automatically)
- Admin UI (WooCommerce → Status → Scheduled Actions)
- Queue management (view pending/in-progress/complete)
- Traceable (logs all executions)

### Single Action Scheduling

```php
// Schedule action for specific timestamp
as_schedule_single_action(
    time() + HOUR_IN_SECONDS,
    'process_order_export',
    array( 'order_id' => 123 ),
    'wc-exports' // Group name for organization
);

// Schedule immediate background processing
as_enqueue_async_action(
    'process_webhook',
    array( 'payload' => $data ),
    'wc-webhooks'
);

// Hook to process action
add_action( 'process_order_export', 'do_order_export', 10, 1 );
function do_order_export( $order_id ) {
    $order = wc_get_order( $order_id );
    // Export logic
}
```

### Recurring Actions

```php
// Schedule recurring action (replaces wp_schedule_event)
as_schedule_recurring_action(
    time(),
    HOUR_IN_SECONDS,
    'sync_inventory',
    array(),
    'wc-sync'
);

add_action( 'sync_inventory', 'do_inventory_sync' );
function do_inventory_sync() {
    // Sync logic
}

// BAD: wp_cron for WooCommerce background tasks
wp_schedule_event( time(), 'hourly', 'sync_inventory' );
// Unreliable - depends on site traffic
// No retry logic
// No admin UI for monitoring
```

**Severity:** WARNING - wp_cron() for bulk operations (order processing, inventory sync, report generation) should use Action Scheduler.

### Batch Processing Pattern

```php
// Initial trigger (e.g., user clicks "Export All Orders")
function start_bulk_order_export() {
    as_enqueue_async_action(
        'process_order_export_batch',
        array( 'offset' => 0 ),
        'wc-exports'
    );
}

// Process batch
add_action( 'process_order_export_batch', 'process_export_batch', 10, 1 );
function process_export_batch( $offset ) {
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
            'process_order_export_batch',
            array( 'offset' => $offset + $batch_size ),
            'wc-exports'
        );
    }
}
```

### Migration from wp_cron()

```php
// OLD: wp_cron
wp_schedule_event( time(), 'hourly', 'my_hourly_task' );
add_action( 'my_hourly_task', 'do_task' );

// NEW: Action Scheduler
as_schedule_recurring_action( time(), HOUR_IN_SECONDS, 'my_hourly_task', array(), 'my-group' );
add_action( 'my_hourly_task', 'do_task' );

// OLD: Single event
wp_schedule_single_event( time() + 3600, 'my_delayed_task', array( 'arg' => 123 ) );

// NEW: Action Scheduler
as_schedule_single_action( time() + 3600, 'my_delayed_task', array( 'arg' => 123 ), 'my-group' );
```

## Webhook Security

WooCommerce webhooks require signature verification to prevent forged requests.

### Signature Verification

```php
// Webhook handler endpoint
add_action( 'rest_api_init', 'register_webhook_endpoint' );
function register_webhook_endpoint() {
    register_rest_route( 'myextension/v1', '/webhook', array(
        'methods' => 'POST',
        'callback' => 'handle_webhook',
        'permission_callback' => '__return_true' // Public endpoint
    ) );
}

function handle_webhook( WP_REST_Request $request ) {
    $payload = $request->get_body();
    $signature = $request->get_header( 'X-WC-Webhook-Signature' );
    $delivery_id = $request->get_header( 'X-WC-Webhook-Delivery-ID' );
    $secret = get_option( 'webhook_secret' ); // Store secret in settings

    // CRITICAL: Verify signature
    $expected = base64_encode( hash_hmac( 'sha256', $payload, $secret, true ) );

    // CRITICAL: Use hash_equals to prevent timing attacks
    if ( ! hash_equals( $expected, $signature ) ) {
        return new WP_Error( 'invalid_signature', 'Invalid webhook signature', array( 'status' => 401 ) );
    }

    // GOOD: Check for duplicate delivery
    if ( get_transient( 'webhook_delivery_' . $delivery_id ) ) {
        // Already processed
        return rest_ensure_response( array( 'status' => 'already_processed' ) );
    }
    set_transient( 'webhook_delivery_' . $delivery_id, true, DAY_IN_SECONDS );

    // GOOD: Process async via Action Scheduler
    as_enqueue_async_action(
        'process_webhook_data',
        array( 'payload' => $payload ),
        'wc-webhooks'
    );

    // Respond immediately
    return rest_ensure_response( array( 'status' => 'received' ) );
}

// Process webhook asynchronously
add_action( 'process_webhook_data', 'do_process_webhook', 10, 1 );
function do_process_webhook( $payload ) {
    $data = json_decode( $payload, true );
    // Process webhook data
}

// BAD: Processing webhook without verification
function handle_webhook( WP_REST_Request $request ) {
    $data = $request->get_json_params();
    // Direct processing without signature check
    // Allows forged webhooks
}
```

**Severity:** CRITICAL - No webhook signature verification allows forged requests that could modify orders, trigger refunds, or update inventory.

## Coupon/Discount Validation

Surface-level coverage of coupon validation patterns.

### Custom Coupon Validation

```php
add_filter( 'woocommerce_coupon_is_valid', 'validate_custom_coupon_logic', 10, 3 );
function validate_custom_coupon_logic( $valid, $coupon, $discount ) {

    // Example: Require minimum 3 items in cart
    if ( WC()->cart->get_cart_contents_count() < 3 ) {
        throw new Exception(
            __( 'This coupon requires at least 3 items in cart.', 'text-domain' ),
            109 // Error code
        );
    }

    // Example: Coupon only valid on Tuesdays
    if ( 2 !== (int) date( 'N' ) ) {
        throw new Exception(
            __( 'This coupon is only valid on Tuesdays.', 'text-domain' ),
            110
        );
    }

    return $valid;
}
```

### Custom Coupon Type

```php
add_filter( 'woocommerce_coupon_discount_types', 'add_custom_coupon_type' );
function add_custom_coupon_type( $types ) {
    $types['custom_discount'] = __( 'Custom Discount', 'text-domain' );
    return $types;
}

// Calculate custom discount
add_filter( 'woocommerce_coupon_get_discount_amount', 'calculate_custom_discount', 10, 5 );
function calculate_custom_discount( $discount, $discounting_amount, $cart_item, $single, $coupon ) {
    if ( 'custom_discount' !== $coupon->get_discount_type() ) {
        return $discount;
    }

    // Custom discount calculation
    $discount = $discounting_amount * 0.15; // 15% discount

    return $discount;
}
```

**Cross-reference:** See wp-security-review for coupon validation security patterns (capability checks, nonce verification).

## Cross-References

This reference cross-references other skills for related patterns:

- **wp-security-review:** Payment data handling (never store raw cards), REST API authentication, AJAX nonce verification, capability checks in order management, webhook security
- **wp-plugin-development:** WC as plugin extension (dependency checking, activation/deactivation), hooks architecture, REST API endpoint registration, Settings API
- **wp-block-development:** WooCommerce Blocks (product grid, cart, checkout), Store API integration, block-based checkout extensions
- **wp-theme-development:** WooCommerce template overrides in themes (woocommerce/ directory), shop page customization, product display templates
- **wp-performance-review:** Cart fragments optimization, WC_Product_Query vs WP_Query, session table performance, Action Scheduler vs wp_cron

---

*WooCommerce 8.2+ with HPOS default | WordPress 6.x+ | PHP 7.4+*
