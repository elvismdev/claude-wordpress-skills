# WordPress Authorization Patterns Reference

Authorization ensures users can only perform actions they're permitted to. This guide covers WordPress's capability-based authorization system, REST API permission callbacks, AJAX handler security, and common authorization mistakes.

## Quick Reference Table

| Function | Purpose | Returns | Use Case |
|----------|---------|---------|----------|
| `current_user_can( 'cap' )` | Check static capability | Boolean | General capability check |
| `current_user_can( 'cap', $id )` | Check meta capability | Boolean | Object-specific authorization |
| `user_can( $user, 'cap' )` | Check capability for user | Boolean | Check specific user's capabilities |
| `user_can( $user, 'cap', $id )` | Check meta cap for user | Boolean | Check user's access to specific object |
| `wp_get_current_user()` | Get current user object | WP_User | Access user data |
| `get_current_user_id()` | Get current user ID | Integer | Quick user ID check |
| `is_user_logged_in()` | Check if user is logged in | Boolean | Require authentication |
| `map_meta_cap()` | Map meta cap to primitives | Array | Custom capability mapping |
| `add_cap()` | Add capability to role | Void | Grant new capability |
| `remove_cap()` | Remove capability from role | Void | Revoke capability |

## Capabilities vs Roles

WordPress uses a capability-based system, not role-based. **Always check capabilities, never check roles.**

### Why Capabilities, Not Roles?

Roles are containers of capabilities. Checking roles breaks when:
- Custom roles are added
- Role capabilities are modified
- Multisite super admins need access
- Plugins change the capability model

```php
// ❌ BAD: Checking user role
$user = wp_get_current_user();
if ( in_array( 'administrator', $user->roles, true ) ) {
    // Breaks if custom role has permission
    // Breaks if admin caps are modified
    wp_delete_post( $post_id );
}

// ❌ BAD: Comparing user role
if ( 'editor' === $user->roles[0] ) {
    // Assumes single role
    // Breaks with custom roles
    update_post( $post_id );
}

// ✅ GOOD: Check capability
if ( current_user_can( 'delete_posts' ) ) {
    // Works regardless of role
    // Works with custom roles
    // Works with modified capabilities
    wp_delete_post( $post_id );
}

// ✅ GOOD: Check object-specific capability
if ( current_user_can( 'delete_post', $post_id ) ) {
    // Checks if user can delete THIS specific post
    wp_delete_post( $post_id );
}
```

### Role Hierarchy

WordPress doesn't enforce role hierarchy. Don't assume Administrator > Editor > Author.

```php
// ❌ BAD: Assuming role hierarchy
$user_role = $user->roles[0];
$allowed_roles = array( 'administrator', 'editor' );
if ( in_array( $user_role, $allowed_roles, true ) ) {
    // Brittle! Breaks with custom roles
}

// ✅ GOOD: Check capability regardless of hierarchy
if ( current_user_can( 'edit_others_posts' ) ) {
    // Editors and admins have this, custom roles might too
}
```

### Meta Capabilities vs Primitive Capabilities

**Primitive capabilities**: Direct permissions attached to roles
- `edit_posts`, `edit_others_posts`, `delete_posts`, `manage_options`

**Meta capabilities**: Context-dependent, mapped to primitives by `map_meta_cap()`
- `edit_post`, `delete_post`, `edit_user`, `delete_user`

```php
// ❌ BAD: Checking primitive cap for specific post
if ( current_user_can( 'edit_posts' ) ) {
    // User can edit SOME posts, but can they edit THIS post?
    wp_update_post( array( 'ID' => $post_id, 'post_title' => $title ) );
}

// ✅ GOOD: Check meta capability with object ID
if ( current_user_can( 'edit_post', $post_id ) ) {
    // Checks if user can edit THIS specific post
    // Considers post author, post status, etc.
    wp_update_post( array( 'ID' => $post_id, 'post_title' => $title ) );
}

// ✅ GOOD: Meta capability for user editing
if ( current_user_can( 'edit_user', $user_id ) ) {
    // Can this user edit THIS user?
    // Prevents privilege escalation
    wp_update_user( array( 'ID' => $user_id, 'display_name' => $name ) );
}
```

**How meta capabilities work:**
1. `current_user_can( 'edit_post', $post_id )` is called
2. WordPress calls `map_meta_cap( 'edit_post', $user_id, $post_id )`
3. `map_meta_cap()` returns primitive caps like `['edit_posts']` or `['edit_others_posts']`
4. WordPress checks if user has those primitive capabilities

## Common Capability Checks by Operation

### Content Management

```php
// Create posts
if ( current_user_can( 'create_posts' ) ) {
    wp_insert_post( $post_data );
}

// Edit ANY posts
if ( current_user_can( 'edit_posts' ) ) {
    // Can edit own posts
}

// Edit OTHERS' posts
if ( current_user_can( 'edit_others_posts' ) ) {
    // Can edit posts by other users
}

// Edit THIS specific post
if ( current_user_can( 'edit_post', $post_id ) ) {
    // Most secure: checks ownership, status, etc.
    wp_update_post( array( 'ID' => $post_id, 'post_title' => $title ) );
}

// Publish posts
if ( current_user_can( 'publish_posts' ) ) {
    wp_update_post( array( 'ID' => $post_id, 'post_status' => 'publish' ) );
}

// Delete posts
if ( current_user_can( 'delete_post', $post_id ) ) {
    wp_delete_post( $post_id, true );
}

// Read private posts
if ( current_user_can( 'read_private_posts' ) ) {
    // Can access private posts in queries
}
```

### User Management

```php
// List users
if ( current_user_can( 'list_users' ) ) {
    $users = get_users();
}

// Edit users (any)
if ( current_user_can( 'edit_users' ) ) {
    // Can edit some users
}

// Edit THIS specific user
if ( current_user_can( 'edit_user', $user_id ) ) {
    // Prevents privilege escalation
    // Contributors can't edit admins
    wp_update_user( array( 'ID' => $user_id, 'display_name' => $name ) );
}

// Delete users
if ( current_user_can( 'delete_user', $user_id ) ) {
    wp_delete_user( $user_id );
}

// Create users
if ( current_user_can( 'create_users' ) ) {
    wp_insert_user( $user_data );
}

// Promote users (change roles)
if ( current_user_can( 'promote_users' ) ) {
    $user = new WP_User( $user_id );
    $user->set_role( 'editor' );
}
```

### Plugin and Theme Management

```php
// Install plugins
if ( current_user_can( 'install_plugins' ) ) {
    // Can install new plugins
}

// Activate plugins
if ( current_user_can( 'activate_plugins' ) ) {
    activate_plugin( $plugin_path );
}

// Edit plugins (code editor)
if ( current_user_can( 'edit_plugins' ) ) {
    // Can edit plugin files directly
}

// Delete plugins
if ( current_user_can( 'delete_plugins' ) ) {
    delete_plugins( array( $plugin_path ) );
}

// Install themes
if ( current_user_can( 'install_themes' ) ) {
    // Can install new themes
}

// Switch themes
if ( current_user_can( 'switch_themes' ) ) {
    switch_theme( $stylesheet );
}

// Edit themes (code editor)
if ( current_user_can( 'edit_themes' ) ) {
    // Can edit theme files directly
}
```

### Site Management

```php
// Manage options
if ( current_user_can( 'manage_options' ) ) {
    // Can modify site settings
    update_option( 'blogname', $name );
}

// Manage categories/tags
if ( current_user_can( 'manage_categories' ) ) {
    wp_insert_term( $term_name, 'category' );
}

// Import content
if ( current_user_can( 'import' ) ) {
    // Can run importers
}

// Export content
if ( current_user_can( 'export' ) ) {
    // Can export site data
}

// Update core
if ( current_user_can( 'update_core' ) ) {
    // Can update WordPress core
}
```

### Media/Upload Management

```php
// Upload files
if ( current_user_can( 'upload_files' ) ) {
    $attachment_id = media_handle_upload( 'file', $post_id );
}

// Edit THIS attachment
if ( current_user_can( 'delete_post', $attachment_id ) ) {
    // Attachments are posts, use same caps
    wp_delete_attachment( $attachment_id, true );
}
```

## REST API Permission Callbacks

WordPress REST API requires `permission_callback` on all routes (enforced since WP 5.5).

### Missing permission_callback (CWE-862)

```php
// ❌ BAD: Missing permission_callback (WP 5.5+)
register_rest_route( 'myapp/v1', '/data', array(
    'methods'  => 'POST',
    'callback' => 'prefix_create_data',
    // No permission_callback! Anyone can POST
) );

// ❌ BAD: Intentionally missing (generates warning)
register_rest_route( 'myapp/v1', '/data', array(
    'methods'             => 'POST',
    'callback'            => 'prefix_create_data',
    'permission_callback' => null, // Still generates warning!
) );

// ✅ GOOD: Explicit permission check
register_rest_route( 'myapp/v1', '/data', array(
    'methods'             => 'POST',
    'callback'            => 'prefix_create_data',
    'permission_callback' => function() {
        return current_user_can( 'edit_posts' );
    },
) );

// ✅ GOOD: Public read-only endpoint
register_rest_route( 'myapp/v1', '/public-data', array(
    'methods'             => 'GET',
    'callback'            => 'prefix_get_public_data',
    'permission_callback' => '__return_true', // Explicitly public
) );
```

### Context-Aware Authorization

```php
// ❌ BAD: Same permission for all methods
register_rest_route( 'myapp/v1', '/posts/(?P<id>\d+)', array(
    array(
        'methods'             => WP_REST_Server::READABLE, // GET
        'callback'            => 'prefix_get_post',
        'permission_callback' => function() {
            return current_user_can( 'edit_posts' ); // Too restrictive for GET!
        },
    ),
    array(
        'methods'             => WP_REST_Server::EDITABLE, // POST, PUT, PATCH
        'callback'            => 'prefix_update_post',
        'permission_callback' => function() {
            return current_user_can( 'edit_posts' ); // Too permissive! Should check specific post
        },
    ),
) );

// ✅ GOOD: Context-aware permissions
register_rest_route( 'myapp/v1', '/posts/(?P<id>\d+)', array(
    array(
        'methods'             => WP_REST_Server::READABLE,
        'callback'            => 'prefix_get_post',
        'permission_callback' => function( $request ) {
            $post_id = absint( $request['id'] );
            $post = get_post( $post_id );

            // Allow if post is published (public)
            if ( $post && 'publish' === $post->post_status ) {
                return true;
            }

            // Otherwise require read_private_posts capability
            return current_user_can( 'read_private_posts' );
        },
    ),
    array(
        'methods'             => WP_REST_Server::EDITABLE,
        'callback'            => 'prefix_update_post',
        'permission_callback' => function( $request ) {
            $post_id = absint( $request['id'] );
            // Check if user can edit THIS specific post
            return current_user_can( 'edit_post', $post_id );
        },
    ),
    array(
        'methods'             => WP_REST_Server::DELETABLE,
        'callback'            => 'prefix_delete_post',
        'permission_callback' => function( $request ) {
            $post_id = absint( $request['id'] );
            // Check if user can delete THIS specific post
            return current_user_can( 'delete_post', $post_id );
        },
    ),
) );
```

### Multiple Method Endpoints

```php
// ✅ GOOD: Different permissions per method
register_rest_route( 'myapp/v1', '/settings', array(
    array(
        'methods'             => WP_REST_Server::READABLE,
        'callback'            => 'prefix_get_settings',
        'permission_callback' => function() {
            // Editors can view settings
            return current_user_can( 'edit_posts' );
        },
    ),
    array(
        'methods'             => WP_REST_Server::CREATABLE,
        'callback'            => 'prefix_update_settings',
        'permission_callback' => function() {
            // Only admins can modify settings
            return current_user_can( 'manage_options' );
        },
    ),
) );
```

### User-Specific Data Access

```php
// ❌ BAD: Returning any user's private data
function prefix_get_user_data( $request ) {
    $user_id = absint( $request['id'] );
    $user = get_userdata( $user_id );

    return array(
        'email'    => $user->user_email, // Private data!
        'password' => $user->user_pass,  // Never return this!
    );
}

register_rest_route( 'myapp/v1', '/users/(?P<id>\d+)', array(
    'methods'             => WP_REST_Server::READABLE,
    'callback'            => 'prefix_get_user_data',
    'permission_callback' => '__return_true', // Anyone can access!
) );

// ✅ GOOD: Check if user can edit target user
function prefix_get_user_data( $request ) {
    $user_id = absint( $request['id'] );

    // Return only public data
    $user = get_userdata( $user_id );

    return array(
        'id'           => $user->ID,
        'display_name' => $user->display_name,
        'avatar'       => get_avatar_url( $user_id ),
    );
}

register_rest_route( 'myapp/v1', '/users/(?P<id>\d+)', array(
    'methods'             => WP_REST_Server::READABLE,
    'callback'            => 'prefix_get_user_data',
    'permission_callback' => function( $request ) {
        $user_id = absint( $request['id'] );
        $current_user_id = get_current_user_id();

        // Allow user to access own data, or admins to access any
        return $current_user_id === $user_id || current_user_can( 'edit_user', $user_id );
    },
) );

// ✅ GOOD: Separate private endpoint
function prefix_get_private_user_data( $request ) {
    $user_id = absint( $request['id'] );
    $user = get_userdata( $user_id );

    return array(
        'email' => $user->user_email,
        'phone' => get_user_meta( $user_id, 'phone', true ),
    );
}

register_rest_route( 'myapp/v1', '/users/(?P<id>\d+)/private', array(
    'methods'             => WP_REST_Server::READABLE,
    'callback'            => 'prefix_get_private_user_data',
    'permission_callback' => function( $request ) {
        $user_id = absint( $request['id'] );
        $current_user_id = get_current_user_id();

        // Only allow user to access own private data
        if ( $current_user_id === $user_id ) {
            return true;
        }

        // Or admins with edit_user capability
        return current_user_can( 'edit_user', $user_id );
    },
) );
```

### Returning WP_Error for Better Error Messages

```php
// ✅ GOOD: Return WP_Error with specific message
register_rest_route( 'myapp/v1', '/premium-content', array(
    'methods'             => 'GET',
    'callback'            => 'prefix_get_premium_content',
    'permission_callback' => function() {
        if ( ! is_user_logged_in() ) {
            return new WP_Error(
                'rest_not_logged_in',
                'You must be logged in to access premium content.',
                array( 'status' => 401 )
            );
        }

        $user_id = get_current_user_id();
        $has_subscription = get_user_meta( $user_id, 'active_subscription', true );

        if ( ! $has_subscription ) {
            return new WP_Error(
                'rest_no_subscription',
                'Active subscription required.',
                array( 'status' => 403 )
            );
        }

        return true;
    },
) );
```

## AJAX Handler Authorization

AJAX requests need the same authorization checks as form submissions.

### wp_ajax_ vs wp_ajax_nopriv_

```php
// ✅ GOOD: Logged-in users only
function prefix_delete_post_ajax() {
    // Check nonce
    check_ajax_referer( 'delete_post_nonce', 'nonce' );

    // Check capability
    $post_id = absint( $_POST['post_id'] );
    if ( ! current_user_can( 'delete_post', $post_id ) ) {
        wp_send_json_error( array( 'message' => 'Permission denied' ) );
    }

    // Perform action
    wp_delete_post( $post_id, true );
    wp_send_json_success();
}
add_action( 'wp_ajax_delete_post', 'prefix_delete_post_ajax' );
// No wp_ajax_nopriv_ hook = logged-out users can't access

// ✅ GOOD: Public AJAX (logged in OR logged out)
function prefix_get_public_data_ajax() {
    // No auth needed, but still verify nonce if state-changing
    $data = get_option( 'prefix_public_data' );
    wp_send_json_success( $data );
}
add_action( 'wp_ajax_get_public_data', 'prefix_get_public_data_ajax' );
add_action( 'wp_ajax_nopriv_get_public_data', 'prefix_get_public_data_ajax' );
```

### Admin AJAX Capability Patterns

```php
// ❌ BAD: No capability check in AJAX handler
function prefix_update_settings_ajax() {
    check_ajax_referer( 'update_settings_nonce', 'nonce' );

    // Anyone logged in can update settings!
    update_option( 'prefix_setting', $_POST['value'] );
    wp_send_json_success();
}
add_action( 'wp_ajax_update_settings', 'prefix_update_settings_ajax' );

// ✅ GOOD: Capability check before action
function prefix_update_settings_ajax() {
    check_ajax_referer( 'update_settings_nonce', 'nonce' );

    if ( ! current_user_can( 'manage_options' ) ) {
        wp_send_json_error( array( 'message' => 'Insufficient permissions' ) );
    }

    $value = sanitize_text_field( wp_unslash( $_POST['value'] ) );
    update_option( 'prefix_setting', $value );
    wp_send_json_success();
}
add_action( 'wp_ajax_update_settings', 'prefix_update_settings_ajax' );
```

### Object-Specific Capabilities in AJAX

```php
// ❌ BAD: Generic capability check
function prefix_edit_post_ajax() {
    check_ajax_referer( 'edit_post_nonce', 'nonce' );

    if ( ! current_user_can( 'edit_posts' ) ) { // Can edit ANY post?
        wp_send_json_error( 'No permission' );
    }

    $post_id = absint( $_POST['post_id'] );
    // User can edit ANY post, not necessarily THIS one
    wp_update_post( array( 'ID' => $post_id, 'post_title' => $_POST['title'] ) );
}

// ✅ GOOD: Object-specific capability check
function prefix_edit_post_ajax() {
    check_ajax_referer( 'edit_post_nonce', 'nonce' );

    $post_id = absint( $_POST['post_id'] );

    if ( ! current_user_can( 'edit_post', $post_id ) ) { // Can edit THIS post?
        wp_send_json_error( array( 'message' => 'Permission denied' ) );
    }

    $title = sanitize_text_field( wp_unslash( $_POST['title'] ) );
    wp_update_post( array( 'ID' => $post_id, 'post_title' => $title ) );
    wp_send_json_success();
}
add_action( 'wp_ajax_edit_post', 'prefix_edit_post_ajax' );
```

## Custom Capabilities

For custom post types and scenarios, register custom capabilities.

### Custom Post Type Capabilities

```php
// ✅ GOOD: Custom post type with custom capabilities
function prefix_register_custom_post_type() {
    register_post_type( 'product', array(
        'public'       => true,
        'label'        => 'Products',
        'capability_type' => 'product',
        'capabilities' => array(
            'edit_post'          => 'edit_product',
            'read_post'          => 'read_product',
            'delete_post'        => 'delete_product',
            'edit_posts'         => 'edit_products',
            'edit_others_posts'  => 'edit_others_products',
            'delete_posts'       => 'delete_products',
            'publish_posts'      => 'publish_products',
            'read_private_posts' => 'read_private_products',
        ),
        'map_meta_cap' => true,
    ) );
}
add_action( 'init', 'prefix_register_custom_post_type' );

// Grant capabilities to role
function prefix_add_product_caps() {
    $role = get_role( 'shop_manager' );
    if ( $role ) {
        $role->add_cap( 'edit_products' );
        $role->add_cap( 'edit_others_products' );
        $role->add_cap( 'publish_products' );
        $role->add_cap( 'read_private_products' );
        $role->add_cap( 'delete_products' );
    }
}
register_activation_hook( __FILE__, 'prefix_add_product_caps' );

// Check custom capability
if ( current_user_can( 'edit_product', $product_id ) ) {
    wp_update_post( array( 'ID' => $product_id, 'post_title' => $title ) );
}
```

### Map Meta Cap Filter

```php
// ✅ GOOD: Custom meta capability mapping
function prefix_map_meta_cap( $caps, $cap, $user_id, $args ) {
    // Custom capability: manage_team_settings
    if ( 'manage_team_settings' === $cap ) {
        $team_id = isset( $args[0] ) ? $args[0] : 0;

        // Get team
        $team = get_post( $team_id );
        if ( ! $team || 'team' !== $team->post_type ) {
            // Require impossible capability if team doesn't exist
            return array( 'do_not_allow' );
        }

        // Team owner can manage
        if ( absint( $team->post_author ) === absint( $user_id ) ) {
            return array( 'edit_posts' );
        }

        // Team admins can manage
        $team_admins = get_post_meta( $team_id, '_team_admins', true );
        if ( is_array( $team_admins ) && in_array( $user_id, $team_admins, true ) ) {
            return array( 'edit_posts' );
        }

        // Otherwise require manage_options
        return array( 'manage_options' );
    }

    return $caps;
}
add_filter( 'map_meta_cap', 'prefix_map_meta_cap', 10, 4 );

// Check custom meta capability
if ( current_user_can( 'manage_team_settings', $team_id ) ) {
    update_post_meta( $team_id, '_team_settings', $settings );
}
```

## Common Authorization Mistakes

### 1. Checking Primitive Capability Instead of Meta Capability

```php
// ❌ BAD: Primitive capability without object context
if ( current_user_can( 'edit_posts' ) ) {
    wp_update_post( array( 'ID' => $post_id, 'post_title' => $title ) );
}

// ✅ GOOD: Meta capability with object ID
if ( current_user_can( 'edit_post', $post_id ) ) {
    wp_update_post( array( 'ID' => $post_id, 'post_title' => $title ) );
}
```

### 2. Using is_admin() for Authorization

```php
// ❌ BAD: is_admin() only checks admin dashboard, not permissions
if ( is_admin() ) {
    // Anyone can access if they're in wp-admin!
    delete_user( $user_id );
}

// ✅ GOOD: Check capability
if ( current_user_can( 'delete_user', $user_id ) ) {
    delete_user( $user_id );
}
```

### 3. Forgetting REST API permission_callback

```php
// ❌ BAD: No permission_callback
register_rest_route( 'myapp/v1', '/data', array(
    'methods'  => 'POST',
    'callback' => 'handler',
) );

// ✅ GOOD: Always include permission_callback
register_rest_route( 'myapp/v1', '/data', array(
    'methods'             => 'POST',
    'callback'            => 'handler',
    'permission_callback' => function() {
        return current_user_can( 'edit_posts' );
    },
) );
```

### 4. Not Checking Authorization in AJAX Handlers

```php
// ❌ BAD: Only nonce check, no capability
function prefix_delete_ajax() {
    check_ajax_referer( 'delete_nonce', 'nonce' );
    wp_delete_post( $_POST['id'] ); // Any logged-in user can delete!
}

// ✅ GOOD: Nonce AND capability check
function prefix_delete_ajax() {
    check_ajax_referer( 'delete_nonce', 'nonce' );
    $post_id = absint( $_POST['id'] );

    if ( ! current_user_can( 'delete_post', $post_id ) ) {
        wp_send_json_error( 'Permission denied' );
    }

    wp_delete_post( $post_id );
    wp_send_json_success();
}
```

### 5. Checking Authorization After Action

```php
// ❌ BAD: Authorization check after performing action
function prefix_update_post_handler() {
    // Action performed first
    wp_update_post( array( 'ID' => $post_id, 'post_title' => $title ) );

    // Check too late!
    if ( ! current_user_can( 'edit_post', $post_id ) ) {
        wp_die( 'No permission' );
    }
}

// ✅ GOOD: Check authorization BEFORE action
function prefix_update_post_handler() {
    $post_id = absint( $_POST['post_id'] );

    // Check first
    if ( ! current_user_can( 'edit_post', $post_id ) ) {
        wp_die( 'No permission' );
    }

    // Then perform action
    $title = sanitize_text_field( wp_unslash( $_POST['title'] ) );
    wp_update_post( array( 'ID' => $post_id, 'post_title' => $title ) );
}
```

### 6. Trusting Client-Side Authorization

```php
// ❌ BAD: Only hiding UI, no server-side check
// JavaScript:
// if (userData.role === 'admin') {
//     showDeleteButton();
// }

function prefix_delete_handler() {
    // No server-side check! Anyone who knows the endpoint can delete
    wp_delete_post( $_POST['id'] );
}

// ✅ GOOD: Always check on server
function prefix_delete_handler() {
    $post_id = absint( $_POST['id'] );

    if ( ! current_user_can( 'delete_post', $post_id ) ) {
        wp_send_json_error( 'Permission denied' );
    }

    wp_delete_post( $post_id );
    wp_send_json_success();
}
```

## Summary

Key principles for WordPress authorization:

1. **Check capabilities, not roles** — Roles are containers of capabilities
2. **Use meta capabilities** — `current_user_can( 'edit_post', $post_id )` not `current_user_can( 'edit_posts' )`
3. **Always check authorization** — In form handlers, AJAX, REST API, shortcodes
4. **Check before action** — Authorization first, action second
5. **REST API requires permission_callback** — Enforced since WP 5.5
6. **Context-aware permissions** — Public GET, authenticated POST
7. **Don't trust client-side** — Always verify on server

Cross-references:
- CSRF protection → nonce-csrf-guide.md
- Authorization vulnerabilities → vulnerability-patterns.md
- Complete security review → SKILL.md
