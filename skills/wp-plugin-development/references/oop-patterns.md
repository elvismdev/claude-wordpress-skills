# WordPress Plugin OOP Patterns Reference

This reference covers object-oriented programming patterns for WordPress plugin development. Includes class-based architecture, namespaces, PSR-4 autoloading, traits, abstract classes, and service containers for WordPress 6.x+. All examples follow WordPress PHP Coding Standards.

## Quick Reference Table

| Pattern | Use Case | Complexity | PHP Requirement |
|---------|----------|------------|-----------------|
| **Procedural functions** | Simple plugins, <5 functions | Low | PHP 5.6+ |
| **Single class (singleton)** | Medium plugins, organized structure | Low-Medium | PHP 5.6+ |
| **Multiple classes (no namespace)** | Medium plugins, multiple components | Medium | PHP 5.6+ |
| **PHP namespaces** | Complex plugins, avoid name collisions | Medium | PHP 5.3+ |
| **PSR-4 autoloading** | Large plugins, many classes | Medium-High | PHP 5.3+ |
| **Traits for code reuse** | Share behavior across classes | Medium | PHP 5.4+ |
| **Abstract classes** | Define interfaces for similar components | Medium-High | PHP 5.6+ |
| **Service container** | Advanced DI, swappable dependencies | High | PHP 5.6+ |

## Class-Based Plugin Pattern

### Standard Singleton Pattern

**Use for:** Medium-complexity plugins with organized structure. Most common WordPress plugin pattern.

```php
<?php
/**
 * Plugin Name: My Plugin
 * Version: 1.0.0
 */

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

// ❌ BAD: Hooks in constructor (can't test, immediate execution)
class MyPlugin_Bad {
	public function __construct() {
		add_action( 'init', array( $this, 'init' ) );
		add_action( 'admin_menu', array( $this, 'add_menu' ) );
		// Runs immediately when instance created - no control
	}
}

$plugin = new MyPlugin_Bad(); // Hooks fire now

// ✅ GOOD: Separate registration method
class MyPlugin {
	/**
	 * Single instance of the plugin.
	 *
	 * @var MyPlugin
	 */
	private static $instance = null;

	/**
	 * Get plugin instance.
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
	 * Constructor (private for singleton).
	 */
	private function __construct() {
		$this->define_constants();
		$this->load_dependencies();
	}

	/**
	 * Define plugin constants.
	 */
	private function define_constants() {
		if ( ! defined( 'MYPL_VERSION' ) ) {
			define( 'MYPL_VERSION', '1.0.0' );
		}
		if ( ! defined( 'MYPL_PLUGIN_DIR' ) ) {
			define( 'MYPL_PLUGIN_DIR', plugin_dir_path( __FILE__ ) );
		}
	}

	/**
	 * Load plugin dependencies.
	 */
	private function load_dependencies() {
		require_once MYPL_PLUGIN_DIR . 'includes/functions.php';
	}

	/**
	 * Register all hooks.
	 */
	public function register_hooks() {
		add_action( 'init', array( $this, 'init' ) );
		add_action( 'admin_menu', array( $this, 'add_admin_menu' ) );
		add_action( 'wp_enqueue_scripts', array( $this, 'enqueue_assets' ) );
	}

	/**
	 * Initialize plugin.
	 */
	public function init() {
		// Register post types, taxonomies, etc.
	}

	/**
	 * Add admin menu.
	 */
	public function add_admin_menu() {
		// Add menu pages
	}

	/**
	 * Enqueue frontend assets.
	 */
	public function enqueue_assets() {
		// Enqueue scripts and styles
	}
}

// Initialize plugin
$mypl = MyPlugin::get_instance();
$mypl->register_hooks();
```

**Why this pattern works:**
- **Singleton ensures one instance** - Prevents multiple plugin instances
- **Private constructor** - Enforces singleton pattern
- **Separate register_hooks()** - Testable, can be called multiple times
- **Clear separation** - Setup (constructor) vs behavior (hooks)

### Static Factory Pattern (Alternative)

**Use for:** Better testability, dependency injection, unit tests.

```php
class MyPlugin {
	/**
	 * Create plugin instance.
	 *
	 * @return MyPlugin
	 */
	public static function create() {
		return new self();
	}

	/**
	 * Constructor (public for testing).
	 */
	public function __construct() {
		$this->define_constants();
	}

	/**
	 * Register hooks.
	 */
	public function register_hooks() {
		add_action( 'init', array( $this, 'init' ) );
	}

	/**
	 * Initialize plugin.
	 */
	public function init() {
		// ...
	}
}

// Production usage
$plugin = MyPlugin::create();
$plugin->register_hooks();

// Testing usage (can create multiple instances)
$plugin1 = new MyPlugin();
$plugin2 = new MyPlugin();
```

**Singleton vs Static Factory:**

| Aspect | Singleton | Static Factory |
|--------|-----------|---------------|
| **Instances** | One only | Multiple allowed |
| **Testing** | Harder | Easier |
| **Memory** | Lower | Higher |
| **Use case** | Production plugins | Test-driven development |

## PHP Namespaces for WordPress

### Namespace Declaration

**Purpose:** Organize code, avoid name collisions without prefixes.

```php
<?php
/**
 * File: my-plugin/src/Admin/Settings.php
 */

namespace MyPlugin\Admin;

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

/**
 * Settings class.
 */
class Settings {
	/**
	 * Initialize settings.
	 */
	public function init() {
		add_action( 'admin_init', array( $this, 'register_settings' ) );
	}

	/**
	 * Register settings.
	 */
	public function register_settings() {
		// Still prefix option names (not namespaced in WordPress)
		register_setting( 'mypl_options', 'mypl_settings' );
	}
}
```

### Using Namespaced Classes

```php
<?php
/**
 * File: my-plugin/my-plugin.php
 */

namespace MyPlugin;

// Import classes with 'use'
use MyPlugin\Admin\Settings;
use MyPlugin\API\Endpoint;

class Plugin {
	/**
	 * Initialize plugin.
	 */
	public function init() {
		if ( is_admin() ) {
			$settings = new Settings(); // Imported above
			$settings->init();
		}

		$endpoint = new Endpoint();
		$endpoint->register();
	}
}

// Instantiate
$plugin = new Plugin();
$plugin->init();
```

### Hooking Namespaced Functions

```php
namespace MyPlugin;

// ❌ BAD: Function reference doesn't include namespace
add_action( 'init', 'load_textdomain' ); // PHP: Call to undefined function

// ✅ GOOD: Full namespace path
add_action( 'init', 'MyPlugin\\load_textdomain' );

// ✅ GOOD: Use __NAMESPACE__ constant
add_action( 'init', __NAMESPACE__ . '\\load_textdomain' );

// ✅ GOOD: Class methods automatically work
class Plugin {
	public function init() {
		add_action( 'init', array( $this, 'on_init' ) ); // Works
	}
}

// ✅ GOOD: Static methods with namespace
class Utils {
	public static function helper() { }
}
add_action( 'init', array( 'MyPlugin\\Utils', 'helper' ) );
```

### What Still Needs Prefixing

**Namespaces only affect PHP scope, not WordPress:**

```php
namespace MyPlugin;

// ✅ No prefix needed (PHP namespace)
class Settings { }
function save_data() { }
const VERSION = '1.0.0';

// ❌ Still need prefix (WordPress scope)
register_post_type( 'mypl_book', $args );      // Post type names
register_taxonomy( 'mypl_genre', $args );       // Taxonomy names
add_option( 'mypl_settings', $data );          // Option names
set_transient( 'mypl_cache', $data );          // Transient names
update_post_meta( $id, 'mypl_field', $value ); // Meta keys
do_action( 'mypl_custom_hook' );               // Custom hooks
```

## PSR-4 Autoloading

### Composer Autoloading Setup

**Use for:** Plugins with 10+ classes, avoid manual `require_once`.

**File: composer.json**
```json
{
	"name": "vendor/my-plugin",
	"description": "WordPress plugin with PSR-4 autoloading",
	"type": "wordpress-plugin",
	"require": {
		"php": ">=7.4"
	},
	"autoload": {
		"psr-4": {
			"MyPlugin\\": "src/"
		}
	}
}
```

**Directory structure:**
```
/my-plugin/
├── my-plugin.php
├── composer.json
├── /vendor/             # Composer dependencies (commit for WordPress.org)
│   └── autoload.php
└── /src/                # Maps to MyPlugin\ namespace
    ├── Plugin.php       # MyPlugin\Plugin
    ├── /Admin/
    │   ├── Settings.php # MyPlugin\Admin\Settings
    │   └── Menu.php     # MyPlugin\Admin\Menu
    └── /API/
        └── Endpoint.php # MyPlugin\API\Endpoint
```

**File: my-plugin.php**
```php
<?php
/**
 * Plugin Name: My Plugin
 * Version: 1.0.0
 */

namespace MyPlugin;

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

// Load Composer autoloader
require_once __DIR__ . '/vendor/autoload.php';

// Classes automatically loaded when referenced
$plugin = new Plugin(); // Loads src/Plugin.php
$plugin->init();
```

**File: src/Plugin.php**
```php
<?php

namespace MyPlugin;

use MyPlugin\Admin\Settings;
use MyPlugin\API\Endpoint;

class Plugin {
	/**
	 * Initialize plugin.
	 */
	public function init() {
		$settings = new Settings(); // Autoloaded from src/Admin/Settings.php
		$settings->init();

		$endpoint = new Endpoint(); // Autoloaded from src/API/Endpoint.php
		$endpoint->register();
	}
}
```

### Manual PSR-4 Autoloader (No Composer)

**Use for:** Avoid Composer dependency, simple autoloading.

```php
<?php
/**
 * File: my-plugin.php
 */

namespace MyPlugin;

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

/**
 * PSR-4 autoloader.
 *
 * @param string $class Class name.
 */
spl_autoload_register( function ( $class ) {
	$prefix = 'MyPlugin\\';
	$base_dir = __DIR__ . '/src/';

	// Check if class uses this namespace
	$len = strlen( $prefix );
	if ( strncmp( $prefix, $class, $len ) !== 0 ) {
		return;
	}

	// Get relative class name
	$relative_class = substr( $class, $len );

	// Replace namespace separators with directory separators
	$file = $base_dir . str_replace( '\\', '/', $relative_class ) . '.php';

	// Require file if exists
	if ( file_exists( $file ) ) {
		require_once $file;
	}
} );

// Classes automatically loaded
$plugin = new Plugin();
$plugin->init();
```

### WordPress.org Considerations

**CRITICAL:** WordPress.org plugins cannot require `composer install`.

```php
// ❌ BAD: Requires user to run composer install
require_once __DIR__ . '/vendor/autoload.php'; // Fails if vendor/ not present

// ✅ GOOD: Commit /vendor directory to WordPress.org SVN
// Include /vendor in your plugin zip
// Plugin works without composer install
```

**Steps for WordPress.org:**
1. Run `composer install --no-dev` locally
2. Commit `/vendor` directory to plugin
3. Upload to WordPress.org SVN with vendor included

## Trait Usage

### Reusable Behavior Across Classes

**Purpose:** Share methods across multiple classes without inheritance.

```php
<?php

namespace MyPlugin\Traits;

/**
 * Singleton trait.
 */
trait Singleton {
	/**
	 * Instance.
	 *
	 * @var object
	 */
	private static $instance = null;

	/**
	 * Get instance.
	 *
	 * @return object
	 */
	public static function get_instance() {
		if ( null === self::$instance ) {
			self::$instance = new self();
		}
		return self::$instance;
	}

	/**
	 * Private constructor.
	 */
	private function __construct() {
		$this->init();
	}

	/**
	 * Initialize (override in class).
	 */
	abstract protected function init();
}
```

**Using traits:**

```php
namespace MyPlugin\Admin;

use MyPlugin\Traits\Singleton;

/**
 * Settings class.
 */
class Settings {
	use Singleton;

	/**
	 * Initialize settings.
	 */
	protected function init() {
		add_action( 'admin_init', array( $this, 'register_settings' ) );
	}

	/**
	 * Register settings.
	 */
	public function register_settings() {
		// ...
	}
}

// Usage
$settings = Settings::get_instance();
```

### Practical WordPress Traits

**HasHooks trait:**

```php
namespace MyPlugin\Traits;

trait HasHooks {
	/**
	 * Registered hooks.
	 *
	 * @var array
	 */
	private $hooks = array();

	/**
	 * Register hook.
	 *
	 * @param string $hook     Hook name.
	 * @param string $method   Method name.
	 * @param int    $priority Priority.
	 * @param int    $args     Accepted args.
	 */
	protected function add_action( $hook, $method, $priority = 10, $args = 1 ) {
		add_action( $hook, array( $this, $method ), $priority, $args );
		$this->hooks[] = array( 'action', $hook, $method, $priority );
	}

	/**
	 * Register filter.
	 *
	 * @param string $hook     Hook name.
	 * @param string $method   Method name.
	 * @param int    $priority Priority.
	 * @param int    $args     Accepted args.
	 */
	protected function add_filter( $hook, $method, $priority = 10, $args = 1 ) {
		add_filter( $hook, array( $this, $method ), $priority, $args );
		$this->hooks[] = array( 'filter', $hook, $method, $priority );
	}

	/**
	 * Remove all registered hooks.
	 */
	public function remove_hooks() {
		foreach ( $this->hooks as $hook_data ) {
			list( $type, $hook, $method, $priority ) = $hook_data;

			if ( 'action' === $type ) {
				remove_action( $hook, array( $this, $method ), $priority );
			} else {
				remove_filter( $hook, array( $this, $method ), $priority );
			}
		}
	}
}
```

**HasSettings trait:**

```php
namespace MyPlugin\Traits;

trait HasSettings {
	/**
	 * Get setting value.
	 *
	 * @param string $key     Setting key.
	 * @param mixed  $default Default value.
	 * @return mixed
	 */
	protected function get_setting( $key, $default = null ) {
		$settings = get_option( $this->get_option_name(), array() );
		return isset( $settings[ $key ] ) ? $settings[ $key ] : $default;
	}

	/**
	 * Update setting value.
	 *
	 * @param string $key   Setting key.
	 * @param mixed  $value Setting value.
	 */
	protected function update_setting( $key, $value ) {
		$settings = get_option( $this->get_option_name(), array() );
		$settings[ $key ] = $value;
		update_option( $this->get_option_name(), $settings );
	}

	/**
	 * Get option name (must be implemented by class).
	 *
	 * @return string
	 */
	abstract protected function get_option_name();
}
```

### Trait vs Inheritance

| Approach | Use When | Pros | Cons |
|----------|----------|------|------|
| **Trait** | Share behavior across unrelated classes | Flexible, multiple traits per class | Can cause method conflicts |
| **Inheritance** | Is-a relationship, strict hierarchy | Clear parent-child relationship | Single inheritance only |

```php
// ❌ BAD: Deep inheritance for code reuse
class Base { }
class Level1 extends Base { }
class Level2 extends Level1 { }
class Level3 extends Level2 { } // Too deep!

// ✅ GOOD: Traits for code reuse
class Admin {
	use Singleton;
	use HasHooks;
	use HasSettings;
}
```

## Abstract Classes for Extensibility

### Base Class Pattern

**Purpose:** Define interface for similar components, enforce structure.

```php
<?php

namespace MyPlugin\Abstracts;

/**
 * Abstract admin page.
 */
abstract class Abstract_Admin_Page {
	/**
	 * Page slug.
	 *
	 * @var string
	 */
	protected $slug;

	/**
	 * Page title.
	 *
	 * @var string
	 */
	protected $title;

	/**
	 * Constructor.
	 */
	public function __construct() {
		$this->slug = $this->get_slug();
		$this->title = $this->get_title();
		$this->register_hooks();
	}

	/**
	 * Register hooks.
	 */
	public function register_hooks() {
		add_action( 'admin_menu', array( $this, 'add_page' ) );
		add_action( 'admin_enqueue_scripts', array( $this, 'enqueue_assets' ) );
	}

	/**
	 * Add admin page.
	 */
	public function add_page() {
		add_menu_page(
			$this->title,
			$this->title,
			$this->get_capability(),
			$this->slug,
			array( $this, 'render' ),
			$this->get_icon()
		);
	}

	/**
	 * Enqueue assets.
	 *
	 * @param string $hook_suffix Current admin page.
	 */
	public function enqueue_assets( $hook_suffix ) {
		if ( 'toplevel_page_' . $this->slug !== $hook_suffix ) {
			return;
		}

		$this->enqueue_scripts();
		$this->enqueue_styles();
	}

	/**
	 * Get page slug (must implement).
	 *
	 * @return string
	 */
	abstract protected function get_slug();

	/**
	 * Get page title (must implement).
	 *
	 * @return string
	 */
	abstract protected function get_title();

	/**
	 * Get capability (can override).
	 *
	 * @return string
	 */
	protected function get_capability() {
		return 'manage_options';
	}

	/**
	 * Get icon (can override).
	 *
	 * @return string
	 */
	protected function get_icon() {
		return 'dashicons-admin-generic';
	}

	/**
	 * Render page (must implement).
	 */
	abstract public function render();

	/**
	 * Enqueue scripts (can override).
	 */
	protected function enqueue_scripts() { }

	/**
	 * Enqueue styles (can override).
	 */
	protected function enqueue_styles() { }
}
```

**Using abstract class:**

```php
namespace MyPlugin\Admin;

use MyPlugin\Abstracts\Abstract_Admin_Page;

/**
 * Settings page.
 */
class Settings_Page extends Abstract_Admin_Page {
	/**
	 * Get slug.
	 *
	 * @return string
	 */
	protected function get_slug() {
		return 'mypl-settings';
	}

	/**
	 * Get title.
	 *
	 * @return string
	 */
	protected function get_title() {
		return __( 'My Plugin Settings', 'my-plugin' );
	}

	/**
	 * Render page.
	 */
	public function render() {
		?>
		<div class="wrap">
			<h1><?php echo esc_html( $this->title ); ?></h1>
			<!-- Settings form -->
		</div>
		<?php
	}

	/**
	 * Enqueue scripts.
	 */
	protected function enqueue_scripts() {
		wp_enqueue_script( 'mypl-settings', MYPL_URL . 'admin/js/settings.js', array(), MYPL_VERSION, true );
	}
}

// Usage
$settings_page = new Settings_Page();
```

### When to Use Abstract Classes

| Use Case | Appropriate? | Why |
|----------|-------------|-----|
| Multiple similar admin pages | ✅ Yes | Common structure, different content |
| Multiple custom post types | ✅ Yes | Shared registration logic |
| Multiple REST endpoints | ✅ Yes | Common validation patterns |
| Single unique class | ❌ No | Over-engineering |
| Unrelated classes | ❌ No | Use traits instead |

## Service Container Pattern

### Simple WordPress Service Container

**Purpose:** Manage dependencies, dependency injection, testability.

```php
<?php

namespace MyPlugin;

/**
 * Service container.
 */
class Container {
	/**
	 * Services.
	 *
	 * @var array
	 */
	private $services = array();

	/**
	 * Register service.
	 *
	 * @param string   $name    Service name.
	 * @param callable $factory Factory function.
	 */
	public function register( $name, $factory ) {
		$this->services[ $name ] = $factory;
	}

	/**
	 * Get service (singleton).
	 *
	 * @param string $name Service name.
	 * @return mixed
	 */
	public function get( $name ) {
		if ( ! isset( $this->services[ $name ] ) ) {
			throw new \Exception( "Service {$name} not found" );
		}

		// Create instance if not already created
		if ( is_callable( $this->services[ $name ] ) ) {
			$this->services[ $name ] = call_user_func( $this->services[ $name ], $this );
		}

		return $this->services[ $name ];
	}
}
```

**Registering services:**

```php
namespace MyPlugin;

use MyPlugin\Admin\Settings;
use MyPlugin\API\Client;

$container = new Container();

// Register services
$container->register( 'settings', function( $container ) {
	return new Settings();
} );

$container->register( 'api_client', function( $container ) {
	$api_key = $container->get( 'settings' )->get_api_key();
	return new Client( $api_key );
} );

// Get services
$settings = $container->get( 'settings' );
$client = $container->get( 'api_client' );
```

**Dependency injection with container:**

```php
namespace MyPlugin\Admin;

use MyPlugin\API\Client;

/**
 * Dashboard class.
 */
class Dashboard {
	/**
	 * API client.
	 *
	 * @var Client
	 */
	private $client;

	/**
	 * Constructor.
	 *
	 * @param Client $client API client.
	 */
	public function __construct( Client $client ) {
		$this->client = $client;
	}

	/**
	 * Fetch data.
	 */
	public function fetch_data() {
		return $this->client->get( '/endpoint' );
	}
}

// Without container (tight coupling)
$client = new Client( 'api-key' );
$dashboard = new Dashboard( $client );

// With container (loose coupling)
$container->register( 'dashboard', function( $container ) {
	return new Dashboard( $container->get( 'api_client' ) );
} );
$dashboard = $container->get( 'dashboard' );
```

## WordPress-Specific OOP Considerations

### Hooking Class Methods

```php
class MyPlugin {
	public function __construct() {
		// ✅ GOOD: Array syntax for instance methods
		add_action( 'init', array( $this, 'init' ) );
	}

	public function init() { }
}

class MyPlugin_Static {
	public static function init() { }
}

// ✅ GOOD: Array syntax for static methods
add_action( 'init', array( 'MyPlugin_Static', 'init' ) );
```

### Removing Class Method Hooks

```php
class MyPlugin {
	public function __construct() {
		add_action( 'init', array( $this, 'init' ), 10 );
	}

	public function init() { }

	public function remove_hooks() {
		// ✅ GOOD: Same instance can remove
		remove_action( 'init', array( $this, 'init' ), 10 );
	}
}

$plugin = new MyPlugin();
$plugin->remove_hooks(); // Works

// ❌ BAD: Different instance can't remove
$plugin2 = new MyPlugin();
remove_action( 'init', array( $plugin2, 'init' ) ); // Won't remove $plugin's hook
```

### $this vs self:: vs static::

```php
class MyPlugin {
	private $property = 'value';
	private static $static_property = 'static value';

	public function method() {
		// $this - Current instance
		echo $this->property;

		// self:: - Current class (early binding)
		echo self::$static_property;

		// static:: - Called class (late binding, PHP 5.3+)
		echo static::$static_property;
	}
}
```

## Anti-Patterns

| Anti-Pattern | Why It's Bad | Fix |
|--------------|-------------|-----|
| **Over-engineering simple plugins** | Complex OOP for 50 LOC plugin | Use procedural or single class |
| **Framework-itis** | Building a framework instead of a plugin | Focus on plugin functionality |
| **God class** | One class does everything (1000+ LOC) | Separate concerns into multiple classes |
| **Tight coupling** | `new Dependency()` inside methods | Dependency injection via constructor |
| **Static abuse** | Everything static, no testability | Use instance methods with DI |
| **Deep inheritance** | 4+ levels of inheritance | Use traits for code reuse |
| **Missing type hints** | No type safety (PHP 7.0+ available) | Add type hints to method signatures |
| **Public properties** | Direct property access breaks encapsulation | Use getter/setter methods |

## Cross-References

- **Plugin architecture and file structure:** See `architecture-patterns.md`
- **Hooks system for class methods:** See `hooks-guide.md`
- **WordPress API integration:** See `api-patterns.md`
- **Security patterns (dependency injection security):** See `wp-security-review` skill
