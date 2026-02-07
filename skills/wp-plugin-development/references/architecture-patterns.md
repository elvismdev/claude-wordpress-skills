# WordPress Plugin Architecture Patterns Reference

This reference catalogs plugin architecture patterns, file organization, lifecycle management, and WordPress-specific structural decisions for WordPress 6.x+ plugin development. Each pattern includes decision criteria, implementation examples, and BAD/GOOD code pairs following WordPress PHP Coding Standards.

## Quick Reference Table

| Pattern | When to Use | Complexity | Best For |
|---------|-------------|------------|----------|
| **Procedural (single file)** | Simple utility plugins, <100 LOC | Low | Form tweaks, simple filters, basic functionality |
| **Procedural (multi-file)** | Small plugins with organization needs | Low-Medium | Multiple features, separate admin/public code |
| **Class-based (singleton)** | Medium complexity, organized structure | Medium | Most plugins with multiple components |
| **Class-based (static factory)** | Need testability, dependency injection | Medium | Test-driven development, unit tests |
| **Namespace-based (PSR-4)** | Complex plugins, many classes | High | Large codebases, team development |
| **Service container** | Advanced DI, swappable components | High | Enterprise plugins, extensible architecture |

## Recommended File Structure

### Standard WordPress Plugin Structure

```
/my-plugin/
├── my-plugin.php              # Main plugin file with header (required)
├── uninstall.php              # Uninstall cleanup (preferred over hook)
├── readme.txt                 # WordPress.org readme (required for .org)
├── README.md                  # GitHub readme (optional)
├── LICENSE.txt                # License text (GPL v2+ for .org)
├── /languages/                # Translation files
│   ├── my-plugin.pot          # POT template (generated)
│   └── my-plugin-{locale}.po  # Translations
├── /includes/                 # Core plugin classes/functions
│   ├── class-main.php         # Main plugin class
│   ├── class-settings.php     # Settings management
│   └── functions.php          # Helper functions
├── /admin/                    # Admin-specific code
│   ├── class-admin.php        # Admin initialization
│   ├── /css/
│   │   └── admin-style.css
│   ├── /js/
│   │   └── admin-script.js
│   └── /views/
│       └── settings-page.php  # Admin templates
└── /public/                   # Public-facing code
    ├── class-public.php       # Public initialization
    ├── /css/
    │   └── public-style.css
    └── /js/
        └── public-script.js
```

**Why this structure:**
- **Separation of concerns:** Admin vs public code isolated
- **Conditional loading:** Load only what's needed (see Conditional Loading section)
- **Asset organization:** Clear CSS/JS separation
- **WordPress.org standard:** Matches Plugin Boilerplate, familiar to reviewers
- **Scalability:** Easy to add new components

### Alternative: Namespace-Based Structure (PSR-4)

```
/my-plugin/
├── my-plugin.php              # Bootstrap file
├── /src/                      # Namespaced classes (MyPlugin\)
│   ├── Plugin.php             # Main plugin class (MyPlugin\Plugin)
│   ├── Admin/
│   │   ├── Settings.php       # MyPlugin\Admin\Settings
│   │   └── Dashboard.php
│   ├── API/
│   │   └── Endpoint.php       # MyPlugin\API\Endpoint
│   └── Traits/
│       └── Singleton.php
├── /vendor/                   # Composer dependencies (committed for .org)
├── composer.json              # Autoloading config
└── uninstall.php
```

**When to use:**
- 10+ classes
- Team development
- External dependencies via Composer
- Need PHP namespaces for organization

## Main Plugin File Patterns

### Pattern 1: Procedural (Simple Plugins)

**Use for:** Small plugins with <5 functions, no admin settings, simple functionality.

```php
<?php
/**
 * Plugin Name: My Simple Plugin
 * Version: 1.0.0
 * Text Domain: my-simple-plugin
 */

// Direct file access prevention
if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

// Define constants
define( 'MYSP_VERSION', '1.0.0' );
define( 'MYSP_PLUGIN_DIR', plugin_dir_path( __FILE__ ) );

// Activation hook
register_activation_hook( __FILE__, 'mysp_activate' );

function mysp_activate() {
	add_option( 'mysp_version', MYSP_VERSION );
	flush_rewrite_rules();
}

// Core functionality
add_filter( 'the_content', 'mysp_add_footer' );

function mysp_add_footer( $content ) {
	if ( is_single() ) {
		$footer = '<p>' . esc_html__( 'Thanks for reading!', 'my-simple-plugin' ) . '</p>';
		$content .= $footer;
	}
	return $content;
}
```

**Pros:**
- Simple, easy to understand
- No class overhead
- Fast for small plugins

**Cons:**
- Global namespace pollution (must prefix everything)
- Hard to test
- Doesn't scale beyond 100 LOC

### Pattern 2: Class-Based with Singleton

**Use for:** Medium-complexity plugins with multiple components, settings, admin pages.

```php
<?php
/**
 * Plugin Name: My Plugin
 * Version: 1.0.0
 * Text Domain: my-plugin
 */

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

// ❌ BAD: Hooks in constructor (can't test, runs immediately)
class MyPlugin_Bad {
	public function __construct() {
		add_action( 'init', array( $this, 'register_post_types' ) );
		add_action( 'admin_menu', array( $this, 'add_menu' ) );
	}

	public static function get_instance() {
		static $instance = null;
		if ( null === $instance ) {
			$instance = new self();
		}
		return $instance;
	}

	public function register_post_types() {
		// ...
	}
}

// Instance created immediately - hooks fire before developer control
MyPlugin_Bad::get_instance();

// ✅ GOOD: Separate registration method, hooks after constructor
class MyPlugin {
	/**
	 * Plugin instance.
	 *
	 * @var MyPlugin
	 */
	private static $instance = null;

	/**
	 * Get singleton instance.
	 *
	 * @return MyPlugin
	 */
	public static function get_instance() {
		if ( null === self::$instance ) {
			self::$instance = new self();
		}
		return self::$instance;
	}

	/**
	 * Constructor (private to enforce singleton).
	 */
	private function __construct() {
		// Setup only, no hooks here
		$this->define_constants();
	}

	/**
	 * Define plugin constants.
	 */
	private function define_constants() {
		define( 'MYPL_VERSION', '1.0.0' );
		define( 'MYPL_PLUGIN_DIR', plugin_dir_path( __FILE__ ) );
		define( 'MYPL_PLUGIN_URL', plugin_dir_url( __FILE__ ) );
	}

	/**
	 * Register all hooks.
	 */
	public function register_hooks() {
		add_action( 'init', array( $this, 'register_post_types' ) );
		add_action( 'admin_menu', array( $this, 'add_admin_menu' ) );
		add_action( 'wp_enqueue_scripts', array( $this, 'enqueue_public_assets' ) );
	}

	/**
	 * Register custom post types.
	 */
	public function register_post_types() {
		register_post_type( 'mypl_book', array( /* ... */ ) );
	}

	/**
	 * Add admin menu.
	 */
	public function add_admin_menu() {
		add_menu_page( /* ... */ );
	}

	/**
	 * Enqueue public assets.
	 */
	public function enqueue_public_assets() {
		wp_enqueue_style( 'mypl-style', MYPL_PLUGIN_URL . 'public/css/style.css', array(), MYPL_VERSION );
	}
}

// Initialize plugin
$mypl = MyPlugin::get_instance();
$mypl->register_hooks();
```

**Why separate register_hooks():**
- Constructor runs once, sets up instance
- `register_hooks()` can be called multiple times for testing
- Clear separation: setup vs behavior
- Can remove hooks later if needed

### Pattern 3: Namespace-Based with PSR-4 Autoloading

**Use for:** Complex plugins with 10+ classes, need PHP namespaces.

**File: my-plugin.php**
```php
<?php
/**
 * Plugin Name: My Advanced Plugin
 * Version: 1.0.0
 * Text Domain: my-advanced-plugin
 */

namespace MyAdvancedPlugin;

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

// Composer autoloader (for WordPress.org, commit /vendor directory)
require_once __DIR__ . '/vendor/autoload.php';

// Initialize plugin
$plugin = new Plugin();
$plugin->init();
```

**File: src/Plugin.php**
```php
<?php

namespace MyAdvancedPlugin;

use MyAdvancedPlugin\Admin\Settings;
use MyAdvancedPlugin\API\Endpoint;

class Plugin {
	/**
	 * Initialize plugin components.
	 */
	public function init() {
		$this->define_constants();
		$this->register_hooks();
		$this->load_components();
	}

	/**
	 * Define plugin constants.
	 */
	private function define_constants() {
		if ( ! defined( 'MYADV_VERSION' ) ) {
			define( 'MYADV_VERSION', '1.0.0' );
		}
	}

	/**
	 * Register WordPress hooks.
	 */
	private function register_hooks() {
		add_action( 'init', array( $this, 'load_textdomain' ) );
		add_action( 'plugins_loaded', array( $this, 'on_plugins_loaded' ) );
	}

	/**
	 * Load plugin components.
	 */
	private function load_components() {
		if ( is_admin() ) {
			$settings = new Settings();
			$settings->init();
		}

		$endpoint = new Endpoint();
		$endpoint->register();
	}

	/**
	 * Load text domain.
	 */
	public function load_textdomain() {
		load_plugin_textdomain( 'my-advanced-plugin', false, dirname( plugin_basename( __FILE__ ) ) . '/languages' );
	}

	/**
	 * Hook: plugins_loaded.
	 */
	public function on_plugins_loaded() {
		do_action( 'myadv_loaded' );
	}
}
```

**File: composer.json**
```json
{
	"autoload": {
		"psr-4": {
			"MyAdvancedPlugin\\": "src/"
		}
	}
}
```

**Hooking namespaced functions:**
```php
// ❌ BAD: Function reference doesn't include namespace
add_action( 'init', 'load_textdomain' ); // Won't find function

// ✅ GOOD: Full namespace path
add_action( 'init', 'MyAdvancedPlugin\\load_textdomain' );

// ✅ GOOD: Use __NAMESPACE__ constant
add_action( 'init', __NAMESPACE__ . '\\load_textdomain' );

// ✅ GOOD: Array syntax for class methods (already works)
add_action( 'init', array( $this, 'method_name' ) );
```

## Plugin Constants

### Defining Plugin Constants

Constants provide plugin-wide access to version, paths, and URLs. Define early in main file.

```php
// ❌ BAD: Hardcoded paths break on custom WP installations
define( 'MYPL_PLUGIN_DIR', '/var/www/html/wp-content/plugins/my-plugin/' );
define( 'MYPL_PLUGIN_URL', 'https://example.com/wp-content/plugins/my-plugin/' );

// ❌ BAD: Relative paths don't work in all contexts
define( 'MYPL_PLUGIN_DIR', __DIR__ ); // Wrong from subdirectories

// ✅ GOOD: WordPress path functions
define( 'MYPL_VERSION', '1.0.0' );
define( 'MYPL_PLUGIN_FILE', __FILE__ );
define( 'MYPL_PLUGIN_DIR', plugin_dir_path( __FILE__ ) ); // Includes trailing slash
define( 'MYPL_PLUGIN_URL', plugin_dir_url( __FILE__ ) );  // Includes trailing slash
define( 'MYPL_PLUGIN_BASENAME', plugin_basename( __FILE__ ) ); // my-plugin/my-plugin.php

// Usage examples
require_once MYPL_PLUGIN_DIR . 'includes/functions.php';
wp_enqueue_script( 'mypl-script', MYPL_PLUGIN_URL . 'js/script.js', array(), MYPL_VERSION );
```

**Standard constants to define:**
- `{PREFIX}_VERSION` - Plugin version (for cache busting, upgrades)
- `{PREFIX}_PLUGIN_FILE` - Full path to main file (for activation hooks from other files)
- `{PREFIX}_PLUGIN_DIR` - Directory path (for requiring files)
- `{PREFIX}_PLUGIN_URL` - URL path (for enqueuing assets)
- `{PREFIX}_PLUGIN_BASENAME` - Basename (for plugin action links)

## Activation/Deactivation/Uninstall Deep Dive

### Activation Hook: Setup and Initialization

**Purpose:** One-time setup when plugin is activated. Set defaults, create DB tables, flush rewrites, add capabilities.

```php
register_activation_hook( __FILE__, 'mypl_activate' );

function mypl_activate() {
	// 1. Version check (only run on new install or upgrade)
	$current_version = get_option( 'mypl_version', '0.0.0' );
	$new_version = '1.2.0';

	if ( version_compare( $current_version, $new_version, '<' ) ) {
		mypl_upgrade_routine( $current_version, $new_version );
	}

	// 2. Set default options (only if not exist)
	add_option( 'mypl_settings', array(
		'api_key' => '',
		'enabled' => false,
	) );

	// 3. Create custom database table (use dbDelta)
	global $wpdb;
	$table_name = $wpdb->prefix . 'mypl_data';
	$charset_collate = $wpdb->get_charset_collate();

	// CRITICAL: dbDelta requires specific formatting
	$sql = "CREATE TABLE $table_name (
		id mediumint(9) NOT NULL AUTO_INCREMENT,
		user_id bigint(20) NOT NULL,
		data text NOT NULL,
		created datetime DEFAULT '0000-00-00 00:00:00' NOT NULL,
		PRIMARY KEY  (id),
		KEY user_id (user_id)
	) $charset_collate;";

	require_once ABSPATH . 'wp-admin/includes/upgrade.php';
	dbDelta( $sql );

	// 4. Register CPT before flushing rewrites
	mypl_register_post_types();

	// 5. Flush rewrite rules (expensive - only on activation)
	flush_rewrite_rules();

	// 6. Add custom capabilities
	$role = get_role( 'administrator' );
	if ( $role ) {
		$role->add_cap( 'manage_mypl_books' );
	}

	// 7. Update version
	update_option( 'mypl_version', $new_version );
}

function mypl_upgrade_routine( $from_version, $to_version ) {
	// Version-specific upgrade logic
	if ( version_compare( $from_version, '1.1.0', '<' ) ) {
		// Upgrade from < 1.1.0 to 1.1.0
		// Add new database column, migrate data, etc.
	}

	if ( version_compare( $from_version, '1.2.0', '<' ) ) {
		// Upgrade from < 1.2.0 to 1.2.0
	}
}
```

**Common activation mistakes:**
- Calling `flush_rewrite_rules()` without registering CPT first → 404 errors
- Using `$wpdb->query()` instead of `dbDelta()` → no upgrade path
- Not checking if options already exist → overwrites user settings on reactivation

### Deactivation Hook: Temporary Cleanup

**Purpose:** Temporary cleanup when plugin is deactivated (user may reactivate). Clear cache, unschedule cron, flush rewrites. DO NOT delete permanent data.

```php
register_deactivation_hook( __FILE__, 'mypl_deactivate' );

function mypl_deactivate() {
	// 1. Clear transients
	delete_transient( 'mypl_cache' );
	delete_transient( 'mypl_api_response' );

	// 2. Unschedule cron events
	$timestamp = wp_next_scheduled( 'mypl_daily_sync' );
	if ( $timestamp ) {
		wp_unschedule_event( $timestamp, 'mypl_daily_sync' );
	}

	// Clear all scheduled events for this plugin
	wp_clear_scheduled_hook( 'mypl_daily_sync' );

	// 3. Flush rewrite rules (removes custom CPT permalinks)
	flush_rewrite_rules();

	// 4. DO NOT delete options, tables, or permanent data
	// User may reactivate - data should persist
}
```

**What NOT to do in deactivation:**
- ❌ Delete options
- ❌ Drop database tables
- ❌ Remove user capabilities
- ❌ Delete post meta or user meta
- **These belong in uninstall only**

### Uninstall: Permanent Cleanup

**Purpose:** Complete removal when plugin is deleted (not just deactivated). Delete all traces: options, tables, metadata, capabilities.

**Method 1: uninstall.php (PREFERRED)**

**File: uninstall.php**
```php
<?php
/**
 * Uninstall script.
 *
 * Runs when plugin is deleted via WordPress admin.
 */

// Exit if not called by WordPress
if ( ! defined( 'WP_UNINSTALL_PLUGIN' ) ) {
	exit;
}

// Delete options
delete_option( 'mypl_version' );
delete_option( 'mypl_settings' );

// Delete transients
delete_transient( 'mypl_cache' );

// Drop custom database table
global $wpdb;
$table_name = $wpdb->prefix . 'mypl_data';
$wpdb->query( "DROP TABLE IF EXISTS $table_name" ); // OK here (not in create)

// Delete all post meta for this plugin
delete_post_meta_by_key( 'mypl_custom_field' );

// Delete all user meta for this plugin
delete_metadata( 'user', 0, 'mypl_user_setting', '', true ); // true = delete for all users

// Remove custom capabilities
$role = get_role( 'administrator' );
if ( $role ) {
	$role->remove_cap( 'manage_mypl_books' );
}

// Delete custom posts (if plugin created them)
$posts = get_posts( array(
	'post_type'   => 'mypl_book',
	'numberposts' => -1,
	'post_status' => 'any',
) );

foreach ( $posts as $post ) {
	wp_delete_post( $post->ID, true ); // true = force delete (bypass trash)
}

// Multisite: Delete site options if network activated
if ( is_multisite() ) {
	delete_site_option( 'mypl_network_settings' );
}
```

**Method 2: register_uninstall_hook() (alternative)**

Only use if `uninstall.php` doesn't exist. Less preferred because it requires plugin files to be present.

```php
// In main plugin file
register_uninstall_hook( __FILE__, 'mypl_uninstall' );

function mypl_uninstall() {
	// Same cleanup code as uninstall.php
	// Note: Function must be defined in main plugin file
}
```

**Uninstall vs Deactivation:**

| Action | Deactivation | Uninstall |
|--------|--------------|-----------|
| Delete options | ❌ No | ✅ Yes |
| Drop tables | ❌ No | ✅ Yes |
| Delete metadata | ❌ No | ✅ Yes |
| Remove capabilities | ❌ No | ✅ Yes |
| Clear cache | ✅ Yes | Optional |
| Unschedule cron | ✅ Yes | ✅ Yes |

### Version Upgrade Pattern

**Tracking version for upgrade routines:**

```php
// On activation or init
function mypl_check_version() {
	$installed_version = get_option( 'mypl_version', '0.0.0' );
	$current_version = MYPL_VERSION; // From constant

	if ( version_compare( $installed_version, $current_version, '<' ) ) {
		mypl_upgrade( $installed_version, $current_version );
		update_option( 'mypl_version', $current_version );
	}
}

function mypl_upgrade( $from, $to ) {
	// 1.0.0 → 1.1.0: Add new database column
	if ( version_compare( $from, '1.1.0', '<' ) ) {
		global $wpdb;
		$table_name = $wpdb->prefix . 'mypl_data';
		$wpdb->query( "ALTER TABLE $table_name ADD COLUMN status varchar(20) DEFAULT 'active'" );
	}

	// 1.1.0 → 1.2.0: Migrate old option format
	if ( version_compare( $from, '1.2.0', '<' ) ) {
		$old_option = get_option( 'mypl_old_format' );
		if ( $old_option ) {
			$new_format = mypl_migrate_option_format( $old_option );
			update_option( 'mypl_settings', $new_format );
			delete_option( 'mypl_old_format' );
		}
	}
}
```

## Conditional Loading

### Admin vs Public Code Separation

**Pattern:** Load admin code only in admin, public code only on frontend.

```php
// ❌ BAD: Loading everything everywhere
require_once MYPL_PLUGIN_DIR . 'admin/class-admin.php';
require_once MYPL_PLUGIN_DIR . 'public/class-public.php';

new MyPlugin_Admin(); // Loads on every request (frontend too)
new MyPlugin_Public();

// ✅ GOOD: Conditional loading
if ( is_admin() ) {
	require_once MYPL_PLUGIN_DIR . 'admin/class-admin.php';
	$admin = new MyPlugin_Admin();
	$admin->init();
} else {
	require_once MYPL_PLUGIN_DIR . 'public/class-public.php';
	$public = new MyPlugin_Public();
	$public->init();
}

// AJAX caution: is_admin() returns true for AJAX requests
if ( is_admin() && ! wp_doing_ajax() ) {
	// Admin-only code (not AJAX)
}

if ( is_admin() && wp_doing_ajax() ) {
	// AJAX handlers
}
```

**Other conditional contexts:**

```php
// REST API requests
if ( defined( 'REST_REQUEST' ) && REST_REQUEST ) {
	// Load REST-specific code
}

// Cron requests
if ( wp_doing_cron() ) {
	// Load cron-specific code
}

// Frontend only (not admin, not AJAX, not REST, not cron)
if ( ! is_admin() && ! wp_doing_ajax() && ! wp_doing_cron() ) {
	// Pure frontend code
}
```

## Prefixing Deep Dive

### What to Prefix

**Rule:** Everything in the global namespace must be prefixed or namespaced.

```php
// ❌ BAD: No prefix
function save_data() { }
class Settings { }
define( 'VERSION', '1.0' );
$cache = array();
update_option( 'plugin_settings', $data );

// ✅ GOOD: Everything prefixed
function mypl_save_data() { }
class MyPlugin_Settings { }
define( 'MYPL_VERSION', '1.0' );
$mypl_cache = array(); // Avoid globals, but prefix if needed
update_option( 'mypl_settings', $data );
```

**Complete prefix checklist:**

| Item | Example | Why |
|------|---------|-----|
| Functions | `mypl_save_settings()` | Global namespace collision |
| Classes | `MyPlugin_Admin` | Class name collision |
| Global variables | `$mypl_cache` | Variable collision |
| Options | `mypl_settings` | Database option collision |
| Transients | `mypl_api_cache` | Transient name collision |
| Post types | `mypl_book` | Query var collision |
| Taxonomies | `mypl_genre` | Query var collision |
| Shortcodes | `[mypl_form]` | Shortcode collision |
| Cron hooks | `mypl_daily_sync` | Cron hook collision |
| REST namespace | `mypl/v1` | REST route collision |
| Admin page slugs | `mypl-settings` | Menu slug collision |
| Metadata keys | `mypl_custom_field` | Meta key collision |

**Choosing a prefix:**
- 4-5 characters minimum (2-3 too common)
- Memorable and related to plugin name
- Lowercase for functions/options
- CamelCase for classes: `MyPrefix_ClassName`
- Check WordPress.org for conflicts before deciding

**Examples:**
- WooCommerce: `wc_` / `WC_`
- Yoast SEO: `wpseo_` / `WPSEO_`
- Advanced Custom Fields: `acf_` / `ACF_`

### PHP Namespaces as Alternative

**Namespaces eliminate need for function/class prefixes:**

```php
namespace MyPlugin;

// No prefix needed on functions/classes within namespace
function save_data() { }
class Settings { }

// Still prefix options, post types, etc. (database/WordPress scope)
add_option( 'mypl_settings', $data );
register_post_type( 'mypl_book', $args );
```

**Namespace guidelines:**
- Use PascalCase: `MyPlugin\Admin\Settings`
- Match directory structure for PSR-4
- Still prefix: options, post types, taxonomies, metadata (WordPress doesn't namespace these)

## Plugin Dependencies

### Declaring Dependencies with Requires Plugins Header

**WordPress 6.5+ feature:** Declare plugin dependencies in header.

```php
/**
 * Plugin Name: My WooCommerce Extension
 * Requires Plugins: woocommerce
 * Version: 1.0.0
 */
```

**Multiple dependencies:**
```php
/**
 * Requires Plugins: woocommerce, jetpack, advanced-custom-fields
 */
```

**Effect:**
- WordPress prevents activation if dependencies not installed
- Shows admin notice with missing plugins
- Handles activation order automatically

### Manual Dependency Checks (Pre-6.5 or for functions)

```php
// Check if WooCommerce is active
if ( ! class_exists( 'WooCommerce' ) ) {
	add_action( 'admin_notices', 'mypl_woocommerce_missing_notice' );
	return; // Don't load plugin
}

function mypl_woocommerce_missing_notice() {
	?>
	<div class="notice notice-error">
		<p><?php esc_html_e( 'My Plugin requires WooCommerce to be installed and activated.', 'my-plugin' ); ?></p>
	</div>
	<?php
}

// Check for specific function
if ( ! function_exists( 'acf_add_local_field_group' ) ) {
	// ACF not available
}

// Check for minimum version
if ( defined( 'WC_VERSION' ) && version_compare( WC_VERSION, '5.0', '<' ) ) {
	add_action( 'admin_notices', 'mypl_woocommerce_version_notice' );
	return;
}
```

## Common Architecture Anti-Patterns

| Anti-Pattern | Why It's Bad | Fix |
|--------------|-------------|-----|
| **God class** | One class does everything, impossible to maintain | Separate concerns: Admin, Public, Settings, API classes |
| **No separation of concerns** | Admin code runs on frontend, performance waste | Use `is_admin()` for conditional loading |
| **Global state abuse** | `global $mypl_data` everywhere | Use class properties or options API |
| **Loading everything on init** | All code loads on every page, slow | Conditional loading by context |
| **Hardcoded paths** | `/wp-content/plugins/my-plugin/` breaks custom installations | Use `plugin_dir_path()`, `plugin_dir_url()` |
| **Hooks in constructor** | Can't test, immediate execution | Use separate `register_hooks()` method |
| **Missing uninstall cleanup** | Leaves database pollution after deletion | Create `uninstall.php` |
| **No version tracking** | Can't run upgrade routines | Store version in option, check on activation |
| **Direct DB queries instead of WP functions** | Misses caching, Multisite compatibility | Use `get_posts()`, not `$wpdb->get_results()` when possible |
| **Tight coupling** | Classes directly instantiate dependencies | Dependency injection or service container |

## Cross-References

- **Hooks system details:** See `hooks-guide.md`
- **OOP architecture patterns:** See `oop-patterns.md`
- **WordPress API integration:** See `api-patterns.md`
- **Security patterns (nonces, capabilities, sanitization):** See `wp-security-review` skill
