# WordPress Hooks System Reference

This reference provides comprehensive coverage of WordPress actions and filters for plugin development. Includes lifecycle, priority system, hook removal, custom hooks, and common patterns for WordPress 6.x+. All examples follow WordPress PHP Coding Standards.

## Quick Reference Table

| Function | Purpose | Parameters | Returns |
|----------|---------|------------|---------|
| `add_action()` | Register action callback | `$hook_name, $callback, $priority = 10, $accepted_args = 1` | `true` |
| `add_filter()` | Register filter callback | `$hook_name, $callback, $priority = 10, $accepted_args = 1` | `true` |
| `remove_action()` | Unregister action | `$hook_name, $callback, $priority = 10` | `bool` |
| `remove_filter()` | Unregister filter | `$hook_name, $callback, $priority = 10` | `bool` |
| `do_action()` | Fire custom action | `$hook_name, ...$args` | `void` |
| `do_action_ref_array()` | Fire action with array args | `$hook_name, $args` | `void` |
| `apply_filters()` | Fire custom filter | `$hook_name, $value, ...$args` | `mixed` |
| `apply_filters_ref_array()` | Fire filter with array args | `$hook_name, $args` | `mixed` |
| `has_action()` | Check if action registered | `$hook_name, $callback = false` | `bool\|int` |
| `has_filter()` | Check if filter registered | `$hook_name, $callback = false` | `bool\|int` |
| `did_action()` | Count how many times action fired | `$hook_name` | `int` |
| `current_action()` | Get current action name | None | `string` |
| `current_filter()` | Get current filter name | None | `string` |
| `doing_action()` | Check if action is currently running | `$hook_name = null` | `bool` |
| `doing_filter()` | Check if filter is currently running | `$hook_name = null` | `bool` |

## Actions vs Filters

### Fundamental Distinction

**Actions:** DO something. Trigger side effects. Don't return a value.

**Filters:** MODIFY something. Transform data. MUST return a value.

```php
// ❌ BAD: Action returning a value (ignored by WordPress)
add_action( 'save_post', 'mypl_save_metadata' );

function mypl_save_metadata( $post_id ) {
	update_post_meta( $post_id, 'mypl_processed', true );
	return $post_id; // ❌ Return value is ignored
}

// ✅ GOOD: Action performs side effect
add_action( 'save_post', 'mypl_save_metadata' );

function mypl_save_metadata( $post_id ) {
	update_post_meta( $post_id, 'mypl_processed', true );
	// No return needed
}

// ❌ CRITICAL: Filter without return (fatal error)
add_filter( 'the_content', 'mypl_add_footer' );

function mypl_add_footer( $content ) {
	if ( is_single() ) {
		$content .= '<p>Footer text</p>';
	}
	// ❌ Missing return - breaks filter chain
}

// ✅ GOOD: Filter ALWAYS returns value
add_filter( 'the_content', 'mypl_add_footer' );

function mypl_add_footer( $content ) {
	if ( is_single() ) {
		$content .= '<p>Footer text</p>';
	}
	return $content; // CRITICAL: Always return
}

// ✅ GOOD: Even if no changes, return original
add_filter( 'the_title', 'mypl_modify_title' );

function mypl_modify_title( $title ) {
	if ( ! is_admin() ) {
		// Modify only on frontend
		$title = strtoupper( $title );
	}
	// Always return, even if unchanged
	return $title;
}
```

**When to use which:**

| Use Case | Hook Type | Example |
|----------|-----------|---------|
| Send email notification | Action | `do_action( 'mypl_order_complete', $order_id )` |
| Save metadata | Action | `save_post` action |
| Log event | Action | `wp_login` action |
| Enqueue scripts | Action | `wp_enqueue_scripts` action |
| Modify post content | Filter | `the_content` filter |
| Change email subject | Filter | `wp_mail` filter |
| Adjust query parameters | Filter | `pre_get_posts` filter |
| Sanitize user input | Filter | Custom sanitize filter |

## Hook Lifecycle

### Complete WordPress Execution Order

**Early lifecycle (before content):**

```php
// 1. muplugins_loaded - Must-use plugins loaded
add_action( 'muplugins_loaded', 'mypl_mu_loaded' ); // Earliest hook

// 2. registered_taxonomy - After each taxonomy is registered
add_action( 'registered_taxonomy', 'mypl_after_taxonomy_registered', 10, 3 );

// 3. registered_post_type - After each post type is registered
add_action( 'registered_post_type', 'mypl_after_cpt_registered', 10, 2 );

// 4. plugins_loaded - All plugins loaded
add_action( 'plugins_loaded', 'mypl_plugins_loaded' ); // Good for checking other plugins

// 5. set_current_user - User loaded
add_action( 'set_current_user', 'mypl_user_loaded' );

// 6. init - Initialize plugin features
add_action( 'init', 'mypl_register_post_types' ); // Most common hook

// 7. widgets_init - Register widgets
add_action( 'widgets_init', 'mypl_register_widgets' );

// 8. wp_loaded - WordPress fully loaded
add_action( 'wp_loaded', 'mypl_wp_loaded' ); // All files loaded, user authenticated
```

**Admin vs frontend split:**

```php
// ADMIN CONTEXT
if ( is_admin() ) {
	// 9a. admin_menu - Add admin menus
	add_action( 'admin_menu', 'mypl_add_admin_menu' );

	// 10a. admin_init - Admin initialization
	add_action( 'admin_init', 'mypl_register_settings' );

	// 11a. admin_enqueue_scripts - Admin assets
	add_action( 'admin_enqueue_scripts', 'mypl_enqueue_admin_assets' );
}

// FRONTEND CONTEXT
if ( ! is_admin() ) {
	// 9b. template_redirect - Before template loaded
	add_action( 'template_redirect', 'mypl_redirect_logic' );

	// 10b. wp_enqueue_scripts - Frontend assets
	add_action( 'wp_enqueue_scripts', 'mypl_enqueue_public_assets' );

	// 11b. wp_head - Output in <head>
	add_action( 'wp_head', 'mypl_add_meta_tags' );

	// 12b. wp_footer - Output before </body>
	add_action( 'wp_footer', 'mypl_footer_scripts' );
}
```

**REST API context:**

```php
// REST API requests
add_action( 'rest_api_init', 'mypl_register_rest_routes' );
```

**Content filters (render time):**

```php
// the_content filter runs when post content is displayed
add_filter( 'the_content', 'mypl_modify_content' );

// the_title filter runs when post title is displayed
add_filter( 'the_title', 'mypl_modify_title', 10, 2 );
```

**Late lifecycle (after content):**

```php
// wp_footer action - End of page
add_action( 'wp_footer', 'mypl_footer_code' );

// shutdown - Very last hook
add_action( 'shutdown', 'mypl_cleanup' ); // For logging, cleanup
```

### Hook Timing Mistakes

```php
// ❌ BAD: Registering CPT too early (before 'init')
add_action( 'plugins_loaded', 'mypl_register_post_type' );

function mypl_register_post_type() {
	register_post_type( 'mypl_book', array() ); // Too early!
}

// ✅ GOOD: Register CPT on 'init' or later
add_action( 'init', 'mypl_register_post_type' );

// ❌ BAD: Enqueuing scripts on 'init' (too early, won't work)
add_action( 'init', 'mypl_enqueue_scripts' );

function mypl_enqueue_scripts() {
	wp_enqueue_script( 'mypl-script', MYPL_URL . 'js/script.js' ); // Too early!
}

// ✅ GOOD: Enqueue on proper hook
add_action( 'wp_enqueue_scripts', 'mypl_enqueue_scripts' ); // Frontend
add_action( 'admin_enqueue_scripts', 'mypl_enqueue_admin_scripts' ); // Admin

// ❌ BAD: Settings API on 'init' (wrong hook)
add_action( 'init', 'mypl_register_settings' );

// ✅ GOOD: Settings API on 'admin_init'
add_action( 'admin_init', 'mypl_register_settings' );
```

## Priority System Deep Dive

### How Priority Works

**Priority:** Lower number = earlier execution. Default is 10.

```php
// Priority 5 runs BEFORE core (core uses 10)
add_action( 'init', 'mypl_early', 5 );

// Priority 10 runs WITH core (default)
add_action( 'init', 'mypl_default' ); // Same as priority 10

// Priority 15 runs AFTER core
add_action( 'init', 'mypl_late', 15 );

// Priority 999 runs very late
add_action( 'init', 'mypl_very_late', 999 );
```

**Execution order:**

```
Priority 5  → mypl_early()
Priority 10 → core_function()
Priority 10 → mypl_default() (registered after core, same priority)
Priority 15 → mypl_late()
Priority 999 → mypl_very_late()
```

### When to Use Different Priorities

**Lower priority (< 10): Run BEFORE core**

```php
// Register CPT before core processes them
add_action( 'init', 'mypl_register_cpt', 9 );

// Modify query before core query runs
add_action( 'pre_get_posts', 'mypl_modify_query', 5 );

// Load textdomain early
add_action( 'init', 'mypl_load_textdomain', 1 );
```

**Default priority (10): Run WITH core**

```php
// Most plugin code can use default
add_action( 'init', 'mypl_init_plugin' ); // Priority 10
add_filter( 'the_content', 'mypl_modify_content' ); // Priority 10
```

**Higher priority (> 10): Run AFTER core**

```php
// Override core filters
add_filter( 'the_content', 'mypl_final_content_changes', 20 );

// Ensure scripts load after dependencies
add_action( 'wp_enqueue_scripts', 'mypl_enqueue_last', 999 );

// Run after all other plugins
add_action( 'init', 'mypl_after_everything', 999 );
```

### Priority Anti-Patterns

```php
// ❌ BAD: Relying on default priority for order-dependent code
add_filter( 'the_content', 'mypl_add_wrapper' ); // Needs to run last
add_filter( 'the_content', 'mypl_add_header' );  // Needs to run first
// Order depends on registration sequence - fragile!

// ✅ GOOD: Explicit priorities for order dependencies
add_filter( 'the_content', 'mypl_add_header', 5 );  // Runs first
add_filter( 'the_content', 'mypl_add_wrapper', 15 ); // Runs last

// ❌ BAD: Using extremely low priority when not needed
add_action( 'init', 'mypl_simple_function', -999 ); // Unnecessarily early

// ✅ GOOD: Use default unless you need specific timing
add_action( 'init', 'mypl_simple_function' ); // Priority 10
```

## Accepted Args

### How Accepted Args Works

**Parameter 4 of add_action/add_filter:** Tells WordPress how many parameters to pass to callback.

```php
// ❌ BAD: Not declaring enough accepted_args
add_filter( 'the_title', 'mypl_modify_title' ); // Defaults to 1 arg

function mypl_modify_title( $title, $post_id ) {
	// $post_id is undefined! Only $title passed.
	if ( 123 === $post_id ) { // Fatal: undefined variable
		return 'Modified: ' . $title;
	}
	return $title;
}

// ✅ GOOD: Declare correct number of accepted_args
add_filter( 'the_title', 'mypl_modify_title', 10, 2 ); // 2 args

function mypl_modify_title( $title, $post_id ) {
	// Both parameters available
	if ( 123 === $post_id ) {
		return 'Modified: ' . $title;
	}
	return $title;
}

// ✅ GOOD: Function signature matches accepted_args
add_action( 'save_post', 'mypl_on_save_post', 10, 3 );

function mypl_on_save_post( $post_id, $post, $update ) {
	// All 3 parameters available
	if ( $update ) {
		// Existing post updated
	} else {
		// New post created
	}
}
```

**Finding accepted args count:**

Check WordPress documentation for hook. Example from [the_title filter docs](https://developer.wordpress.org/reference/hooks/the_title/):

```php
apply_filters( 'the_title', string $title, int $post_id )
```

Two parameters after hook name = 2 accepted_args.

### Common Hooks with Multiple Args

| Hook | Accepted Args | Parameters |
|------|---------------|------------|
| `save_post` | 3 | `$post_id, $post, $update` |
| `the_title` | 2 | `$title, $post_id` |
| `the_content` | 1 | `$content` |
| `wp_mail` | 1 | `$args` (array) |
| `pre_get_posts` | 1 | `$query` (object) |
| `admin_enqueue_scripts` | 1 | `$hook_suffix` |
| `wp_insert_post_data` | 2 | `$data, $postarr` |
| `post_updated` | 3 | `$post_id, $post_after, $post_before` |

## Removing Hooks

### Basic Removal

**Critical:** Must match EXACT registration parameters (callback, priority).

```php
// Registration
add_action( 'init', 'mypl_init_function', 15 );

// ✅ GOOD: Exact match removes successfully
remove_action( 'init', 'mypl_init_function', 15 );

// ❌ BAD: Priority mismatch (won't remove)
remove_action( 'init', 'mypl_init_function' ); // Defaults to 10, registered as 15

// ❌ BAD: Wrong callback name (won't remove)
remove_action( 'init', 'mypl_init' ); // Function is mypl_init_function
```

### Removing Class Method Hooks

**Pattern 1: Removing from same instance**

```php
class MyPlugin {
	public function __construct() {
		add_action( 'init', array( $this, 'init' ) );
	}

	public function init() {
		// ...
	}

	public function remove_hooks() {
		// ✅ GOOD: Same instance, works
		remove_action( 'init', array( $this, 'init' ) );
	}
}

$plugin = new MyPlugin();
$plugin->remove_hooks(); // Successfully removes
```

**Pattern 2: Removing from different instance (won't work)**

```php
class MyPlugin {
	public function __construct() {
		add_action( 'init', array( $this, 'init' ) );
	}

	public function init() {
		// ...
	}
}

$plugin1 = new MyPlugin(); // Registers hook

// ❌ BAD: Different instance can't remove
$plugin2 = new MyPlugin();
remove_action( 'init', array( $plugin2, 'init' ) ); // Won't remove $plugin1's hook
```

**Pattern 3: Removing static method hooks**

```php
class MyPlugin {
	public static function init() {
		// ...
	}
}

// Registration
add_action( 'init', array( 'MyPlugin', 'init' ) );

// ✅ GOOD: Static method can be removed
remove_action( 'init', array( 'MyPlugin', 'init' ) );
```

**Pattern 4: Removing another plugin's hooks**

```php
// Another plugin registers:
add_action( 'wp_footer', 'some_plugin_footer', 20 );

// You can remove it (use cautiously):
add_action( 'wp_footer', 'mypl_remove_other_plugin_hook', 1 ); // Run early

function mypl_remove_other_plugin_hook() {
	// Must run before priority 20
	remove_action( 'wp_footer', 'some_plugin_footer', 20 );
}
```

### Removal Timing

```php
// ❌ BAD: Trying to remove before it's registered
remove_action( 'init', 'some_function' ); // Runs on plugins_loaded
add_action( 'init', 'some_function' );    // Registered later

// ✅ GOOD: Remove after registration
add_action( 'init', 'some_function' );
remove_action( 'init', 'some_function' ); // Same execution order

// ✅ GOOD: Remove during hook execution with higher priority
add_action( 'init', 'mypl_remove_hooks', 999 ); // Runs last

function mypl_remove_hooks() {
	remove_action( 'init', 'some_early_function' ); // Already ran
	remove_action( 'wp_footer', 'some_footer_function' ); // Hasn't run yet - prevented
}
```

## Custom Hooks for Extensibility

### Creating Custom Actions

**Purpose:** Allow other developers to hook into your plugin.

```php
// Your plugin code
function mypl_process_order( $order_id ) {
	// Validate order
	$order = mypl_get_order( $order_id );

	// Allow plugins to modify order before processing
	do_action( 'mypl_before_process_order', $order_id, $order );

	// Process order
	$result = mypl_internal_process( $order );

	// Allow plugins to act after processing
	do_action( 'mypl_after_process_order', $order_id, $order, $result );

	// Allow plugins to modify result
	$result = apply_filters( 'mypl_order_result', $result, $order_id );

	return $result;
}

// Other developers can hook in:
add_action( 'mypl_before_process_order', 'my_custom_validation', 10, 2 );

function my_custom_validation( $order_id, $order ) {
	// Custom validation logic
	if ( ! mypl_custom_check( $order ) ) {
		wp_die( 'Order validation failed' );
	}
}
```

### Creating Custom Filters

**Purpose:** Allow other developers to modify data in your plugin.

```php
// Your plugin code
function mypl_get_email_content( $order_id ) {
	$content = 'Thank you for your order #' . $order_id;

	// Allow plugins to modify email content
	$content = apply_filters( 'mypl_email_content', $content, $order_id );

	return $content;
}

// Other developers can modify:
add_filter( 'mypl_email_content', 'my_custom_email_content', 10, 2 );

function my_custom_email_content( $content, $order_id ) {
	$content .= "\n\nCustom footer text";
	return $content;
}

// Multiple parameters
function mypl_calculate_total( $items ) {
	$subtotal = array_sum( wp_list_pluck( $items, 'price' ) );
	$tax = $subtotal * 0.1;
	$total = $subtotal + $tax;

	// Allow plugins to modify total calculation
	$total = apply_filters( 'mypl_order_total', $total, $subtotal, $tax, $items );

	return $total;
}
```

### Hook Naming Conventions

**Pattern:** `{prefix}_{noun}_{verb}` or `{prefix}_{context}_{noun}`

```php
// ✅ GOOD: Clear naming
do_action( 'mypl_order_complete', $order_id );
do_action( 'mypl_user_registered', $user_id );
do_action( 'mypl_cache_cleared' );

apply_filters( 'mypl_product_price', $price, $product_id );
apply_filters( 'mypl_email_subject', $subject, $order_id );
apply_filters( 'mypl_query_args', $args );

// ❌ BAD: Unclear naming
do_action( 'mypl_process' ); // Process what?
do_action( 'hook_1' ); // Not descriptive

// ✅ GOOD: Before/after pairs
do_action( 'mypl_before_save_order', $order_id );
do_action( 'mypl_after_save_order', $order_id );

// ✅ GOOD: Specific context
do_action( 'mypl_admin_order_page_loaded' );
do_action( 'mypl_rest_api_order_created', $order_id );
```

### Documenting Custom Hooks

```php
/**
 * Process order.
 *
 * @param int $order_id Order ID.
 */
function mypl_process_order( $order_id ) {
	/**
	 * Fires before order processing begins.
	 *
	 * @since 1.0.0
	 *
	 * @param int   $order_id Order ID.
	 * @param array $order    Order data.
	 */
	do_action( 'mypl_before_process_order', $order_id, $order );

	// ...
}

/**
 * Get product price.
 *
 * @param int $product_id Product ID.
 * @return float Price.
 */
function mypl_get_product_price( $product_id ) {
	$price = get_post_meta( $product_id, '_price', true );

	/**
	 * Filters the product price.
	 *
	 * @since 1.0.0
	 *
	 * @param float $price      Product price.
	 * @param int   $product_id Product ID.
	 */
	return apply_filters( 'mypl_product_price', $price, $product_id );
}
```

## Common Hook Patterns

### Init Hook: Register CPTs and Taxonomies

```php
add_action( 'init', 'mypl_register_post_types' );

function mypl_register_post_types() {
	register_post_type( 'mypl_book', array(
		'labels' => array(
			'name'          => __( 'Books', 'my-plugin' ),
			'singular_name' => __( 'Book', 'my-plugin' ),
		),
		'public'       => true,
		'has_archive'  => true,
		'show_in_rest' => true,
	) );

	register_taxonomy( 'mypl_genre', 'mypl_book', array(
		'labels' => array(
			'name'          => __( 'Genres', 'my-plugin' ),
			'singular_name' => __( 'Genre', 'my-plugin' ),
		),
		'hierarchical' => true,
		'show_in_rest' => true,
	) );
}
```

### admin_init: Settings API

```php
add_action( 'admin_init', 'mypl_register_settings' );

function mypl_register_settings() {
	register_setting( 'mypl_options_group', 'mypl_settings', array(
		'type'              => 'array',
		'sanitize_callback' => 'mypl_sanitize_settings',
	) );

	add_settings_section(
		'mypl_main_section',
		__( 'Main Settings', 'my-plugin' ),
		'mypl_section_callback',
		'mypl-settings'
	);

	add_settings_field(
		'mypl_api_key',
		__( 'API Key', 'my-plugin' ),
		'mypl_api_key_callback',
		'mypl-settings',
		'mypl_main_section'
	);
}
```

### wp_enqueue_scripts: Frontend Assets

```php
add_action( 'wp_enqueue_scripts', 'mypl_enqueue_assets' );

function mypl_enqueue_assets() {
	wp_enqueue_style(
		'mypl-style',
		MYPL_PLUGIN_URL . 'public/css/style.css',
		array(),
		MYPL_VERSION
	);

	wp_enqueue_script(
		'mypl-script',
		MYPL_PLUGIN_URL . 'public/js/script.js',
		array( 'jquery' ),
		MYPL_VERSION,
		true
	);

	wp_localize_script( 'mypl-script', 'myplData', array(
		'ajaxurl' => admin_url( 'admin-ajax.php' ),
		'nonce'   => wp_create_nonce( 'mypl_nonce' ),
	) );
}
```

### admin_enqueue_scripts: Admin Assets

```php
add_action( 'admin_enqueue_scripts', 'mypl_admin_assets' );

function mypl_admin_assets( $hook_suffix ) {
	// Only load on plugin settings page
	if ( 'toplevel_page_my-plugin' !== $hook_suffix ) {
		return;
	}

	wp_enqueue_style( 'mypl-admin-style', MYPL_PLUGIN_URL . 'admin/css/admin.css', array(), MYPL_VERSION );
	wp_enqueue_script( 'mypl-admin-script', MYPL_PLUGIN_URL . 'admin/js/admin.js', array( 'jquery' ), MYPL_VERSION, true );
}
```

### rest_api_init: REST Routes

```php
add_action( 'rest_api_init', 'mypl_register_rest_routes' );

function mypl_register_rest_routes() {
	register_rest_route( 'mypl/v1', '/books', array(
		'methods'             => 'GET',
		'callback'            => 'mypl_get_books',
		'permission_callback' => '__return_true',
	) );
}
```

### admin_menu: Admin Pages

```php
add_action( 'admin_menu', 'mypl_add_admin_menu' );

function mypl_add_admin_menu() {
	add_menu_page(
		__( 'My Plugin', 'my-plugin' ),
		__( 'My Plugin', 'my-plugin' ),
		'manage_options',
		'my-plugin',
		'mypl_settings_page',
		'dashicons-book',
		20
	);

	add_submenu_page(
		'my-plugin',
		__( 'Settings', 'my-plugin' ),
		__( 'Settings', 'my-plugin' ),
		'manage_options',
		'my-plugin-settings',
		'mypl_settings_submenu_page'
	);
}
```

### plugin_action_links: Settings Link

```php
add_filter( 'plugin_action_links_' . plugin_basename( __FILE__ ), 'mypl_plugin_action_links' );

function mypl_plugin_action_links( $links ) {
	$settings_link = sprintf(
		'<a href="%s">%s</a>',
		admin_url( 'admin.php?page=my-plugin-settings' ),
		__( 'Settings', 'my-plugin' )
	);

	array_unshift( $links, $settings_link );

	return $links;
}
```

### wp_ajax_: AJAX Handlers

```php
// For logged-in users
add_action( 'wp_ajax_mypl_save_data', 'mypl_ajax_save_data' );

// For logged-out users
add_action( 'wp_ajax_nopriv_mypl_save_data', 'mypl_ajax_save_data' );

function mypl_ajax_save_data() {
	check_ajax_referer( 'mypl_nonce', 'nonce' );

	if ( ! current_user_can( 'edit_posts' ) ) {
		wp_send_json_error( array( 'message' => __( 'Permission denied', 'my-plugin' ) ) );
	}

	$data = sanitize_text_field( wp_unslash( $_POST['data'] ) );

	update_option( 'mypl_data', $data );

	wp_send_json_success( array( 'message' => __( 'Data saved', 'my-plugin' ) ) );
}
```

## Hook Debugging

### Checking Hook Registration

```php
// Check if action registered
if ( has_action( 'init', 'mypl_init_function' ) ) {
	// Hook is registered
}

// Get priority of registered hook
$priority = has_action( 'init', 'mypl_init_function' );
if ( false !== $priority ) {
	// Returns priority (int) if registered, false if not
	echo "Registered at priority: $priority";
}

// Check if any callback registered to hook
if ( has_action( 'init' ) ) {
	// At least one callback registered
}

// Check if filter registered
if ( has_filter( 'the_content', 'mypl_modify_content' ) ) {
	// Filter is registered
}
```

### Checking Hook Execution

```php
// Count how many times action fired
$count = did_action( 'init' );
if ( $count > 0 ) {
	// init has fired
}

// Get current hook name
add_action( 'init', 'mypl_debug_hook' );

function mypl_debug_hook() {
	$current = current_filter(); // Returns 'init'
	error_log( "Running on hook: $current" );
}

// Check if hook is currently running
add_action( 'save_post', 'mypl_on_save_post' );

function mypl_on_save_post( $post_id ) {
	if ( doing_action( 'save_post' ) ) {
		// Currently inside save_post action
	}
}
```

### Debugging with WP_DEBUG

```php
// Add to wp-config.php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );

// Log hook execution
add_action( 'init', 'mypl_debug_init' );

function mypl_debug_init() {
	error_log( 'mypl_init fired' );
	error_log( 'Current user: ' . get_current_user_id() );
}
```

## Hook Anti-Patterns

| Anti-Pattern | Why It's Bad | Fix |
|--------------|-------------|-----|
| **Hooking too early** | Register CPT on `plugins_loaded` | Use `init` hook (or later) |
| **Wrong hook priority** | `add_action( 'init', 'func', 999 )` when default works | Use default priority unless needed |
| **Forgetting to return in filters** | Filter breaks chain, fatal error | Always `return $value` |
| **Anonymous functions** | Can't be removed with `remove_action()` | Use named functions or class methods |
| **Action returning value** | Return value ignored by WordPress | Actions don't need return |
| **Excessive remove_action on core** | Breaks WordPress functionality | Only remove if necessary, understand impact |
| **Wrong accepted_args count** | Parameters undefined, PHP warnings | Match function signature to accepted_args |
| **Priority mismatch on removal** | `remove_action()` fails silently | Match exact registration priority |
| **Removing before registration** | Hook not yet registered | Remove after registration or with higher priority |

## Cross-References

- **Plugin architecture and lifecycle:** See `architecture-patterns.md`
- **Class-based hook registration:** See `oop-patterns.md`
- **Settings API hooks:** See `api-patterns.md`
- **Security patterns (nonces, capabilities):** See `wp-security-review` skill
