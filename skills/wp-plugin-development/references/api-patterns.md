# WordPress Plugin API Patterns Reference

This reference covers WordPress core APIs for plugin development: Settings API, Options API, REST API, Transients API, custom database tables, AJAX handlers, Cron API, and Admin Menu API. All examples follow WordPress PHP Coding Standards. For security patterns (nonces, capability checks, sanitization), see wp-security-review skill.

## Quick Reference Table

| API | Key Function | Hook | Purpose |
|-----|--------------|------|---------|
| **Settings API** | `register_setting()` | `admin_init` | Structured settings pages with automatic validation |
| **Options API** | `update_option()` | Any | Store plugin configuration in database |
| **REST API** | `register_rest_route()` | `rest_api_init` | Create custom REST endpoints |
| **Transients API** | `set_transient()` | Any | Temporary cached data with expiration |
| **Custom DB Tables** | `dbDelta()` | Activation hook | Create custom database tables |
| **AJAX API** | `wp_ajax_{action}` | Hook | Handle AJAX requests |
| **Cron API** | `wp_schedule_event()` | Activation hook | Schedule recurring tasks |
| **Admin Menu API** | `add_menu_page()` | `admin_menu` | Create admin menu pages |

## Settings API Complete Workflow

### Registration Process

**All Settings API functions hook to `admin_init`:**

```php
add_action( 'admin_init', 'mypl_register_settings' );

function mypl_register_settings() {
	// Step 1: Register setting (option name in database)
	register_setting(
		'mypl_options_group',  // Option group (used in settings_fields())
		'mypl_settings',        // Option name (database key)
		array(
			'type'              => 'array',
			'sanitize_callback' => 'mypl_sanitize_settings',
			'default'           => array(
				'api_key' => '',
				'enabled' => false,
			),
			'show_in_rest'      => false, // Don't expose in REST API
		)
	);

	// Step 2: Add settings section
	add_settings_section(
		'mypl_main_section',         // Section ID
		__( 'Main Settings', 'my-plugin' ), // Section title
		'mypl_section_callback',     // Callback for section description
		'mypl-settings'              // Page slug (must match do_settings_sections())
	);

	// Step 3: Add settings fields to section
	add_settings_field(
		'mypl_api_key',              // Field ID
		__( 'API Key', 'my-plugin' ), // Field label
		'mypl_api_key_callback',     // Callback to render field HTML
		'mypl-settings',             // Page slug (must match section)
		'mypl_main_section'          // Section ID (must match add_settings_section())
	);

	add_settings_field(
		'mypl_enabled',
		__( 'Enable Feature', 'my-plugin' ),
		'mypl_enabled_callback',
		'mypl-settings',
		'mypl_main_section'
	);
}
```

### Field Render Callbacks

```php
/**
 * Section description callback.
 */
function mypl_section_callback() {
	echo '<p>' . esc_html__( 'Configure your plugin settings below.', 'my-plugin' ) . '</p>';
}

/**
 * Text field callback.
 */
function mypl_api_key_callback() {
	$options = get_option( 'mypl_settings', array() );
	$value = isset( $options['api_key'] ) ? $options['api_key'] : '';
	?>
	<input type="text"
	       name="mypl_settings[api_key]"
	       value="<?php echo esc_attr( $value ); ?>"
	       class="regular-text">
	<p class="description">
		<?php esc_html_e( 'Enter your API key from the provider dashboard.', 'my-plugin' ); ?>
	</p>
	<?php
}

/**
 * Checkbox field callback.
 */
function mypl_enabled_callback() {
	$options = get_option( 'mypl_settings', array() );
	$checked = isset( $options['enabled'] ) && $options['enabled'];
	?>
	<label>
		<input type="checkbox"
		       name="mypl_settings[enabled]"
		       value="1"
		       <?php checked( $checked, true ); ?>>
		<?php esc_html_e( 'Enable this feature', 'my-plugin' ); ?>
	</label>
	<?php
}

/**
 * Select field callback.
 */
function mypl_mode_callback() {
	$options = get_option( 'mypl_settings', array() );
	$mode = isset( $options['mode'] ) ? $options['mode'] : 'basic';
	?>
	<select name="mypl_settings[mode]">
		<option value="basic" <?php selected( $mode, 'basic' ); ?>>
			<?php esc_html_e( 'Basic', 'my-plugin' ); ?>
		</option>
		<option value="advanced" <?php selected( $mode, 'advanced' ); ?>>
			<?php esc_html_e( 'Advanced', 'my-plugin' ); ?>
		</option>
	</select>
	<?php
}

/**
 * Number field callback.
 */
function mypl_timeout_callback() {
	$options = get_option( 'mypl_settings', array() );
	$timeout = isset( $options['timeout'] ) ? $options['timeout'] : 30;
	?>
	<input type="number"
	       name="mypl_settings[timeout]"
	       value="<?php echo esc_attr( $timeout ); ?>"
	       min="5"
	       max="300"
	       class="small-text">
	<span><?php esc_html_e( 'seconds', 'my-plugin' ); ?></span>
	<?php
}
```

### Sanitization Callback

```php
/**
 * Sanitize settings.
 *
 * @param array $input Raw input from form.
 * @return array Sanitized settings.
 */
function mypl_sanitize_settings( $input ) {
	$sanitized = array();

	// Text field
	if ( isset( $input['api_key'] ) ) {
		$sanitized['api_key'] = sanitize_text_field( $input['api_key'] );
	}

	// Checkbox (boolean)
	$sanitized['enabled'] = isset( $input['enabled'] ) && $input['enabled'] ? true : false;

	// Select (whitelist)
	if ( isset( $input['mode'] ) && in_array( $input['mode'], array( 'basic', 'advanced' ), true ) ) {
		$sanitized['mode'] = $input['mode'];
	} else {
		$sanitized['mode'] = 'basic';
	}

	// Number (range)
	if ( isset( $input['timeout'] ) ) {
		$timeout = absint( $input['timeout'] );
		$sanitized['timeout'] = max( 5, min( 300, $timeout ) ); // Clamp to 5-300
	}

	// Array field
	if ( isset( $input['allowed_ips'] ) && is_array( $input['allowed_ips'] ) ) {
		$sanitized['allowed_ips'] = array_map( 'sanitize_text_field', $input['allowed_ips'] );
	}

	return $sanitized;
}
```

### Settings Page HTML

```php
/**
 * Render settings page.
 */
function mypl_settings_page() {
	// Check user capability
	if ( ! current_user_can( 'manage_options' ) ) {
		return;
	}

	// Check if settings updated
	if ( isset( $_GET['settings-updated'] ) ) {
		add_settings_error(
			'mypl_messages',
			'mypl_message',
			__( 'Settings saved successfully.', 'my-plugin' ),
			'updated'
		);
	}

	// Show error/success messages
	settings_errors( 'mypl_messages' );
	?>
	<div class="wrap">
		<h1><?php echo esc_html( get_admin_page_title() ); ?></h1>

		<form method="post" action="options.php">
			<?php
			// Output nonce, action, option_page fields
			settings_fields( 'mypl_options_group' );

			// Output sections and fields
			do_settings_sections( 'mypl-settings' );

			// Output save button
			submit_button( __( 'Save Settings', 'my-plugin' ) );
			?>
		</form>
	</div>
	<?php
}
```

### Common Settings API Mistakes

```php
// ❌ BAD: Field won't save (missing register_setting)
add_action( 'admin_init', 'mypl_bad_settings' );

function mypl_bad_settings() {
	add_settings_field( 'mypl_field', 'Field', 'mypl_callback', 'mypl-page', 'section' );
	// Missing register_setting() - form won't save!
}

// ❌ BAD: Section doesn't appear (wrong page slug)
add_settings_section( 'section', 'Title', 'callback', 'wrong-page' );
do_settings_sections( 'mypl-settings' ); // Slug mismatch

// ❌ BAD: Missing sanitize_callback (security risk)
register_setting( 'group', 'mypl_settings' ); // No sanitization!

// ✅ GOOD: Complete registration
register_setting( 'group', 'mypl_settings', array(
	'sanitize_callback' => 'mypl_sanitize',
) );
```

## Options API Best Practices

### Basic Option Operations

```php
// Add option (only if doesn't exist)
add_option( 'mypl_settings', array(
	'api_key' => '',
	'enabled' => false,
), '', 'no' ); // autoload = no for large options

// Get option with default
$settings = get_option( 'mypl_settings', array(
	'api_key' => '',
	'enabled' => false,
) );

// Update option (creates if doesn't exist)
update_option( 'mypl_settings', array(
	'api_key' => 'new-key',
	'enabled' => true,
) );

// Delete option
delete_option( 'mypl_settings' );

// Check if option exists
if ( false !== get_option( 'mypl_settings' ) ) {
	// Option exists
}
```

### Autoloading Options

**Autoload:** Loads option on every page load. Use `'no'` for large/infrequent options.

```php
// ❌ BAD: Large transient data autoloaded
add_option( 'mypl_large_cache', $large_data ); // Autoload defaults to 'yes'

// ✅ GOOD: Disable autoload for large data
add_option( 'mypl_large_cache', $large_data, '', 'no' );

// ✅ GOOD: Autoload small, frequently-used options
add_option( 'mypl_api_key', 'key123', '', 'yes' );

// ❌ BAD: Using options for temporary data
update_option( 'mypl_cache_' . $key, $data ); // Use transients instead

// ✅ GOOD: Use transients for temporary data
set_transient( 'mypl_cache_' . $key, $data, HOUR_IN_SECONDS );
```

### Single Option vs Multiple Options

**Pattern 1: Single array option (RECOMMENDED)**

```php
// ✅ GOOD: One option, multiple settings
$settings = array(
	'api_key' => 'key',
	'enabled' => true,
	'timeout' => 30,
	'mode'    => 'advanced',
);
update_option( 'mypl_settings', $settings );

// Retrieve
$settings = get_option( 'mypl_settings' );
$api_key = $settings['api_key'];
```

**Pattern 2: Multiple separate options**

```php
// ❌ LESS EFFICIENT: Multiple DB queries
update_option( 'mypl_api_key', 'key' );
update_option( 'mypl_enabled', true );
update_option( 'mypl_timeout', 30 );
update_option( 'mypl_mode', 'advanced' );

// Retrieval requires multiple queries
$api_key = get_option( 'mypl_api_key' );
$enabled = get_option( 'mypl_enabled' );
// ...
```

**When to use multiple options:**
- Settings in different contexts (some autoload, some don't)
- Very large individual values
- Options updated independently by different features

### Prefixing Option Names

```php
// ❌ BAD: Generic names (collision risk)
update_option( 'settings', $data );
update_option( 'version', '1.0' );

// ✅ GOOD: Prefixed names
update_option( 'mypl_settings', $data );
update_option( 'mypl_version', '1.0' );
```

## REST API Endpoint Registration

**Note:** For security patterns (permission_callback, capability checks, nonces), see wp-security-review skill. This section covers non-security aspects.

### Basic Endpoint Registration

```php
add_action( 'rest_api_init', 'mypl_register_rest_routes' );

function mypl_register_rest_routes() {
	register_rest_route(
		'mypl/v1',              // Namespace (plugin/version)
		'/books',               // Route
		array(
			'methods'             => 'GET',
			'callback'            => 'mypl_get_books',
			'permission_callback' => '__return_true', // Public endpoint
			'args'                => array(
				'per_page' => array(
					'default'           => 10,
					'sanitize_callback' => 'absint',
					'validate_callback' => 'mypl_validate_per_page',
				),
			),
		)
	);
}
```

### Route Parameters with Regex

```php
// Route with ID parameter
register_rest_route(
	'mypl/v1',
	'/books/(?P<id>\d+)', // Regex: \d+ = one or more digits
	array(
		'methods'             => 'GET',
		'callback'            => 'mypl_get_book',
		'permission_callback' => '__return_true',
		'args'                => array(
			'id' => array(
				'validate_callback' => function( $param ) {
					return is_numeric( $param );
				},
			),
		),
	)
);

function mypl_get_book( $request ) {
	$book_id = $request->get_param( 'id' );
	$book = get_post( $book_id );

	if ( ! $book || 'mypl_book' !== $book->post_type ) {
		return new WP_Error(
			'not_found',
			__( 'Book not found', 'my-plugin' ),
			array( 'status' => 404 )
		);
	}

	return rest_ensure_response( $book );
}
```

### HTTP Methods

```php
// Multiple methods on same route
register_rest_route(
	'mypl/v1',
	'/books/(?P<id>\d+)',
	array(
		// GET method
		array(
			'methods'             => 'GET',
			'callback'            => 'mypl_get_book',
			'permission_callback' => '__return_true',
		),
		// PUT method
		array(
			'methods'             => 'PUT',
			'callback'            => 'mypl_update_book',
			'permission_callback' => 'mypl_update_book_permission',
			'args'                => array(
				'title' => array(
					'required'          => true,
					'sanitize_callback' => 'sanitize_text_field',
					'validate_callback' => function( $param ) {
						return ! empty( $param ) && strlen( $param ) <= 200;
					},
				),
			),
		),
		// DELETE method
		array(
			'methods'             => 'DELETE',
			'callback'            => 'mypl_delete_book',
			'permission_callback' => 'mypl_delete_book_permission',
		),
	)
);

// Use WP_REST_Server constants
register_rest_route(
	'mypl/v1',
	'/books',
	array(
		'methods'             => WP_REST_Server::READABLE, // GET
		'callback'            => 'mypl_get_books',
		'permission_callback' => '__return_true',
	)
);

// Other constants:
// WP_REST_Server::CREATABLE  - POST
// WP_REST_Server::EDITABLE   - POST, PUT, PATCH
// WP_REST_Server::DELETABLE  - DELETE
// WP_REST_Server::ALLMETHODS - All methods
```

### Args: Sanitize, Validate, Default

```php
register_rest_route(
	'mypl/v1',
	'/search',
	array(
		'methods'             => 'GET',
		'callback'            => 'mypl_search',
		'permission_callback' => '__return_true',
		'args'                => array(
			// String parameter
			'query' => array(
				'required'          => true,
				'sanitize_callback' => 'sanitize_text_field',
				'validate_callback' => function( $param ) {
					return ! empty( $param ) && strlen( $param ) <= 100;
				},
			),
			// Integer parameter
			'page' => array(
				'default'           => 1,
				'sanitize_callback' => 'absint',
				'validate_callback' => function( $param ) {
					return $param > 0 && $param <= 1000;
				},
			),
			// Boolean parameter
			'include_private' => array(
				'default'           => false,
				'sanitize_callback' => 'rest_sanitize_boolean',
			),
			// Array parameter
			'categories' => array(
				'default'           => array(),
				'sanitize_callback' => function( $param ) {
					return array_map( 'absint', (array) $param );
				},
				'validate_callback' => function( $param ) {
					return is_array( $param );
				},
			),
		),
	)
);
```

### Response Handling

```php
function mypl_get_books( $request ) {
	$per_page = $request->get_param( 'per_page' );

	$books = get_posts( array(
		'post_type'      => 'mypl_book',
		'posts_per_page' => $per_page,
	) );

	// ✅ GOOD: Return WP_REST_Response
	return new WP_REST_Response( $books, 200 );

	// ✅ GOOD: Use rest_ensure_response()
	return rest_ensure_response( $books );

	// ✅ GOOD: Return WP_Error for errors
	return new WP_Error(
		'error_code',
		__( 'Error message', 'my-plugin' ),
		array( 'status' => 400 )
	);

	// ✅ GOOD: Return array (converted to WP_REST_Response)
	return array(
		'books' => $books,
		'total' => count( $books ),
	);
}
```

### Schema Definition

```php
register_rest_route(
	'mypl/v1',
	'/books',
	array(
		'methods'             => 'GET',
		'callback'            => 'mypl_get_books',
		'permission_callback' => '__return_true',
		'schema'              => 'mypl_book_schema', // Schema callback
	)
);

function mypl_book_schema() {
	return array(
		'$schema'    => 'http://json-schema.org/draft-04/schema#',
		'title'      => 'book',
		'type'       => 'object',
		'properties' => array(
			'id'     => array(
				'description' => __( 'Unique identifier for the book.', 'my-plugin' ),
				'type'        => 'integer',
				'readonly'    => true,
			),
			'title'  => array(
				'description' => __( 'Book title.', 'my-plugin' ),
				'type'        => 'string',
			),
			'author' => array(
				'description' => __( 'Book author.', 'my-plugin' ),
				'type'        => 'string',
			),
		),
	);
}
```

## Transients API

### Basic Transient Operations

```php
// Set transient with expiration
set_transient( 'mypl_api_cache', $api_data, HOUR_IN_SECONDS );

// Get transient
$cached = get_transient( 'mypl_api_cache' );
if ( false === $cached ) {
	// Cache miss - fetch fresh data
	$cached = mypl_fetch_api_data();
	set_transient( 'mypl_api_cache', $cached, HOUR_IN_SECONDS );
}

// Delete transient
delete_transient( 'mypl_api_cache' );

// Time constants
MINUTE_IN_SECONDS  // 60
HOUR_IN_SECONDS    // 3600
DAY_IN_SECONDS     // 86400
WEEK_IN_SECONDS    // 604800
MONTH_IN_SECONDS   // 2592000 (30 days)
YEAR_IN_SECONDS    // 31536000
```

### Transient Naming

```php
// ❌ BAD: Generic name (collision risk)
set_transient( 'cache', $data, HOUR_IN_SECONDS );

// ❌ BAD: Too long (limit 172 chars including _transient_ prefix)
set_transient( 'mypl_this_is_a_very_long_transient_name_that_exceeds_the_database_field_length_limit_and_will_be_truncated_causing_issues', $data, HOUR_IN_SECONDS );

// ✅ GOOD: Prefixed, descriptive, under 172 chars
set_transient( 'mypl_api_response', $data, HOUR_IN_SECONDS );
set_transient( 'mypl_user_' . $user_id . '_posts', $posts, DAY_IN_SECONDS );
```

### Choosing Expiration Time

```php
// ✅ GOOD: API response (1 hour)
set_transient( 'mypl_api_data', $data, HOUR_IN_SECONDS );

// ✅ GOOD: Daily report (24 hours)
set_transient( 'mypl_daily_report', $report, DAY_IN_SECONDS );

// ✅ GOOD: Expensive query (10 minutes)
set_transient( 'mypl_complex_query', $results, 10 * MINUTE_IN_SECONDS );

// ❌ BAD: No expiration (use options instead)
set_transient( 'mypl_permanent_data', $data, 0 );
```

### When to Invalidate Transients

```php
// Clear transient when data changes
add_action( 'save_post_mypl_book', 'mypl_clear_book_cache' );

function mypl_clear_book_cache( $post_id ) {
	delete_transient( 'mypl_all_books' );
	delete_transient( 'mypl_book_' . $post_id );
}

// Clear transient on settings update
add_action( 'update_option_mypl_settings', 'mypl_clear_cache_on_settings_change' );

function mypl_clear_cache_on_settings_change() {
	delete_transient( 'mypl_api_cache' );
}
```

## Custom Database Tables

### When to Use Custom Tables

**Decision guide:**

| Use Case | Solution | Why |
|----------|----------|-----|
| Plugin settings | Options API | Simple, cached, Multisite-compatible |
| Post metadata | Post Meta API | Attached to posts, WP_Query integration |
| User preferences | User Meta API | Attached to users, per-user storage |
| Large datasets | Custom table | Performance, complex queries |
| Logs/analytics | Custom table | High write volume, archiving |
| Relationships | Custom table | Many-to-many relationships |

### Creating Tables with dbDelta()

**CRITICAL:** `dbDelta()` has strict SQL formatting requirements.

```php
register_activation_hook( __FILE__, 'mypl_create_tables' );

function mypl_create_tables() {
	global $wpdb;

	$table_name = $wpdb->prefix . 'mypl_logs';
	$charset_collate = $wpdb->get_charset_collate();

	// CRITICAL: dbDelta() SQL format requirements:
	// - Two spaces after PRIMARY KEY
	// - Each field on its own line
	// - Key definitions after field definitions
	// - No spaces around = in default values
	$sql = "CREATE TABLE $table_name (
		id bigint(20) unsigned NOT NULL AUTO_INCREMENT,
		user_id bigint(20) unsigned NOT NULL,
		action varchar(50) NOT NULL,
		data text,
		ip_address varchar(45) NOT NULL,
		created datetime DEFAULT '0000-00-00 00:00:00' NOT NULL,
		PRIMARY KEY  (id),
		KEY user_id (user_id),
		KEY created (created)
	) $charset_collate;";

	require_once ABSPATH . 'wp-admin/includes/upgrade.php';
	dbDelta( $sql );

	// Store database version for upgrades
	add_option( 'mypl_db_version', '1.0' );
}
```

### dbDelta() Common Mistakes

```php
// ❌ BAD: Using $wpdb->query() (no upgrade path)
$wpdb->query( "CREATE TABLE $table_name (...)" );

// ❌ BAD: One space after PRIMARY KEY
$sql = "CREATE TABLE $table_name (
	id int NOT NULL AUTO_INCREMENT,
	PRIMARY KEY (id)
)"; // Won't create primary key

// ✅ GOOD: Two spaces after PRIMARY KEY
$sql = "CREATE TABLE $table_name (
	id int NOT NULL AUTO_INCREMENT,
	PRIMARY KEY  (id)
)"; // Works

// ❌ BAD: Missing charset/collate
$sql = "CREATE TABLE $table_name (...)";

// ✅ GOOD: Include charset/collate
$sql = "CREATE TABLE $table_name (...) $charset_collate;";

// ❌ BAD: Key definition before field definitions
$sql = "CREATE TABLE $table_name (
	PRIMARY KEY  (id),
	id int NOT NULL AUTO_INCREMENT
)";

// ✅ GOOD: Field definitions first, then keys
$sql = "CREATE TABLE $table_name (
	id int NOT NULL AUTO_INCREMENT,
	PRIMARY KEY  (id)
)";
```

### Database Version Upgrades

```php
function mypl_check_db_version() {
	$installed_version = get_option( 'mypl_db_version', '0.0' );
	$current_version = '1.1';

	if ( version_compare( $installed_version, $current_version, '<' ) ) {
		mypl_upgrade_db( $installed_version, $current_version );
		update_option( 'mypl_db_version', $current_version );
	}
}

function mypl_upgrade_db( $from, $to ) {
	global $wpdb;
	$table_name = $wpdb->prefix . 'mypl_logs';

	if ( version_compare( $from, '1.1', '<' ) ) {
		// Add new column in version 1.1
		$wpdb->query( "ALTER TABLE $table_name ADD COLUMN user_agent varchar(255) DEFAULT ''" );
	}
}
```

## AJAX Handler Patterns

**Note:** For security patterns (nonces, capability checks), see wp-security-review skill.

### Basic AJAX Handler

```php
// For logged-in users
add_action( 'wp_ajax_mypl_action', 'mypl_ajax_handler' );

// For logged-out users
add_action( 'wp_ajax_nopriv_mypl_action', 'mypl_ajax_handler' );

function mypl_ajax_handler() {
	// Nonce verification (see wp-security-review for details)
	check_ajax_referer( 'mypl_nonce', 'nonce' );

	// Capability check (see wp-security-review for details)
	if ( ! current_user_can( 'edit_posts' ) ) {
		wp_send_json_error( array(
			'message' => __( 'Permission denied', 'my-plugin' ),
		), 403 );
	}

	// Get and sanitize input
	$data = sanitize_text_field( wp_unslash( $_POST['data'] ) );

	// Process
	$result = mypl_process_data( $data );

	// Send JSON response
	if ( $result ) {
		wp_send_json_success( array(
			'message' => __( 'Success', 'my-plugin' ),
			'result'  => $result,
		) );
	} else {
		wp_send_json_error( array(
			'message' => __( 'Failed', 'my-plugin' ),
		) );
	}
}
```

### Passing Data to JavaScript

```php
add_action( 'wp_enqueue_scripts', 'mypl_enqueue_ajax_script' );

function mypl_enqueue_ajax_script() {
	wp_enqueue_script( 'mypl-ajax', MYPL_URL . 'js/ajax.js', array( 'jquery' ), MYPL_VERSION, true );

	wp_localize_script( 'mypl-ajax', 'myplAjax', array(
		'ajaxurl' => admin_url( 'admin-ajax.php' ),
		'nonce'   => wp_create_nonce( 'mypl_nonce' ),
	) );
}
```

### REST API vs AJAX

**Prefer REST API for new code:**

| Aspect | AJAX (admin-ajax.php) | REST API |
|--------|----------------------|----------|
| **Bootstrap** | Full WordPress load | Lighter load |
| **Caching** | Harder | Easier |
| **Discoverability** | No | Yes (self-documenting) |
| **Standard** | WordPress-specific | Industry standard |
| **Use case** | Legacy support | New development |

## Cron API

### Scheduling Events

```php
register_activation_hook( __FILE__, 'mypl_schedule_cron' );

function mypl_schedule_cron() {
	// Check if already scheduled (prevent duplicates)
	if ( ! wp_next_scheduled( 'mypl_daily_sync' ) ) {
		wp_schedule_event( time(), 'daily', 'mypl_daily_sync' );
	}
}

// Hook to cron event
add_action( 'mypl_daily_sync', 'mypl_run_daily_sync' );

function mypl_run_daily_sync() {
	// Sync logic
	$data = mypl_fetch_remote_data();
	update_option( 'mypl_synced_data', $data );
}
```

### Unscheduling Events

```php
register_deactivation_hook( __FILE__, 'mypl_unschedule_cron' );

function mypl_unschedule_cron() {
	$timestamp = wp_next_scheduled( 'mypl_daily_sync' );
	if ( $timestamp ) {
		wp_unschedule_event( $timestamp, 'mypl_daily_sync' );
	}

	// Clear all scheduled events for this hook
	wp_clear_scheduled_hook( 'mypl_daily_sync' );
}
```

### Custom Cron Schedules

```php
add_filter( 'cron_schedules', 'mypl_add_cron_schedules' );

function mypl_add_cron_schedules( $schedules ) {
	$schedules['every_five_minutes'] = array(
		'interval' => 5 * MINUTE_IN_SECONDS,
		'display'  => __( 'Every 5 Minutes', 'my-plugin' ),
	);

	$schedules['twice_daily'] = array(
		'interval' => 12 * HOUR_IN_SECONDS,
		'display'  => __( 'Twice Daily', 'my-plugin' ),
	);

	return $schedules;
}

// Use custom schedule
wp_schedule_event( time(), 'every_five_minutes', 'mypl_frequent_sync' );
```

## Admin Menu API

### Adding Menu Pages

```php
add_action( 'admin_menu', 'mypl_add_admin_menu' );

function mypl_add_admin_menu() {
	// Top-level menu
	add_menu_page(
		__( 'My Plugin', 'my-plugin' ),  // Page title
		__( 'My Plugin', 'my-plugin' ),  // Menu title
		'manage_options',                 // Capability
		'my-plugin',                      // Menu slug
		'mypl_main_page',                 // Callback function
		'dashicons-book',                 // Icon
		20                                // Position
	);

	// Submenu page
	add_submenu_page(
		'my-plugin',                      // Parent slug
		__( 'Settings', 'my-plugin' ),    // Page title
		__( 'Settings', 'my-plugin' ),    // Menu title
		'manage_options',                 // Capability
		'my-plugin-settings',             // Menu slug
		'mypl_settings_page'              // Callback function
	);

	// Add to existing menu (Settings)
	add_options_page(
		__( 'My Plugin Settings', 'my-plugin' ),
		__( 'My Plugin', 'my-plugin' ),
		'manage_options',
		'my-plugin-settings',
		'mypl_settings_page'
	);
}
```

### Plugin Action Links

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

## Cross-References

- **Plugin architecture and lifecycle:** See `architecture-patterns.md`
- **Hooks system for API integration:** See `hooks-guide.md`
- **OOP patterns for API classes:** See `oop-patterns.md`
- **Security patterns (nonces, capabilities, sanitization, permission_callback):** See `wp-security-review` skill
