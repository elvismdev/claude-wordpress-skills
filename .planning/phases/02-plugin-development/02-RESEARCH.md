# Phase 2: Plugin Development - Research

**Researched:** 2026-02-06
**Domain:** WordPress plugin architecture and WordPress.org submission standards (code-level)
**Confidence:** HIGH

## Summary

WordPress plugin development centers on architecture patterns, WordPress.org submission compliance, and API integration for WordPress 6.x+. The standard approach follows WordPress's plugin file structure conventions, prefixing requirements, lifecycle hooks (activation/deactivation/uninstall), and Plugin Check (PCP) tool standards for WordPress.org submissions.

Research confirmed that all major plugin APIs (Settings API, hooks system, custom post types/taxonomies, internationalization) remain stable in WordPress 6.x with extensive official documentation. The skill should mirror the security/performance skill's structure but with plugin-specific content organized by architectural concerns and compliance requirements. Each finding maps to WordPress.org standards and Plugin Check tool requirements for credibility and includes concrete fix examples following WordPress PHP Coding Standards.

**Primary recommendation:** Structure the skill around WordPress's plugin development lifecycle: initialization (headers, prefixing), activation/deactivation/uninstall hooks, core APIs (Settings, hooks, CPT/taxonomies, i18n), and WordPress.org compliance (Plugin Check standards, readme.txt, licensing). Detection patterns must distinguish between WordPress.org submissions (strictest standards), private plugins (more flexibility), and must-use plugins (different loading behavior).

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Detection scope & depth:**
- **Deep coverage (dedicated sections in SKILL.md):** Plugin headers and metadata, activation/deactivation/uninstall hooks, custom post types and taxonomies (register_post_type, register_taxonomy), Settings API (register_setting, add_settings_section, add_settings_field), hooks system (add_action/add_filter patterns, priority, removal), internationalization (text domains, __(), _e(), _n(), _x()), prefixing and namespacing
- **Surface-level coverage (scan patterns + brief guidance):** REST API endpoint registration (deep security patterns live in wp-security-review), AJAX handler registration (security patterns in wp-security-review), shortcode registration (deprecated pattern for blocks but still common), widget registration (classic widgets — blocks preferred in WP 6.x+), admin notices and dismissible messages
- **WordPress.org compliance checks:** Plugin Check (PCP) tool standards — proper headers, no direct file access, text domain matching slug, no deprecated functions, readme.txt/readme.md requirements, license compatibility (GPLv2+)
- **Context-aware detection:** Distinguish between:
  - WordPress.org submission (strictest standards — PCP compliance required)
  - Private/enterprise plugins (more flexibility on naming, no readme.txt needed)
  - Must-use plugins (mu-plugins — different loading behavior, no activation hooks)
  - Drop-in plugins (special files like object-cache.php, db.php)

**Security skill crossover:**
- **Do NOT duplicate security patterns** — the plugin skill references wp-security-review for security concerns
- **Cross-reference approach:** When the plugin review encounters a security-relevant pattern (e.g., form handler without nonce, AJAX without capability check), flag it as "See wp-security-review skill for detailed security analysis" rather than repeating the full pattern
- **Plugin-specific security notes:** Include brief reminders of the three-step pattern (nonce + capability + sanitize) in the context of plugin handlers, but point to security skill for depth
- **Slash command integration:** If /wp-plugin-review finds security concerns, suggest running /wp-sec-review for comprehensive security analysis

**Finding output style:**
- Mirror the security/performance skill output format exactly:
  - Group findings by FILE (consistent with other skills)
  - Each finding: line number, severity, issue description, explanation, ❌ BAD / ✅ GOOD code pair with fix
  - Summary at end with total counts by severity
- **Severity mapping for plugins:**
  - CRITICAL: Will cause WordPress.org rejection OR breaks plugin functionality (missing headers, wrong text domain, missing uninstall cleanup, direct DB table creation without dbDelta)
  - WARNING: Non-standard patterns that cause maintainability issues or compatibility problems (global namespace pollution, missing prefixing, hardcoded paths, tight coupling)
  - INFO: Best practice improvements (could use register_activation_hook instead of manual check, missing flush_rewrite_rules on activation, etc.)
- **No additional fields beyond what security/performance use** — keep format consistent across all skills

**Reference doc structure:**
- Follow the performance/security skill pattern: reference docs are **deep-dive companions** to SKILL.md
- Each reference doc is a **cookbook** — quick reference table + detailed patterns + edge cases + WP nuances
- Four reference docs planned:
  - `architecture-patterns.md` — Plugin file organization, singleton vs static patterns, autoloading, dependency injection, main plugin class patterns, conditional loading
  - `hooks-guide.md` — Action/filter lifecycle, priority system, hook removal patterns, custom hooks, pluggable functions, hook naming conventions
  - `oop-patterns.md` — Class-based plugin architecture, namespaces, autoloading (PSR-4), trait usage, abstract classes for extensibility, service container patterns
  - `api-patterns.md` — Settings API complete workflow, Options API best practices, REST API endpoint registration (non-security aspects like schema validation, sanitize_callback, default values), Transients API for plugin data, custom database tables with dbDelta

**Quick scan vs full review boundary:**
- `/wp-plugin` (quick scan): Grep-based pattern detection — scan for missing plugin headers, missing text domain, unprefixed functions/classes, direct file access without ABSPATH check, deprecated functions, missing activation/deactivation hooks, global namespace pollution
- `/wp-plugin-review` (full review): Complete skill workflow — file structure analysis, architecture review, WordPress.org compliance check, hooks analysis, i18n audit, Settings API review, produces structured report
- Same boundary pattern as security/performance skills

### Claude's Discretion

- Exact grep patterns for quick detection (tuned during implementation)
- Ordering of check categories within SKILL.md sections
- Depth of OOP pattern coverage (enough to guide good architecture, not a PHP design patterns textbook)
- Whether to include a "Migration Patterns" section for classic-to-modern plugin updates (likely brief mention only)
- Platform context section depth (WordPress.org vs private distribution)

### Deferred Ideas (OUT OF SCOPE)

None — all plugin development concerns fit within phase scope
</user_constraints>

## Standard Stack

The established APIs and functions for WordPress plugin development:

### Core Plugin Lifecycle Functions
| Function | Purpose | When to Use |
|----------|---------|-------------|
| `register_activation_hook()` | Execute code on plugin activation | Setting default options, creating DB tables, flushing rewrites |
| `register_deactivation_hook()` | Execute code on plugin deactivation | Removing temp data, clearing cache, flushing rewrites |
| `register_uninstall_hook()` | Execute code on plugin deletion | Cleanup before deletion (alternative to uninstall.php) |
| `uninstall.php` | Magic file for uninstall cleanup | Preferred method - delete options, drop tables, remove data |

**Version:** WordPress 6.0+ (stable APIs, unchanged since WP 3.x)

### Plugin Header Fields
| Field | Required | Purpose | Example |
|-------|----------|---------|---------|
| `Plugin Name` | Yes | Display name in admin | `Plugin Name: My Awesome Plugin` |
| `Version` | No | Current version number | `Version: 1.2.3` |
| `Plugin URI` | No | Plugin homepage URL | `Plugin URI: https://example.com` |
| `Description` | No | Short description (max 140 chars) | `Description: Does amazing things` |
| `Author` | No | Author name(s) | `Author: John Doe, Jane Smith` |
| `Author URI` | No | Author website | `Author URI: https://author.com` |
| `Text Domain` | No | i18n text domain (must match slug) | `Text Domain: my-awesome-plugin` |
| `Domain Path` | No | Translation file location | `Domain Path: /languages` |
| `Requires at least` | No | Minimum WP version | `Requires at least: 6.0` |
| `Requires PHP` | No | Minimum PHP version | `Requires PHP: 7.4` |
| `License` | No | License slug (GPLv2+ recommended) | `License: GPLv2 or later` |
| `License URI` | No | License text URL | `License URI: https://www.gnu.org/licenses/gpl-2.0.html` |
| `Update URI` | No | Prevents WordPress.org conflicts | `Update URI: https://example.com/updates` |
| `Requires Plugins` | No | Comma-separated plugin dependencies | `Requires Plugins: woocommerce, jetpack` |

**Version:** WordPress 6.0+

### Custom Post Type & Taxonomy Functions
| Function | Purpose | Hook |
|----------|---------|------|
| `register_post_type()` | Register custom post type | `init` (priority 10 or later) |
| `register_taxonomy()` | Register custom taxonomy | `init` (priority 10 or later) |
| `flush_rewrite_rules()` | Refresh permalink structure | Activation hook only (expensive) |

**Key requirements:**
- Post type keys: max 20 chars, lowercase alphanumeric + dash/underscore
- Taxonomy names: max 32 chars, lowercase letters + underscore only
- Always prefix custom post types to avoid query var conflicts
- Register taxonomies even when using `taxonomies` arg in `register_post_type()`

### Settings API Functions
| Function | Purpose | Hook |
|----------|---------|------|
| `register_setting()` | Declare setting option name | `admin_init` |
| `add_settings_section()` | Create settings section on page | `admin_init` |
| `add_settings_field()` | Add field to settings section | `admin_init` |
| `settings_fields()` | Output nonce/option group fields | In form HTML |
| `do_settings_sections()` | Output registered sections/fields | In form HTML |

**Best practices:**
- Always include `sanitize_callback` in `register_setting()`
- Set `type` parameter (string, number, boolean, array)
- Use `default` parameter for initial values
- All Settings API functions must hook to `admin_init`

### Hooks System Functions
| Function | Purpose | Parameters |
|----------|---------|-----------|
| `add_action()` | Register action callback | `$hook_name, $callback, $priority = 10, $accepted_args = 1` |
| `add_filter()` | Register filter callback | `$hook_name, $callback, $priority = 10, $accepted_args = 1` |
| `remove_action()` | Unregister action | Must match exact registration params (priority, args) |
| `remove_filter()` | Unregister filter | Must match exact registration params (priority, args) |
| `do_action()` | Fire custom action | For creating custom hooks |
| `apply_filters()` | Fire custom filter | For creating custom hooks |

**Priority notes:**
- Default priority: 10
- Lower numbers = earlier execution (1 runs before 10)
- WordPress core uses priority 10
- Same priority = order of registration
- Filters MUST return a value

### Internationalization (i18n) Functions
| Function | Purpose | Returns |
|----------|---------|---------|
| `__()` | Translate string | String |
| `_e()` | Echo translated string | void (echoes) |
| `_n()` | Translate with singular/plural | String |
| `_x()` | Translate with context | String |
| `_ex()` | Echo translate with context | void (echoes) |
| `esc_html__()` | Translate + escape HTML | String |
| `esc_html_e()` | Echo translate + escape HTML | void (echoes) |
| `esc_attr__()` | Translate + escape attribute | String |
| `esc_attr_e()` | Echo translate + escape attribute | void (echoes) |
| `load_plugin_textdomain()` | Load translation files | bool |

**Critical requirements:**
- Text domain MUST match plugin slug (folder/file name)
- Text domain: lowercase, dashes (not underscores), no spaces
- Always pass text domain to every i18n function call
- Never embed variables in translatable strings (use placeholders with `sprintf()`)
- Load translations on `init` hook (WordPress 4.6+ auto-loads from translate.wordpress.org)

### REST API Functions
| Function | Purpose | Hook |
|----------|---------|------|
| `register_rest_route()` | Register REST endpoint | `rest_api_init` |
| `rest_ensure_response()` | Convert data to REST response | In endpoint callback |
| `WP_REST_Request` | REST request object | Endpoint callback parameter |
| `WP_REST_Response` | REST response object | Return from callback |

**Required parameters:**
- `namespace` - vendor/plugin prefix + version (e.g., `myplugin/v1`)
- `route` - endpoint path (e.g., `/posts/(?P<id>\d+)`)
- `methods` - HTTP methods (`GET`, `POST`, `PUT`, `DELETE`)
- `callback` - function to execute
- `permission_callback` - authorization check (REQUIRED WP 5.5+, use `__return_true` for public)

**Best practices:**
- Use namespaces to avoid route conflicts
- Include version in namespace for API evolution
- Always validate/sanitize via `args` parameter
- Use `sanitize_callback` and `validate_callback` for params

### Plugin Check (PCP) Standards
Plugin Check tool enforces WordPress.org submission standards:

**Static Checks (PHP_CodeSniffer):**
- Proper plugin headers
- Text domain matches slug
- No direct file access (missing `ABSPATH` check)
- Correct i18n function usage
- No deprecated functions
- Coding standards compliance

**Dynamic Checks (runtime):**
- Plugin activates without errors
- No fatal errors or warnings
- Performance checks (enqueued scripts/styles size)

**Best practice:** Run Plugin Check locally before WordPress.org submission to catch violations early.

**Installation:**
```bash
# Via WordPress Admin: Plugins > Add New > Search "Plugin Check"
# Via WP-CLI:
wp plugin install plugin-check --activate
wp plugin check my-plugin.php
```

## Architecture Patterns

### Recommended Plugin File Structure
```
/my-plugin
├── my-plugin.php          # Main plugin file with header
├── uninstall.php          # Uninstall cleanup (preferred over hook)
├── readme.txt             # WordPress.org readme (required for .org)
├── /languages             # Translation .pot/.po/.mo files
├── /includes              # Core plugin classes/functions
├── /admin                 # Admin-specific code
│   ├── /css
│   ├── /js
│   └── /images
└── /public                # Public-facing code
    ├── /css
    ├── /js
    └── /images
```

**Why this structure:**
- Separates admin from public code (conditional loading)
- Clear asset organization
- Standard structure aids collaboration
- Matches WordPress Plugin Boilerplate

### Pattern 1: Prefixing for Namespace Collision Avoidance

**What:** All globally-accessible code requires a unique prefix (4-5+ characters).

**When to use:** Always, unless using PHP namespaces.

**What to prefix:**
- Functions (unless namespaced)
- Classes, interfaces, traits (unless namespaced)
- Global variables
- Options and transients

**Example:**
```php
// ❌ BAD: No prefix - collision risk with other plugins
function save_settings() {
    update_option( 'plugin_settings', $_POST['data'] );
}

add_action( 'admin_init', 'save_settings' );

// ✅ GOOD: Prefixed function name
function mypl_save_settings() {
    update_option( 'mypl_settings', $_POST['data'] );
}

add_action( 'admin_init', 'mypl_save_settings' );

// ✅ BETTER: Use PHP namespaces
namespace MyPlugin\Admin;

function save_settings() {
    update_option( 'mypl_settings', $_POST['data'] );
}

add_action( 'admin_init', __NAMESPACE__ . '\save_settings' );
```

**Source:** [WordPress Plugin Best Practices](https://developer.wordpress.org/plugins/plugin-basics/best-practices/)

### Pattern 2: Activation/Deactivation/Uninstall Lifecycle

**What:** Execute code at plugin lifecycle events.

**When to use:**
- Activation: Initial setup (default options, DB tables, flush rewrites)
- Deactivation: Temporary cleanup (clear cache, remove temp data)
- Uninstall: Permanent cleanup (delete options, drop tables)

**Example:**
```php
// Main plugin file: my-plugin.php

// Activation hook
register_activation_hook( __FILE__, 'mypl_activate' );

function mypl_activate() {
    // Set default options
    add_option( 'mypl_version', '1.0.0' );

    // Create custom database table
    global $wpdb;
    $table_name = $wpdb->prefix . 'mypl_data';

    $sql = "CREATE TABLE {$table_name} (
        id mediumint(9) NOT NULL AUTO_INCREMENT,
        data text NOT NULL,
        PRIMARY KEY  (id)
    ) {$wpdb->get_charset_collate()};";

    require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
    dbDelta( $sql ); // Use dbDelta, not $wpdb->query()

    // Flush rewrite rules ONLY on activation
    flush_rewrite_rules();
}

// Deactivation hook
register_deactivation_hook( __FILE__, 'mypl_deactivate' );

function mypl_deactivate() {
    // Clear transients
    delete_transient( 'mypl_cache' );

    // Flush rewrite rules
    flush_rewrite_rules();

    // DON'T delete permanent data here - use uninstall instead
}

// Uninstall method 1: uninstall.php (PREFERRED)
// Create file: /my-plugin/uninstall.php

if ( ! defined( 'WP_UNINSTALL_PLUGIN' ) ) {
    exit;
}

// Delete options
delete_option( 'mypl_version' );
delete_option( 'mypl_settings' );

// Drop custom table
global $wpdb;
$wpdb->query( "DROP TABLE IF EXISTS {$wpdb->prefix}mypl_data" );

// Clean up user meta
delete_metadata( 'user', 0, 'mypl_user_pref', '', true );

// Uninstall method 2: register_uninstall_hook() (alternative)
// Only use if uninstall.php doesn't exist
register_uninstall_hook( __FILE__, 'mypl_uninstall' );

function mypl_uninstall() {
    // Same cleanup code as uninstall.php
}
```

**Critical distinction:**
- Deactivation = temporary (user may reactivate) - keep data
- Uninstall = permanent (user deleted plugin) - remove everything

**Source:** [WordPress Activation/Deactivation Hooks](https://developer.wordpress.org/plugins/plugin-basics/activation-deactivation-hooks/)

### Pattern 3: Settings API Complete Workflow

**What:** WordPress's standardized way to create admin settings pages with automatic sanitization, validation, and security.

**When to use:** Any plugin with configurable options in admin area.

**Example:**
```php
// Hook all Settings API functions to admin_init
add_action( 'admin_init', 'mypl_register_settings' );

function mypl_register_settings() {
    // Register setting (creates option in database)
    register_setting(
        'mypl_options_group',    // Option group (used in settings_fields())
        'mypl_settings',          // Option name (database key)
        array(
            'type'              => 'array',
            'sanitize_callback' => 'mypl_sanitize_settings',
            'default'           => array(
                'api_key'  => '',
                'enabled'  => false,
            ),
        )
    );

    // Add settings section
    add_settings_section(
        'mypl_main_section',           // Section ID
        'Main Settings',               // Section title
        'mypl_section_callback',       // Callback for section description
        'mypl-settings'                // Page slug (used in do_settings_sections())
    );

    // Add settings fields
    add_settings_field(
        'mypl_api_key',                // Field ID
        'API Key',                     // Field label
        'mypl_api_key_callback',       // Callback to render field HTML
        'mypl-settings',               // Page slug (must match section)
        'mypl_main_section'            // Section ID (must match add_settings_section)
    );

    add_settings_field(
        'mypl_enabled',
        'Enable Feature',
        'mypl_enabled_callback',
        'mypl-settings',
        'mypl_main_section'
    );
}

// Section description callback
function mypl_section_callback() {
    echo '<p>Configure your plugin settings below.</p>';
}

// Field render callbacks
function mypl_api_key_callback() {
    $options = get_option( 'mypl_settings' );
    $value = isset( $options['api_key'] ) ? $options['api_key'] : '';
    ?>
    <input type="text"
           name="mypl_settings[api_key]"
           value="<?php echo esc_attr( $value ); ?>"
           class="regular-text">
    <?php
}

function mypl_enabled_callback() {
    $options = get_option( 'mypl_settings' );
    $checked = isset( $options['enabled'] ) && $options['enabled'];
    ?>
    <input type="checkbox"
           name="mypl_settings[enabled]"
           value="1"
           <?php checked( $checked, true ); ?>>
    <?php
}

// Sanitization callback
function mypl_sanitize_settings( $input ) {
    $sanitized = array();

    if ( isset( $input['api_key'] ) ) {
        $sanitized['api_key'] = sanitize_text_field( $input['api_key'] );
    }

    $sanitized['enabled'] = isset( $input['enabled'] ) && $input['enabled'] ? true : false;

    return $sanitized;
}

// Settings page HTML
function mypl_settings_page() {
    ?>
    <div class="wrap">
        <h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
        <form method="post" action="options.php">
            <?php
            // Output nonce, action, option_page fields
            settings_fields( 'mypl_options_group' );

            // Output sections and fields
            do_settings_sections( 'mypl-settings' );

            submit_button();
            ?>
        </form>
    </div>
    <?php
}
```

**Source:** [WordPress Settings API](https://developer.wordpress.org/plugins/settings/using-settings-api/)

### Pattern 4: Hooks System (Actions and Filters)

**What:** WordPress event-driven architecture using actions (do something) and filters (modify data).

**Priority system:**
- Default priority: 10
- Lower numbers run earlier (1 before 10)
- WordPress core uses priority 10
- Same priority = registration order

**Example:**
```php
// ❌ BAD: No priority control - may run after core
add_action( 'init', 'mypl_register_post_type' );

function mypl_register_post_type() {
    register_post_type( 'book', array( /* args */ ) );
}

// ✅ GOOD: Lower priority runs before core (priority 10)
add_action( 'init', 'mypl_register_post_type', 9 );

// ✅ GOOD: Higher priority runs after core
add_action( 'wp_enqueue_scripts', 'mypl_enqueue_scripts', 20 );

function mypl_enqueue_scripts() {
    wp_enqueue_script( 'mypl-script', plugin_dir_url( __FILE__ ) . 'js/script.js', array( 'jquery' ), '1.0', true );
}

// Filter example - MUST return value
add_filter( 'the_content', 'mypl_add_footer', 10, 1 );

function mypl_add_footer( $content ) {
    if ( is_single() ) {
        $content .= '<p>Added by my plugin</p>';
    }
    return $content; // CRITICAL: Always return in filters
}

// Multiple parameters - declare accepted_args
add_filter( 'wp_mail', 'mypl_modify_email', 10, 1 );

function mypl_modify_email( $args ) {
    // $args is array: to, subject, message, headers, attachments
    $args['from'] = 'Custom From <custom@example.com>';
    return $args;
}

// Removing hooks - MUST match exact registration params
add_action( 'init', 'mypl_setup', 15, 2 );

// To remove, priority and accepted_args must match
remove_action( 'init', 'mypl_setup', 15, 2 ); // ✅ GOOD: Exact match

// ❌ BAD: Won't work - priority/args don't match
remove_action( 'init', 'mypl_setup' ); // Missing priority 15
```

**Source:** [WordPress Hooks](https://developer.wordpress.org/plugins/hooks/)

### Pattern 5: Custom Post Types and Taxonomies

**What:** Register custom content types and taxonomies.

**Critical requirements:**
- Post type keys: max 20 chars, lowercase alphanumeric + dash/underscore
- Taxonomy names: max 32 chars, lowercase letters + underscore only
- Register on `init` hook (not before)
- Always prefix to avoid query var conflicts
- Flush rewrites on activation/deactivation

**Example:**
```php
add_action( 'init', 'mypl_register_post_type_and_taxonomy' );

function mypl_register_post_type_and_taxonomy() {
    // Register custom post type
    register_post_type(
        'mypl_book',
        array(
            'labels'       => array(
                'name'          => __( 'Books', 'my-plugin' ),
                'singular_name' => __( 'Book', 'my-plugin' ),
            ),
            'public'       => true,
            'has_archive'  => true,
            'rewrite'      => array( 'slug' => 'books' ),
            'supports'     => array( 'title', 'editor', 'thumbnail' ),
            'show_in_rest' => true, // Gutenberg support
            'taxonomies'   => array( 'mypl_genre' ), // Associate taxonomy
        )
    );

    // Register custom taxonomy
    register_taxonomy(
        'mypl_genre',
        'mypl_book',        // Object type (post type)
        array(
            'labels'       => array(
                'name'          => __( 'Genres', 'my-plugin' ),
                'singular_name' => __( 'Genre', 'my-plugin' ),
            ),
            'hierarchical' => true,  // Like categories (false = like tags)
            'public'       => true,
            'show_in_rest' => true,  // Gutenberg support
        )
    );
}

// CRITICAL: Flush rewrites on activation
register_activation_hook( __FILE__, 'mypl_activate' );

function mypl_activate() {
    // Register CPT/taxonomy before flushing
    mypl_register_post_type_and_taxonomy();

    // Flush rewrite rules (expensive - only on activation)
    flush_rewrite_rules();
}

// CRITICAL: Flush rewrites on deactivation
register_deactivation_hook( __FILE__, 'mypl_deactivate' );

function mypl_deactivate() {
    // Flush rewrite rules to remove custom CPT permalinks
    flush_rewrite_rules();
}
```

**Common mistakes:**
- Post type key > 20 chars → slug truncation, permalink issues
- Taxonomy name > 32 chars → database errors
- Missing `flush_rewrite_rules()` on activation → 404 errors on CPT pages
- Calling `flush_rewrite_rules()` on every `init` → performance issue

**Source:** [register_post_type()](https://developer.wordpress.org/reference/functions/register_post_type/), [register_taxonomy()](https://developer.wordpress.org/reference/functions/register_taxonomy/)

### Pattern 6: Internationalization (i18n) Workflow

**What:** Make plugin translatable for international audiences.

**Text domain requirements:**
- MUST match plugin slug (folder/file name)
- Lowercase, dashes (not underscaces), no spaces
- Declared in plugin header AND every i18n function call

**Example:**
```php
// Plugin header
/**
 * Plugin Name: My Awesome Plugin
 * Text Domain: my-awesome-plugin
 * Domain Path: /languages
 */

// Load translations on init (optional in WP 4.6+)
add_action( 'init', 'mypl_load_textdomain' );

function mypl_load_textdomain() {
    load_plugin_textdomain(
        'my-awesome-plugin',
        false,
        dirname( plugin_basename( __FILE__ ) ) . '/languages'
    );
}

// ❌ BAD: Variable embedded in translatable string
$count = 5;
echo __( "You have $count messages", 'my-awesome-plugin' );

// ✅ GOOD: Use placeholders with sprintf()
$count = 5;
echo sprintf(
    __( 'You have %d messages', 'my-awesome-plugin' ),
    $count
);

// ❌ BAD: Missing text domain
echo __( 'Save Settings' );

// ✅ GOOD: Always include text domain
echo __( 'Save Settings', 'my-awesome-plugin' );

// Pluralization
$count = 3;
echo sprintf(
    _n(
        '%d item found',     // Singular
        '%d items found',    // Plural
        $count,              // Number
        'my-awesome-plugin'  // Text domain
    ),
    $count
);

// Context-aware translation (same English, different translations)
echo _x( 'Post', 'noun', 'my-awesome-plugin' );  // "Post" as in blog post
echo _x( 'Post', 'verb', 'my-awesome-plugin' );  // "Post" as in submit

// Escaped output + translation
echo '<h1>' . esc_html__( 'Settings', 'my-awesome-plugin' ) . '</h1>';
echo '<input type="text" placeholder="' . esc_attr__( 'Enter name', 'my-awesome-plugin' ) . '">';
```

**Source:** [WordPress i18n](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/)

### Pattern 7: Direct File Access Prevention

**What:** Prevent direct access to plugin PHP files via URL.

**When to use:** Top of every plugin PHP file.

**Example:**
```php
// ✅ GOOD: ABSPATH check (most common)
if ( ! defined( 'ABSPATH' ) ) {
    exit; // Exit if accessed directly
}

// Alternative patterns
defined( 'ABSPATH' ) || exit;
defined( 'WPINC' ) || die;

// ❌ BAD: No protection
<?php
// Plugin code here - accessible via direct URL
```

**Why it matters:**
- Prevents information disclosure (error messages, file paths)
- WordPress.org requirement (Plugin Check enforces this)
- Prevents execution outside WordPress context

### Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Fix |
|--------------|-------------|-----|
| **Global namespace pollution** | Function/class name collisions with other plugins | Always prefix or use PHP namespaces |
| **Using query_posts()** | Breaks main query, pagination, conditionals | Use `WP_Query` for secondary queries, `pre_get_posts` for main query |
| **Flushing rewrites on every init** | Expensive DB operation on every page load | Only flush on activation/deactivation hooks |
| **Direct DB table creation with $wpdb->query()** | Doesn't handle version/charset/collation upgrades | Use `dbDelta()` from `wp-admin/includes/upgrade.php` |
| **Text domain mismatch** | Breaks translations, Plugin Check fails | Text domain MUST match plugin slug exactly |
| **Missing ABSPATH check** | Direct file access possible, WordPress.org rejection | Add `defined( 'ABSPATH' ) || exit;` to all files |
| **Hardcoded paths like /wp-content/plugins/** | Breaks on custom WordPress installations | Use `plugin_dir_path()`, `plugin_dir_url()` |
| **Role checks instead of capabilities** | Breaks on custom roles, brittle authorization | Always use `current_user_can( 'capability' )` |
| **Missing uninstall cleanup** | Leaves plugin data after deletion | Create `uninstall.php` or use `register_uninstall_hook()` |
| **Variables in translatable strings** | Breaks translation extraction | Use placeholders with `sprintf()` |
| **Using lowercase slug in text domain** | Text domain must be lowercase with dashes | `my-plugin` not `My_Plugin` or `my_plugin` |

## Don't Hand-Roll

Plugin development problems that have established WordPress solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| **Settings pages** | Custom HTML forms with manual option handling | WordPress Settings API | Automatic nonces, validation, sanitization, standardized UI |
| **Admin menus** | Manual HTML menu generation | `add_menu_page()`, `add_submenu_page()` | Capability checks, proper ordering, WordPress admin styles |
| **Database table creation** | `$wpdb->query( "CREATE TABLE..." )` | `dbDelta()` from `wp-admin/includes/upgrade.php` | Handles upgrades, charset, collation, table structure changes |
| **Permalink structure** | Manual URL routing | `register_post_type()` with `rewrite` arg + `flush_rewrite_rules()` | WordPress handles rewrites, SEO-friendly, automatic |
| **Internationalization** | Custom translation system | WordPress i18n functions (`__()`, `_e()`, etc.) | Integrates with translate.wordpress.org, standard tooling |
| **Admin notices** | Custom HTML with cookies for dismissal | `admin_notices` action hook + transients | Standard WordPress UI, proper styling, dismissible |
| **Option management** | Custom database tables for settings | `add_option()`, `update_option()`, `get_option()` | Autoloading, caching, Multisite support |
| **Plugin lifecycle** | Manual version checks, custom activation logic | `register_activation_hook()`, `register_deactivation_hook()`, `uninstall.php` | WordPress handles plugin state, proper timing |
| **REST endpoints** | Custom AJAX with `admin-ajax.php` | `register_rest_route()` | Lighter bootstrap, automatic JSON responses, better caching |
| **Transients for caching** | Custom cache files or database tables | `set_transient()`, `get_transient()` | Object cache integration, expiration handling, Multisite support |

**Key insight:** WordPress provides extensive APIs that handle edge cases, Multisite compatibility, caching, security, and internationalization. Custom implementations miss these benefits and create maintenance burden.

## Common Pitfalls

### Pitfall 1: Missing Text Domain or Text Domain Mismatch (WordPress.org Rejection)

**What goes wrong:** Plugin Check fails with "text domain does not match plugin slug" or i18n functions missing text domain.

**Why it happens:** Text domain added to plugin header but not to i18n function calls, or text domain uses underscores instead of dashes.

**Context:** WordPress.org requires text domain to EXACTLY match plugin slug (folder or file name). Case-sensitive, dashes not underscores.

**How to avoid:**
1. Plugin slug (folder/file): `my-awesome-plugin` → Text domain MUST be: `my-awesome-plugin`
2. Add text domain to plugin header: `Text Domain: my-awesome-plugin`
3. Add text domain to EVERY i18n function: `__( 'Text', 'my-awesome-plugin' )`
4. Run Plugin Check before submission

**Warning signs:**
```php
// ❌ BAD: Text domain mismatch
// File: my-awesome-plugin/my-awesome-plugin.php
// Text Domain: my_awesome_plugin (underscores don't match folder dashes)
echo __( 'Settings', 'MyAwesomePlugin' ); // Wrong case

// ✅ GOOD: Exact match
// File: my-awesome-plugin/my-awesome-plugin.php
// Text Domain: my-awesome-plugin
echo __( 'Settings', 'my-awesome-plugin' );
```

### Pitfall 2: Calling flush_rewrite_rules() on Every Init (Performance Issue)

**What goes wrong:** Plugin calls `flush_rewrite_rules()` on `init` hook, causing DB write on every page load.

**Why it happens:** Developer wants to ensure permalinks work for custom post types, doesn't understand activation hook timing.

**Context:** `flush_rewrite_rules()` is expensive (regenerates entire rewrite rule array, writes to DB). Should only run on activation/deactivation.

**How to avoid:** Only flush rewrites in activation/deactivation hooks, never on `init` or regular page loads.

**Warning signs:**
```php
// ❌ CRITICAL: Flushes on every page load
add_action( 'init', 'mypl_register_cpt' );

function mypl_register_cpt() {
    register_post_type( 'book', array( /* args */ ) );
    flush_rewrite_rules(); // BAD: Runs on every request
}

// ✅ GOOD: Flush only on activation
add_action( 'init', 'mypl_register_cpt' );

function mypl_register_cpt() {
    register_post_type( 'book', array( /* args */ ) );
}

register_activation_hook( __FILE__, 'mypl_activate' );

function mypl_activate() {
    mypl_register_cpt(); // Register CPT first
    flush_rewrite_rules(); // Then flush (only on activation)
}
```

### Pitfall 3: Creating Database Tables with $wpdb->query() Instead of dbDelta()

**What goes wrong:** Plugin creates custom DB table using `$wpdb->query( "CREATE TABLE..." )`, table doesn't handle charset/collation or version upgrades properly.

**Why it happens:** Developer unfamiliar with `dbDelta()` function and its special SQL formatting requirements.

**Context:** `dbDelta()` compares existing table structure with desired structure and upgrades intelligently. Handles charset, collation, and schema changes across versions.

**How to avoid:** Always use `dbDelta()` from `wp-admin/includes/upgrade.php` for table creation/updates.

**Warning signs:**
```php
// ❌ BAD: Direct CREATE TABLE (no upgrade path)
register_activation_hook( __FILE__, 'mypl_create_table' );

function mypl_create_table() {
    global $wpdb;
    $table_name = $wpdb->prefix . 'mypl_data';

    $wpdb->query( "CREATE TABLE $table_name (
        id int(11) NOT NULL AUTO_INCREMENT,
        data text,
        PRIMARY KEY (id)
    )" );
}

// ✅ GOOD: Use dbDelta() with proper SQL formatting
register_activation_hook( __FILE__, 'mypl_create_table' );

function mypl_create_table() {
    global $wpdb;
    $table_name = $wpdb->prefix . 'mypl_data';
    $charset_collate = $wpdb->get_charset_collate();

    // dbDelta requires specific SQL format:
    // - Two spaces after PRIMARY KEY
    // - No spaces around = in variable assignments
    // - Each field on its own line
    $sql = "CREATE TABLE $table_name (
        id mediumint(9) NOT NULL AUTO_INCREMENT,
        data text NOT NULL,
        created datetime DEFAULT '0000-00-00 00:00:00' NOT NULL,
        PRIMARY KEY  (id)
    ) $charset_collate;";

    require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
    dbDelta( $sql );
}
```

### Pitfall 4: Post Type Key Exceeds 20 Characters (Registration Fails)

**What goes wrong:** Custom post type key is longer than 20 characters, causing permalink issues and query var conflicts.

**Why it happens:** Developer uses descriptive name without understanding length limit.

**Context:** WordPress limits post type keys to 20 characters for URL slug compatibility and database field size.

**How to avoid:** Keep post type keys under 20 chars, use abbreviations, always prefix.

**Warning signs:**
```php
// ❌ CRITICAL: 29 characters (exceeds 20 limit)
register_post_type( 'my_awesome_plugin_custom_book', array( /* ... */ ) );

// ✅ GOOD: 9 characters with prefix
register_post_type( 'mypl_book', array( /* ... */ ) );

// ✅ GOOD: 15 characters with abbreviation
register_post_type( 'mypl_cust_book', array( /* ... */ ) );
```

### Pitfall 5: Missing register_setting() for Settings API Fields (Settings Don't Save)

**What goes wrong:** Settings page displays fields via `add_settings_field()` but they don't save when form is submitted.

**Why it happens:** Developer adds fields without calling `register_setting()` to declare the option name.

**Context:** Settings API requires `register_setting()` to whitelist option names for saving. Without it, `settings_fields()` doesn't know what to save.

**How to avoid:** Always call `register_setting()` for each option before adding fields.

**Warning signs:**
```php
// ❌ BAD: add_settings_field() without register_setting()
add_action( 'admin_init', 'mypl_settings_init' );

function mypl_settings_init() {
    add_settings_section( 'mypl_section', 'Settings', null, 'mypl-settings' );

    add_settings_field(
        'mypl_api_key',
        'API Key',
        'mypl_api_key_callback',
        'mypl-settings',
        'mypl_section'
    );
    // Missing register_setting() - field won't save!
}

// ✅ GOOD: register_setting() before add_settings_field()
add_action( 'admin_init', 'mypl_settings_init' );

function mypl_settings_init() {
    // CRITICAL: Register setting first
    register_setting(
        'mypl_options_group',
        'mypl_settings',
        array( 'sanitize_callback' => 'mypl_sanitize' )
    );

    add_settings_section( 'mypl_section', 'Settings', null, 'mypl-settings' );

    add_settings_field(
        'mypl_api_key',
        'API Key',
        'mypl_api_key_callback',
        'mypl-settings',
        'mypl_section'
    );
}
```

### Pitfall 6: Not Returning Value from Filter Callback (Fatal Error)

**What goes wrong:** Filter callback doesn't return a value, causing fatal error or breaking filter chain.

**Why it happens:** Developer confuses actions (no return) with filters (must return).

**Context:** Filters modify data and pass it along. Not returning breaks the chain and causes errors.

**How to avoid:** ALWAYS return a value from filter callbacks, even if unchanged.

**Warning signs:**
```php
// ❌ CRITICAL: Filter doesn't return value (fatal error)
add_filter( 'the_content', 'mypl_modify_content' );

function mypl_modify_content( $content ) {
    if ( is_single() ) {
        $content .= '<p>Footer text</p>';
    }
    // Missing return! Fatal error.
}

// ✅ GOOD: Always return value
add_filter( 'the_content', 'mypl_modify_content' );

function mypl_modify_content( $content ) {
    if ( is_single() ) {
        $content .= '<p>Footer text</p>';
    }
    return $content; // CRITICAL: Always return
}
```

### Pitfall 7: Missing permission_callback in REST Route (WordPress 5.5+ Error)

**What goes wrong:** REST endpoint registered without `permission_callback`, WordPress 5.5+ throws PHP warning and blocks endpoint.

**Why it happens:** Developer writes code for pre-5.5 WordPress, doesn't update for new requirement.

**Context:** WordPress 5.5 made `permission_callback` required for all REST routes to force explicit authorization decisions.

**How to avoid:** Always include `permission_callback`. Use `__return_true` for public endpoints, `current_user_can()` callback for protected.

**Warning signs:**
```php
// ❌ CRITICAL: Missing permission_callback (WP 5.5+ error)
add_action( 'rest_api_init', 'mypl_register_routes' );

function mypl_register_routes() {
    register_rest_route( 'mypl/v1', '/posts', array(
        'methods'  => 'GET',
        'callback' => 'mypl_get_posts',
        // Missing: 'permission_callback'
    ) );
}

// ✅ GOOD: Public endpoint with explicit permission_callback
add_action( 'rest_api_init', 'mypl_register_routes' );

function mypl_register_routes() {
    register_rest_route( 'mypl/v1', '/posts', array(
        'methods'             => 'GET',
        'callback'            => 'mypl_get_posts',
        'permission_callback' => '__return_true', // Explicitly public
    ) );
}

// ✅ GOOD: Protected endpoint with capability check
register_rest_route( 'mypl/v1', '/admin/settings', array(
    'methods'             => 'POST',
    'callback'            => 'mypl_save_settings',
    'permission_callback' => function() {
        return current_user_can( 'manage_options' );
    },
) );
```

### Pitfall 8: Variables Embedded in Translatable Strings (Breaks Translation)

**What goes wrong:** Translation tools can't extract strings with embedded PHP variables, translators can't translate properly.

**Why it happens:** Developer doesn't understand how `gettext` extraction works.

**Context:** Translation tools scan for `__()`, `_e()` etc. and extract the exact string literal. Variables break this.

**How to avoid:** Use placeholders with `sprintf()` for dynamic content.

**Warning signs:**
```php
// ❌ BAD: Variable embedded (translation tools can't extract)
$count = 5;
echo __( "You have $count messages", 'my-plugin' );

// ❌ BAD: Concatenation breaks extraction
echo __( 'You have ' . $count . ' messages', 'my-plugin' );

// ✅ GOOD: Placeholder with sprintf()
$count = 5;
echo sprintf(
    __( 'You have %d messages', 'my-plugin' ),
    $count
);

// ✅ GOOD: Multiple placeholders
echo sprintf(
    __( 'Hello %1$s, you have %2$d messages from %3$s', 'my-plugin' ),
    $user_name,
    $count,
    $sender_name
);
```

## Code Examples

Verified patterns from official sources and research:

### Complete Plugin Structure
```php
/**
 * Plugin Name: My Awesome Plugin
 * Plugin URI: https://example.com/my-awesome-plugin
 * Description: Does amazing things with WordPress
 * Version: 1.0.0
 * Requires at least: 6.0
 * Requires PHP: 7.4
 * Author: John Doe
 * Author URI: https://example.com
 * License: GPL v2 or later
 * License URI: https://www.gnu.org/licenses/gpl-2.0.html
 * Text Domain: my-awesome-plugin
 * Domain Path: /languages
 */

// Direct file access prevention
if ( ! defined( 'ABSPATH' ) ) {
    exit;
}

// Define plugin constants
define( 'MYPL_VERSION', '1.0.0' );
define( 'MYPL_PLUGIN_DIR', plugin_dir_path( __FILE__ ) );
define( 'MYPL_PLUGIN_URL', plugin_dir_url( __FILE__ ) );

// Activation hook
register_activation_hook( __FILE__, 'mypl_activate' );

function mypl_activate() {
    // Set default options
    add_option( 'mypl_version', MYPL_VERSION );

    // Register CPT before flushing rewrites
    mypl_register_post_types();

    // Flush rewrite rules
    flush_rewrite_rules();
}

// Deactivation hook
register_deactivation_hook( __FILE__, 'mypl_deactivate' );

function mypl_deactivate() {
    // Flush rewrite rules
    flush_rewrite_rules();
}

// Load text domain
add_action( 'init', 'mypl_load_textdomain' );

function mypl_load_textdomain() {
    load_plugin_textdomain(
        'my-awesome-plugin',
        false,
        dirname( plugin_basename( __FILE__ ) ) . '/languages'
    );
}

// Register custom post type
add_action( 'init', 'mypl_register_post_types' );

function mypl_register_post_types() {
    register_post_type(
        'mypl_book',
        array(
            'labels'       => array(
                'name'          => __( 'Books', 'my-awesome-plugin' ),
                'singular_name' => __( 'Book', 'my-awesome-plugin' ),
            ),
            'public'       => true,
            'has_archive'  => true,
            'rewrite'      => array( 'slug' => 'books' ),
            'supports'     => array( 'title', 'editor', 'thumbnail' ),
            'show_in_rest' => true,
        )
    );
}

// Settings API
add_action( 'admin_init', 'mypl_register_settings' );

function mypl_register_settings() {
    register_setting(
        'mypl_options_group',
        'mypl_settings',
        array(
            'type'              => 'array',
            'sanitize_callback' => 'mypl_sanitize_settings',
        )
    );

    add_settings_section(
        'mypl_main_section',
        __( 'Main Settings', 'my-awesome-plugin' ),
        '__return_empty_string',
        'mypl-settings'
    );

    add_settings_field(
        'mypl_api_key',
        __( 'API Key', 'my-awesome-plugin' ),
        'mypl_api_key_callback',
        'mypl-settings',
        'mypl_main_section'
    );
}

function mypl_sanitize_settings( $input ) {
    $sanitized = array();
    if ( isset( $input['api_key'] ) ) {
        $sanitized['api_key'] = sanitize_text_field( $input['api_key'] );
    }
    return $sanitized;
}

function mypl_api_key_callback() {
    $options = get_option( 'mypl_settings' );
    $value = isset( $options['api_key'] ) ? $options['api_key'] : '';
    printf(
        '<input type="text" name="mypl_settings[api_key]" value="%s" class="regular-text">',
        esc_attr( $value )
    );
}

// Admin menu
add_action( 'admin_menu', 'mypl_add_admin_menu' );

function mypl_add_admin_menu() {
    add_menu_page(
        __( 'My Plugin', 'my-awesome-plugin' ),
        __( 'My Plugin', 'my-awesome-plugin' ),
        'manage_options',
        'my-plugin',
        'mypl_settings_page',
        'dashicons-book',
        20
    );
}

function mypl_settings_page() {
    if ( ! current_user_can( 'manage_options' ) ) {
        return;
    }
    ?>
    <div class="wrap">
        <h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
        <form method="post" action="options.php">
            <?php
            settings_fields( 'mypl_options_group' );
            do_settings_sections( 'mypl-settings' );
            submit_button();
            ?>
        </form>
    </div>
    <?php
}
```

**Source:** Compiled from [WordPress Plugin Handbook](https://developer.wordpress.org/plugins/)

### REST API Endpoint Registration
```php
// Source: https://developer.wordpress.org/rest-api/extending-the-rest-api/adding-custom-endpoints/

add_action( 'rest_api_init', 'mypl_register_rest_routes' );

function mypl_register_rest_routes() {
    // Public endpoint (read-only)
    register_rest_route(
        'mypl/v1',              // Namespace (plugin/version)
        '/posts',               // Route
        array(
            'methods'             => 'GET',
            'callback'            => 'mypl_get_posts',
            'permission_callback' => '__return_true', // Public
            'args'                => array(
                'per_page' => array(
                    'default'           => 10,
                    'sanitize_callback' => 'absint',
                    'validate_callback' => function( $param ) {
                        return is_numeric( $param ) && $param > 0 && $param <= 100;
                    },
                ),
            ),
        )
    );

    // Protected endpoint (requires capability)
    register_rest_route(
        'mypl/v1',
        '/posts/(?P<id>\d+)',
        array(
            'methods'             => 'DELETE',
            'callback'            => 'mypl_delete_post',
            'permission_callback' => function( $request ) {
                return current_user_can( 'delete_posts' );
            },
            'args'                => array(
                'id' => array(
                    'validate_callback' => function( $param ) {
                        return is_numeric( $param );
                    },
                ),
            ),
        )
    );
}

function mypl_get_posts( $request ) {
    $per_page = $request->get_param( 'per_page' );

    $posts = get_posts( array(
        'posts_per_page' => $per_page,
    ) );

    return rest_ensure_response( $posts );
}

function mypl_delete_post( $request ) {
    $post_id = $request->get_param( 'id' );

    $result = wp_delete_post( $post_id, true );

    if ( ! $result ) {
        return new WP_Error(
            'delete_failed',
            __( 'Failed to delete post', 'my-awesome-plugin' ),
            array( 'status' => 500 )
        );
    }

    return rest_ensure_response( array(
        'deleted' => true,
        'id'      => $post_id,
    ) );
}
```

### Hooks Priority Management
```php
// Source: https://developer.wordpress.org/plugins/hooks/

// Default priority (10)
add_action( 'init', 'mypl_init_default' );

// Run earlier than core (priority 9)
add_action( 'init', 'mypl_init_early', 9 );

// Run later than core (priority 11)
add_action( 'init', 'mypl_init_late', 11 );

// Filter with multiple parameters
add_filter( 'wp_mail', 'mypl_modify_email', 10, 1 );

function mypl_modify_email( $args ) {
    // Modify email arguments
    $args['from'] = 'Custom From <from@example.com>';
    return $args; // CRITICAL: Always return in filters
}

// Removing hooks - MUST match exact registration
add_action( 'wp_footer', 'mypl_footer_code', 15, 2 );

// To remove, priority and accepted_args must match
remove_action( 'wp_footer', 'mypl_footer_code', 15, 2 );

// Custom hooks (for extensibility)
function mypl_process_data( $data ) {
    // Allow other plugins to modify data
    $data = apply_filters( 'mypl_before_process', $data );

    // Process data
    $result = process( $data );

    // Allow other plugins to hook after processing
    do_action( 'mypl_after_process', $result );

    return $result;
}
```

### Internationalization Complete Example
```php
// Source: https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/

// Basic translation
echo __( 'Settings saved successfully', 'my-awesome-plugin' );

// Translation with echo
_e( 'Click here to continue', 'my-awesome-plugin' );

// Escaped translation
echo '<h1>' . esc_html__( 'Welcome', 'my-awesome-plugin' ) . '</h1>';
echo '<input placeholder="' . esc_attr__( 'Enter your name', 'my-awesome-plugin' ) . '">';

// Pluralization
$count = 5;
echo sprintf(
    _n(
        'One post found',
        '%d posts found',
        $count,
        'my-awesome-plugin'
    ),
    number_format_i18n( $count )
);

// Context-aware translation
echo _x( 'Post', 'noun - blog post', 'my-awesome-plugin' );
echo _x( 'Post', 'verb - submit', 'my-awesome-plugin' );

// Multiple placeholders
echo sprintf(
    __( 'Hello %1$s, you have %2$d new messages from %3$s', 'my-awesome-plugin' ),
    $user_name,
    $message_count,
    $sender_name
);
```

## State of the Art

| Old Approach | Current Approach (WP 6.x) | When Changed | Impact |
|--------------|---------------------------|--------------|--------|
| Plugin headers in comments | Plugin headers + Requires PHP field | WP 5.2 (2019) | Server compatibility checks before activation |
| permission_callback optional | permission_callback **required** in REST routes | WP 5.5 (2020) | Forces explicit authorization decisions |
| Text domain auto-detection | Text domain MUST match slug | Ongoing enforcement | WordPress.org Plugin Check fails on mismatch |
| Manual translation loading | Auto-load from translate.wordpress.org | WP 4.6 (2016) | `load_plugin_textdomain()` optional if WP 4.6+ |
| Shortcodes for content | Blocks (Gutenberg) preferred | WP 5.0 (2018) | Shortcodes still work but blocks recommended |
| Classic widgets | Block-based widgets | WP 5.8 (2021) | Classic widgets still supported but deprecated |
| admin-ajax.php for AJAX | REST API preferred | WP 4.7+ (2016) | REST has lighter bootstrap, better caching |
| readme.txt header parsing | Headers from main PHP file | WP 5.8 (2021) | Requires PHP/WP version read from plugin file |
| Must-use plugin detection | Requires Plugins header for dependencies | WP 6.5 (2024) | Automatic dependency management |

**Deprecated/outdated:**
- **query_posts():** Use `WP_Query` or `pre_get_posts`. `query_posts()` breaks main query.
- **create_function():** Removed in PHP 8.0. Use anonymous functions or named functions.
- **Direct $wpdb->query() for table creation:** Use `dbDelta()` for proper upgrades.
- **Hardcoded /wp-content/ paths:** Use `WP_CONTENT_DIR` constant or `plugin_dir_path()`.
- **Role checks for authorization:** Use `current_user_can()` with capabilities.

## Open Questions

Things that couldn't be fully resolved:

1. **Plugin Check (PCP) exact check list**
   - What we know: Plugin Check performs static checks via PHP_CodeSniffer and dynamic checks by activating plugin
   - What's unclear: Complete list of all checks and their severity mappings (blocked submission vs warning)
   - Recommendation: Include high-confidence checks (headers, text domain, ABSPATH, deprecated functions) and test with Plugin Check tool during implementation

2. **OOP pattern depth for reference docs**
   - What we know: Modern plugins use namespaces, PSR-4 autoloading, dependency injection patterns
   - What's unclear: How deep to go into design patterns (factory, observer, strategy) without overwhelming developers
   - Recommendation: Focus on practical WordPress-specific patterns (singleton for main class, service container for dependencies, trait usage for code reuse) with brief mentions of advanced patterns

3. **Must-use vs regular plugin detection guidance**
   - What we know: Must-use plugins load differently (no activation hooks, alphabetical load order, can't be deactivated)
   - What's unclear: How to detect mu-plugin context programmatically and adjust review recommendations
   - Recommendation: Include brief section noting mu-plugin differences but keep focus on regular plugin development (99% of use cases)

4. **Migration patterns section necessity**
   - What we know: Many plugins need updates from shortcodes→blocks, classic widgets→block widgets, admin-ajax→REST
   - What's unclear: Whether to include detailed migration patterns or just flag outdated patterns with modern alternatives
   - Recommendation: Flag outdated patterns with INFO severity and point to modern alternatives (e.g., "Consider using blocks instead of shortcodes for WP 5.0+ compatibility")

## Sources

### Primary (HIGH confidence)

**WordPress Official Documentation:**
- [Plugin Handbook](https://developer.wordpress.org/plugins/)
- [Plugin Best Practices](https://developer.wordpress.org/plugins/plugin-basics/best-practices/)
- [Header Requirements](https://developer.wordpress.org/plugins/plugin-basics/header-requirements/)
- [Activation/Deactivation Hooks](https://developer.wordpress.org/plugins/plugin-basics/activation-deactivation-hooks/)
- [Uninstall Methods](https://developer.wordpress.org/plugins/plugin-basics/uninstall-methods/)
- [Using Settings API](https://developer.wordpress.org/plugins/settings/using-settings-api/)
- [Internationalization](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/)
- [WordPress.org Plugin Guidelines](https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/)
- [Plugin Readmes](https://developer.wordpress.org/plugins/wordpress-org/how-your-readme-txt-works/)
- [Actions](https://developer.wordpress.org/plugins/hooks/actions/)
- [Filters](https://developer.wordpress.org/plugins/hooks/)
- [REST API Routes and Endpoints](https://developer.wordpress.org/rest-api/extending-the-rest-api/routes-and-endpoints/)
- [Adding Custom REST Endpoints](https://developer.wordpress.org/rest-api/extending-the-rest-api/adding-custom-endpoints/)

**WordPress Function References:**
- [register_activation_hook()](https://developer.wordpress.org/reference/functions/register_activation_hook/)
- [register_deactivation_hook()](https://developer.wordpress.org/reference/functions/register_deactivation_hook/)
- [register_uninstall_hook()](https://developer.wordpress.org/reference/functions/register_uninstall_hook/)
- [register_post_type()](https://developer.wordpress.org/reference/functions/register_post_type/)
- [register_taxonomy()](https://developer.wordpress.org/reference/functions/register_taxonomy/)
- [register_setting()](https://developer.wordpress.org/reference/functions/register_setting/)
- [add_settings_section()](https://developer.wordpress.org/reference/functions/add_settings_section/)
- [add_settings_field()](https://developer.wordpress.org/reference/functions/add_settings_field/)
- [register_rest_route()](https://developer.wordpress.org/reference/functions/register_rest_route/)
- [add_action()](https://developer.wordpress.org/reference/functions/add_action/)
- [add_filter()](https://developer.wordpress.org/reference/functions/add_filter/)
- [load_plugin_textdomain()](https://developer.wordpress.org/reference/functions/load_plugin_textdomain/)

**Plugin Check (PCP) Tool:**
- [Plugin Check Plugin](https://wordpress.org/plugins/plugin-check/)
- [Introducing Plugin Check (PCP)](https://make.wordpress.org/plugins/2024/09/17/introducing-plugin-check-pcp/)
- [Plugin Check GitHub](https://github.com/WordPress/plugin-check)

### Secondary (MEDIUM confidence)

**WordPress Development Best Practices (verified with official docs):**
- [WordPress Plugin Development Best Practices (ColorWhistle)](https://colorwhistle.com/wordpress-plugin-development-best-practices/)
- [WordPress Plugin Development Best Practices (HammaniTech)](https://hammanitech.com/blog/how-to-create-a-wordpress-plugin/)
- [Building Advanced WordPress Plugins: OOP, Namespaces, Autoloading (BuddyX)](https://buddyxtheme.com/building-advanced-wordpress-plugins-oop-namespaces-autoloading-and-modern-architecture/)
- [WordPress Plugin Architecture: OOP and Design Patterns (Voxfor)](https://www.voxfor.com/wordpress-plugin-architecture-oop-design-patterns/)
- [Common WordPress Plugin Development Issues (WisdmLabs)](https://wisdmlabs.com/blog/common-wordpress-plugin-development-issues/)

**Community Resources (cross-referenced with official sources):**
- [WordPress Hooks Bootcamp (Kinsta)](https://kinsta.com/blog/wordpress-hooks/)
- [Settings API Explained (Press Coders)](https://presscoders.com/wordpress-settings-api-explained/)
- [The Definitive Guide to WordPress Internationalization (InstaWP)](https://instawp.com/wordpress-internationalization-guide/)

### Tertiary (LOW confidence - requires validation)

**General WordPress Plugin Tutorials:**
- Various 2026 WordPress plugin development guides (used for current trends, not authoritative API info)
- Community blog posts on plugin architecture (verified against official docs before including)

## Metadata

**Confidence breakdown:**
- Standard stack (plugin lifecycle hooks, headers, Settings API, hooks system, i18n): **HIGH** - Official WordPress documentation, stable APIs since WP 3.x-4.x
- Architecture patterns (file structure, prefixing, activation/deactivation): **HIGH** - WordPress Plugin Handbook and established community conventions
- Plugin Check (PCP) standards: **MEDIUM** - Official tool exists, but complete check list not fully documented (requires testing during implementation)
- OOP pattern depth: **MEDIUM** - Modern patterns established but guidance on appropriate depth for skill documentation requires discretion
- WordPress.org submission requirements: **HIGH** - Official guidelines documented, Plugin Check enforces standards
- Context-aware detection (mu-plugins, private plugins): **MEDIUM** - Distinctions documented but automated detection patterns need implementation testing

**Research date:** 2026-02-06
**Valid until:** 30-60 days (WordPress plugin APIs stable, but Plugin Check tool evolves; WordPress.org guidelines may update)
