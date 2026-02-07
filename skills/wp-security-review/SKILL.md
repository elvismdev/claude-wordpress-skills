---
name: wp-security-review
description: WordPress security code review and vulnerability detection. Use when reviewing WordPress PHP code for security issues, auditing themes/plugins for vulnerabilities, checking code before launch, analyzing AJAX/REST handlers for exploits, detecting XSS, SQL injection, CSRF, or authorization bypass, or when user mentions "security review", "vulnerability", "XSS", "SQL injection", "CSRF", "nonce", "sanitization", "escaping", "capability check", "privilege escalation", "file upload security", "insecure code", "auth bypass", or "security audit". Detects vulnerabilities in input handling, output escaping, authorization, nonces, database queries, file uploads, and dangerous functions.
---

# WordPress Security Review Skill

## Overview

Systematic security code review for WordPress themes, plugins, and custom code. **Core principle:** Three-pillar security model: (1) Sanitize input early, (2) Validate authorization (nonces + capabilities), (3) Escape output late. Scan critical issues first (SQL injection in public code, XSS on unescaped output, missing nonces on state-changing operations), then warnings, then info-level improvements. Report with line numbers, severity, CWE references, and BAD/GOOD code pairs.

## When to Use

**Use when:**
- Security audit of WordPress theme or plugin
- Pre-launch security review
- Investigating reported vulnerability
- Reviewing form handlers, AJAX endpoints, or REST API routes
- Analyzing user input processing or output rendering
- Code review for exploitable patterns (XSS, SQL injection, CSRF, auth bypass)

**Don't use for:**
- Server hardening or infrastructure security (Apache/nginx config, firewall rules)
- Web Application Firewall (WAF) configuration
- SSL/TLS certificate management
- Performance-only audits (use wp-performance-review)
- General non-WordPress PHP security

## Code Review Workflow

1. **Identify file type** and apply relevant checks below
2. **Scan for CRITICAL patterns first** (SQL injection in public code, XSS on unescaped output, missing nonces on state-changing operations, file upload without validation)
3. **Check WARNING patterns** (authorization issues, admin-only XSS, incomplete validation)
4. **Note INFO improvements** (defense-in-depth, security constants, hardening opportunities)
5. **Apply context-aware severity adjustment** (admin-only code = lower severity, public-facing code = highest severity, WP-CLI/cron = nonces not needed, REST API = different auth model)
6. **Report with line numbers** using output format below

## File-Type Specific Checks

### Plugin/Theme PHP Files (`functions.php`, `plugin.php`, `*.php`)

Scan for:
- Missing `defined( 'ABSPATH' ) || exit;` at top → WARNING: Direct file access possible
- `eval(` → CRITICAL: Code injection vector (CWE-95)
- `exec(`, `shell_exec(`, `system(`, `passthru(` → CRITICAL: Command injection if user input present (CWE-78)
- `base64_decode( $_` → CRITICAL: Possible encoded payload execution
- `unserialize( $_` → CRITICAL: Object injection (CWE-502)
- `$_GET[`, `$_POST[`, `$_REQUEST[` without `sanitize_*` → WARNING: Unsanitized input (CWE-20)
- `include $_`, `require $_`, `include_once $_`, `require_once $_` → CRITICAL: Path traversal/inclusion (CWE-22)
- Raw `move_uploaded_file(` → CRITICAL: Use `wp_handle_upload()` instead

### Form Handlers (`admin-post.php`, `admin_post_*` hooks)

Scan for three-step pattern (ALL must be present):
1. **Nonce verification** → `wp_verify_nonce( $_POST['nonce_field'], 'action_name' )` or `check_admin_referer()` → CRITICAL if missing (CWE-352)
2. **Capability check** → `current_user_can( 'capability' )` → CRITICAL if missing (CWE-862)
3. **Input sanitization** → `sanitize_text_field()`, `sanitize_email()`, etc. → WARNING if missing (CWE-20)

Missing ANY step = vulnerability. State-changing operations MUST have all three.

### AJAX Handlers (`wp_ajax_*`, `wp_ajax_nopriv_*` hooks)

Scan for:
- `check_ajax_referer( 'action_name', 'nonce_field' )` → CRITICAL if missing on state-changing operations (CWE-352)
- `current_user_can( 'capability' )` → CRITICAL if missing when capability required (CWE-862)
- `wp_ajax_nopriv_*` without nonce → CRITICAL: Public endpoint with no CSRF protection
- Input from `$_POST` without `sanitize_*` → WARNING: Unsanitized input (CWE-20)
- `wp_send_json()` without escaping when echoing user input → WARNING: JSON injection

### REST API Endpoints (`register_rest_route`)

Scan for:
- Missing `permission_callback` → CRITICAL: WordPress 5.5+ requires this (CWE-862)
- `'permission_callback' => '__return_true'` on write operations → CRITICAL: Allows unauthorized access
- `'permission_callback' => '__return_true'` on public read endpoints → OK (not a vulnerability)
- Missing `current_user_can()` in permission callback → WARNING: May allow privilege escalation (CWE-863)
- Input validation missing on `$request->get_param()` → WARNING: Unsanitized input (CWE-20)
- REST authentication via `X-WP-Nonce` header or application passwords → INFO: Verify client sends header

### Database Code (`$wpdb->query`, `$wpdb->get_results`, etc.)

Scan for:
- `$wpdb->query(` or `$wpdb->get_results(` with string concatenation or interpolation → CRITICAL: SQL injection (CWE-89)
- Missing `$wpdb->prepare()` when user input present → CRITICAL: SQL injection (CWE-89)
- `$wpdb->prepare( "... LIKE '%{$term}%'" )` → WARNING: Should use `$wpdb->esc_like()` first
- Double-preparing (preparing already-prepared strings) → WARNING: Escapes escape characters
- Direct use of `{$wpdb->prefix}` in queries → OK (not a vulnerability, standard pattern)

Safe pattern: `$wpdb->get_results( $wpdb->prepare( "SELECT * FROM {$wpdb->posts} WHERE ID = %d", $post_id ) )`

### Template/Output Files (`*.php` in theme, templates)

Scan for:
- `echo $` or `print $` without escaping function → CRITICAL: XSS (CWE-79)
- `<input value="<?php echo $` → CRITICAL: Use `esc_attr()` (CWE-79)
- `<a href="<?php echo $` → CRITICAL: Use `esc_url()` (CWE-79)
- `<script>` blocks with PHP variables → CRITICAL: Use `esc_js()` or `wp_localize_script()` (CWE-79)
- Rich HTML output without `wp_kses()` or `wp_kses_post()` → WARNING: May allow unsafe tags

Safe pattern: Always escape at point of output, not at input or storage.

### File Upload Handlers

Scan for:
- `move_uploaded_file(` instead of `wp_handle_upload()` → CRITICAL: Bypasses WP security checks (CWE-434)
- Missing `current_user_can()` check → CRITICAL: Unauthorized file upload (CWE-862)
- No MIME type validation → WARNING: May allow executable uploads
- No file size limit → INFO: Could enable DoS
- Missing `wp_check_filetype_and_ext()` → WARNING: File type spoofing possible (CWE-434)

### JavaScript Files (`*.js`, `*.jsx`)

Scan for:
- `fetch()` or `$.ajax()` to REST API without `X-WP-Nonce` header → WARNING: CSRF vulnerability (CWE-352)
- `$.post( ajaxurl, ...)` without nonce in data → WARNING: CSRF vulnerability (CWE-352)
- `innerHTML = userInput` or `dangerouslySetInnerHTML` → CRITICAL: DOM-based XSS (CWE-79)
- `eval(` → CRITICAL: Code injection vector (CWE-95)

## Search Patterns for Quick Detection (SEC-20)

```bash
# CRITICAL: SQL injection patterns
grep -rn "\$wpdb->query(" . | grep -v "prepare"
grep -rn "\$wpdb->get_results(" . | grep -v "prepare"
grep -rn "\$wpdb->get_var(" . | grep -v "prepare"
grep -rn "\$wpdb->get_row(" . | grep -v "prepare"

# CRITICAL: XSS (unescaped output)
grep -rn "echo \$_" .
grep -rn "print \$_" .
grep -rn "<?php echo \$" . | grep -v "esc_"

# CRITICAL: Dangerous functions
grep -rn "eval(" .
grep -rn "exec(" .
grep -rn "shell_exec(" .
grep -rn "system(" .
grep -rn "passthru(" .
grep -rn "base64_decode(\$_" .
grep -rn "unserialize(\$_" .

# CRITICAL: File upload without WP functions
grep -rn "move_uploaded_file(" .

# CRITICAL: REST API without permission_callback
grep -rn "register_rest_route" . | grep -v "permission_callback"

# CRITICAL: Path traversal / dynamic includes
grep -rn "include \$_" .
grep -rn "require \$_" .
grep -rn "include_once \$_" .
grep -rn "require_once \$_" .

# WARNING: Unsanitized input usage
grep -rn "\$_GET\[" . | grep -v "sanitize_"
grep -rn "\$_POST\[" . | grep -v "sanitize_"
grep -rn "\$_REQUEST\[" . | grep -v "sanitize_"

# WARNING: Missing nonces on public AJAX
grep -rn "wp_ajax_nopriv_" .

# WARNING: Missing capability checks
grep -rn "admin_post_" . | grep -v "current_user_can"
grep -rn "wp_ajax_" . | grep -v "current_user_can"

# INFO: Missing ABSPATH check
grep -rn "<?php" . | head -20 | grep -v "ABSPATH"

# INFO: Security constants missing (check wp-config.php)
grep -n "DISALLOW_FILE_EDIT" wp-config.php
grep -n "FORCE_SSL_ADMIN" wp-config.php
grep -n "DISALLOW_UNFILTERED_HTML" wp-config.php
```

## Platform Context

Different hosting environments provide varying security layers:

**Managed WordPress Hosts** (WP Engine, Pantheon, Pressable, WordPress VIP, etc.):
- Often have platform-level WAF and malware scanning
- May block certain dangerous functions (eval, exec) at platform level
- Check host documentation for additional security requirements
- Code should still be written securely for portability

**WordPress VIP**:
- Strict code review requirements
- Many dangerous functions prohibited
- Additional security helpers available (`wpcom_vip_*` functions)
- See VIP documentation for required patterns

**Self-Hosted / Standard Hosting**:
- No platform-level security beyond what code implements
- More responsibility for secure coding practices
- Ensure all three pillars (sanitize, authorize, escape) are implemented

## Quick Reference: Critical Security Patterns

### XSS Prevention (Output Escaping)

```php
// ❌ CRITICAL: Unescaped output (CWE-79)
<h1><?php echo $_GET['title']; ?></h1>
<input value="<?php echo $user_input; ?>">
<a href="<?php echo $url; ?>">Link</a>
<script>var data = "<?php echo $json; ?>";</script>

// ✅ GOOD: Context-appropriate escaping
<h1><?php echo esc_html( $_GET['title'] ); ?></h1>
<input value="<?php echo esc_attr( $user_input ); ?>">
<a href="<?php echo esc_url( $url ); ?>">Link</a>
<script>var data = <?php echo wp_json_encode( $data ); ?>;</script>

// ❌ CRITICAL: Early escaping (wrong pattern)
$title = esc_html( $_POST['title'] );
update_post_meta( $post_id, 'title', $title ); // Stores HTML entities

// ✅ GOOD: Late escaping (correct pattern)
$title = sanitize_text_field( $_POST['title'] );
update_post_meta( $post_id, 'title', $title );
echo '<h1>' . esc_html( get_post_meta( $post_id, 'title', true ) ) . '</h1>';

// ✅ GOOD: Rich HTML with wp_kses_post() for trusted content
echo wp_kses_post( $content ); // Allows safe HTML tags only

// ✅ GOOD: Custom allowed tags with wp_kses()
$allowed_tags = array(
    'a' => array( 'href' => array(), 'title' => array() ),
    'br' => array(),
    'strong' => array(),
);
echo wp_kses( $user_content, $allowed_tags );
```

### SQL Injection Prevention

```php
// ❌ CRITICAL: Direct interpolation (CWE-89)
$results = $wpdb->get_results( "SELECT * FROM {$wpdb->users} WHERE ID = $user_id" );
$results = $wpdb->query( "UPDATE {$wpdb->posts} SET post_title = '$title'" );

// ✅ GOOD: Use $wpdb->prepare() with placeholders
$results = $wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM {$wpdb->users} WHERE ID = %d",
    $user_id
) );

$wpdb->query( $wpdb->prepare(
    "UPDATE {$wpdb->posts} SET post_title = %s WHERE ID = %d",
    $title,
    $post_id
) );

// ✅ GOOD: Multiple placeholders
$wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM {$wpdb->posts} WHERE post_status = %s AND post_author = %d",
    $status,
    $author_id
) );

// ❌ WARNING: LIKE without esc_like()
$wpdb->prepare( "SELECT * FROM {$wpdb->posts} WHERE post_title LIKE '%%{$term}%%'" );

// ✅ GOOD: Use esc_like() for LIKE queries
$like_term = '%' . $wpdb->esc_like( $term ) . '%';
$wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM {$wpdb->posts} WHERE post_title LIKE %s",
    $like_term
) );

// ✅ OK: Hardcoded SQL without user input (admin context)
if ( current_user_can( 'manage_options' ) ) {
    $wpdb->query( "DELETE FROM {$wpdb->options} WHERE option_name = 'temp_setting'" );
}
```

### CSRF/Nonce Verification (Three-Step Pattern)

```php
// ❌ CRITICAL: Missing nonce verification (CWE-352)
add_action( 'admin_post_save_settings', function() {
    update_option( 'my_setting', $_POST['value'] );
} );

// ❌ CRITICAL: Missing capability check (CWE-862)
add_action( 'admin_post_save_settings', function() {
    check_admin_referer( 'save_settings_action', 'settings_nonce' );
    update_option( 'my_setting', $_POST['value'] );
} );

// ✅ GOOD: Complete three-step pattern
add_action( 'admin_post_save_settings', function() {
    // Step 1: Verify nonce
    if ( ! isset( $_POST['settings_nonce'] ) ||
         ! wp_verify_nonce( $_POST['settings_nonce'], 'save_settings_action' ) ) {
        wp_die( 'Invalid nonce' );
    }

    // Step 2: Check capability
    if ( ! current_user_can( 'manage_options' ) ) {
        wp_die( 'Insufficient permissions' );
    }

    // Step 3: Sanitize input
    $value = sanitize_text_field( $_POST['value'] );
    update_option( 'my_setting', $value );

    wp_redirect( admin_url( 'admin.php?page=settings&updated=true' ) );
    exit;
} );

// ✅ GOOD: Form with nonce field
<form method="post" action="<?php echo esc_url( admin_url( 'admin-post.php' ) ); ?>">
    <?php wp_nonce_field( 'save_settings_action', 'settings_nonce' ); ?>
    <input type="hidden" name="action" value="save_settings">
    <input type="text" name="value" value="<?php echo esc_attr( $current_value ); ?>">
    <?php submit_button(); ?>
</form>

// ✅ GOOD: AJAX with nonce
add_action( 'wp_ajax_update_user_meta', function() {
    check_ajax_referer( 'update_meta_nonce', 'nonce' );

    if ( ! current_user_can( 'edit_users' ) ) {
        wp_send_json_error( 'Insufficient permissions' );
    }

    $user_id = absint( $_POST['user_id'] );
    $value   = sanitize_text_field( $_POST['value'] );

    update_user_meta( $user_id, 'custom_field', $value );
    wp_send_json_success();
} );

// JavaScript for AJAX nonce
jQuery.post( ajaxurl, {
    action: 'update_user_meta',
    nonce: myAjax.nonce, // From wp_localize_script()
    user_id: 123,
    value: 'new value'
} );
```

### Authorization/Capability Checks

```php
// ❌ CRITICAL: No capability check (CWE-862)
add_action( 'admin_post_delete_user', function() {
    wp_delete_user( $_POST['user_id'] );
} );

// ❌ WARNING: Incorrect capability (CWE-863)
add_action( 'admin_post_delete_user', function() {
    if ( ! current_user_can( 'edit_posts' ) ) { // Wrong capability!
        wp_die( 'Insufficient permissions' );
    }
    wp_delete_user( $_POST['user_id'] );
} );

// ✅ GOOD: Correct capability check
add_action( 'admin_post_delete_user', function() {
    if ( ! current_user_can( 'delete_users' ) ) {
        wp_die( 'Insufficient permissions' );
    }

    check_admin_referer( 'delete_user_action', 'delete_nonce' );
    wp_delete_user( absint( $_POST['user_id'] ) );
} );

// ✅ GOOD: Check user can edit specific post
if ( ! current_user_can( 'edit_post', $post_id ) ) {
    wp_die( 'You cannot edit this post' );
}

// ✅ GOOD: REST API permission callback
register_rest_route( 'myapp/v1', '/users/(?P<id>\d+)', array(
    'methods'             => 'DELETE',
    'callback'            => 'myapp_delete_user',
    'permission_callback' => function( $request ) {
        return current_user_can( 'delete_users' );
    },
) );

// ✅ OK: __return_true for public read endpoint
register_rest_route( 'myapp/v1', '/posts', array(
    'methods'             => 'GET',
    'callback'            => 'myapp_get_posts',
    'permission_callback' => '__return_true', // Public read is OK
) );
```

### File Upload Security

```php
// ❌ CRITICAL: Direct move_uploaded_file() (CWE-434)
if ( isset( $_FILES['upload'] ) ) {
    move_uploaded_file( $_FILES['upload']['tmp_name'], '/uploads/' . $_FILES['upload']['name'] );
}

// ❌ CRITICAL: No capability check (CWE-862)
$file = wp_handle_upload( $_FILES['upload'], array( 'test_form' => false ) );

// ✅ GOOD: Complete file upload security
add_action( 'admin_post_upload_file', function() {
    // Step 1: Verify nonce
    check_admin_referer( 'upload_file_action', 'upload_nonce' );

    // Step 2: Check capability
    if ( ! current_user_can( 'upload_files' ) ) {
        wp_die( 'Insufficient permissions' );
    }

    // Step 3: Validate file exists
    if ( ! isset( $_FILES['upload'] ) || UPLOAD_ERR_OK !== $_FILES['upload']['error'] ) {
        wp_die( 'File upload failed' );
    }

    // Step 4: Use wp_handle_upload() (validates MIME type, extension)
    require_once( ABSPATH . 'wp-admin/includes/file.php' );
    $file = wp_handle_upload( $_FILES['upload'], array( 'test_form' => false ) );

    if ( isset( $file['error'] ) ) {
        wp_die( 'Upload error: ' . esc_html( $file['error'] ) );
    }

    // Step 5: Store file info
    $attachment_id = wp_insert_attachment( array(
        'post_title'     => sanitize_file_name( $_FILES['upload']['name'] ),
        'post_mime_type' => $file['type'],
        'post_status'    => 'inherit',
    ), $file['file'] );
} );

// ✅ GOOD: Restrict allowed MIME types
add_filter( 'upload_mimes', function( $mimes ) {
    // Remove potentially dangerous types
    unset( $mimes['exe'] );
    unset( $mimes['php'] );

    // Add custom allowed types if needed
    $mimes['svg'] = 'image/svg+xml';
    return $mimes;
} );

// ✅ GOOD: File type validation
$filetype = wp_check_filetype_and_ext( $_FILES['upload']['tmp_name'], $_FILES['upload']['name'] );
if ( ! $filetype['ext'] ) {
    wp_die( 'Invalid file type' );
}
```

### Object Injection Prevention

```php
// ❌ CRITICAL: Unserialize user input (CWE-502)
$data = unserialize( $_POST['data'] );

// ❌ CRITICAL: Unserialize from cookie
$data = unserialize( $_COOKIE['cart_data'] );

// ✅ GOOD: Use JSON instead of serialize for user-facing data
$data = json_decode( $_POST['data'], true );
if ( JSON_ERROR_NONE !== json_last_error() ) {
    wp_die( 'Invalid JSON' );
}

// ✅ OK: Unserialize trusted internal data only
$data = get_option( 'my_internal_setting' ); // Already unserialized by WP
$meta = get_post_meta( $post_id, 'my_field', true ); // Already unserialized by WP
```

### Dangerous Functions

```php
// ❌ CRITICAL: eval() with user input (CWE-95)
eval( $_POST['code'] );

// ❌ CRITICAL: exec() with user input (CWE-78)
exec( 'ls -la ' . $_GET['dir'] );

// ❌ CRITICAL: shell_exec() with user input
$output = shell_exec( 'grep ' . $_POST['search'] . ' file.txt' );

// ✅ GOOD: Avoid eval(), exec(), shell_exec() entirely in web context
// If absolutely necessary, whitelist input strictly
$allowed_dirs = array( 'uploads', 'cache' );
$dir = sanitize_key( $_GET['dir'] );
if ( in_array( $dir, $allowed_dirs, true ) ) {
    exec( 'ls -la ' . escapeshellarg( $dir ), $output );
}
```

### Direct File Access Prevention

```php
// ❌ WARNING: Missing ABSPATH check
<?php
// Plugin code here without security check

// ✅ GOOD: ABSPATH check at top of every PHP file
<?php
defined( 'ABSPATH' ) || exit;

// Plugin code here

// ✅ GOOD: Alternative pattern
if ( ! defined( 'ABSPATH' ) ) {
    die( 'Direct access not permitted.' );
}
```

### Input Sanitization

```php
// ❌ WARNING: No sanitization (CWE-20)
$title = $_POST['title'];
update_post_meta( $post_id, 'title', $title );

// ✅ GOOD: Sanitize based on expected data type
$title       = sanitize_text_field( $_POST['title'] );
$email       = sanitize_email( $_POST['email'] );
$url         = sanitize_url( $_POST['url'] );
$filename    = sanitize_file_name( $_POST['filename'] );
$key         = sanitize_key( $_POST['key'] );
$int         = absint( $_POST['count'] );
$float       = floatval( $_POST['price'] );
$textarea    = sanitize_textarea_field( $_POST['description'] );

// ✅ GOOD: Array of integers
$ids = array_map( 'absint', (array) $_POST['ids'] );

// ✅ GOOD: Validate against whitelist
$allowed_types = array( 'post', 'page', 'product' );
$type = sanitize_key( $_POST['type'] );
if ( ! in_array( $type, $allowed_types, true ) ) {
    wp_die( 'Invalid type' );
}
```

## Severity Definitions (SEC-22)

| Severity | Description |
|----------|-------------|
| **CRITICAL** | Exploitable without authentication OR leads to data breach, RCE, or privilege escalation. Examples: SQL injection on public endpoint, XSS on unescaped output, missing nonce on admin action, file upload without capability check, eval() with user input |
| **WARNING** | Exploitable with authentication OR requires specific conditions. Examples: XSS in admin-only code, missing capability check on logged-in action, SQL injection in admin context, incomplete input sanitization |
| **INFO** | Defense-in-depth improvement or hardening opportunity. Examples: missing ABSPATH check, security constants not set in wp-config.php, missing security headers |

**Context adjustments:**
- **Admin-only code**: Lower severity by one level (CRITICAL → WARNING)
- **WP-CLI/cron context**: Nonces not required (no CSRF risk)
- **REST API**: Different auth model (permission_callback, not nonces)
- **Public-facing code**: Highest severity (no authentication barrier)

## Output Format (SEC-23)

Group findings by FILE and severity level:

```markdown
## Security Review: [filename]

### CRITICAL Issues
**Line X** | CWE-79 | XSS via unescaped output
- **Issue**: User input echoed directly without escaping
- **Risk**: Allows attackers to inject malicious JavaScript
- **Context**: Public-facing template (HIGH severity)

❌ **BAD:**
```php
<h1><?php echo $_GET['title']; ?></h1>
```

✅ **GOOD:**
```php
<h1><?php echo esc_html( $_GET['title'] ); ?></h1>
```

### WARNING Issues
**Line Y** | CWE-352 | Missing nonce verification
- **Issue**: Form handler missing wp_verify_nonce()
- **Risk**: State can be changed via CSRF attack
- **Context**: Admin-only (reduced severity)

[BAD/GOOD code pair]

### INFO Recommendations
**Line Z** | Defense-in-depth | Missing ABSPATH check
- Add `defined( 'ABSPATH' ) || exit;` at top of file

### Summary
- **Total**: 1 CRITICAL, 1 WARNING, 1 INFO
- **Risk level**: HIGH (exploitable CRITICAL issue present)
- **Immediate action**: Fix line X (unescaped output)
```

## Common Mistakes (SEC-24)

When performing security reviews, avoid these false positives:

| Mistake | Why It's Wrong | Context |
|---------|----------------|---------|
| Flagging missing nonces in WP-CLI commands | WP-CLI runs server-side, no CSRF risk | Nonces only needed for HTTP requests |
| Flagging `wp_kses_post()` as insufficient escaping | `wp_kses_post()` is valid for trusted content with safe HTML | Allows `<p>`, `<a>`, `<strong>`, etc. - appropriate for post content |
| Flagging admin-only `$wpdb->query()` with hardcoded SQL | No user input = no injection risk | Review for user input presence, not just prepare() usage |
| Flagging `current_user_can()` inside REST `permission_callback` as redundant | This is the CORRECT location for capability checks in REST API | REST API auth model, not redundant |
| Flagging `esc_html( get_the_title() )` as double-escaping | WP core sanitizes on storage, but late escaping is still best practice | Not technically double-escaping, WP stores raw data |
| Flagging `update_option()` in admin context as vulnerability | Options API is safe for admin use | Check capability first, then update_option() is fine |
| Flagging REST API with `__return_true` permission on GET endpoints | Public read access is often intentional | Only flag on write operations (POST, PUT, DELETE) |
| Flagging `sanitize_callback` in `register_setting()` as missing sanitization | Settings API handles sanitization via callback | This is the correct pattern for Settings API |

## Security Constants Check (SEC-19)

Review `wp-config.php` for security hardening constants:

```php
// ✅ GOOD: Disable file editor (INFO-level recommendation)
define( 'DISALLOW_FILE_EDIT', true );

// ✅ GOOD: Force SSL for admin (INFO if SSL available)
define( 'FORCE_SSL_ADMIN', true );

// ✅ GOOD: Disable unfiltered HTML for all users including admins
define( 'DISALLOW_UNFILTERED_HTML', true );

// ✅ GOOD: Disable plugin/theme installation (high-security environments)
define( 'DISALLOW_FILE_MODS', true );

// ✅ GOOD: Custom authentication keys (CRITICAL if using defaults)
define( 'AUTH_KEY',         'put-unique-phrase-here' ); // Use https://api.wordpress.org/secret-key/1.1/salt/
define( 'SECURE_AUTH_KEY',  'put-unique-phrase-here' );
define( 'LOGGED_IN_KEY',    'put-unique-phrase-here' );
define( 'NONCE_KEY',        'put-unique-phrase-here' );
define( 'AUTH_SALT',        'put-unique-phrase-here' );
define( 'SECURE_AUTH_SALT', 'put-unique-phrase-here' );
define( 'LOGGED_IN_SALT',   'put-unique-phrase-here' );
define( 'NONCE_SALT',       'put-unique-phrase-here' );
```

## Deep-Dive References

Load these references for comprehensive vulnerability patterns and examples:

| Task | Reference to Load |
|------|-------------------|
| Comprehensive vulnerability catalog with CWE mappings | `references/vulnerability-patterns.md` |
| Output escaping patterns and context-specific functions | `references/escaping-guide.md` |
| Input sanitization patterns by data type | `references/sanitization-guide.md` |
| Authorization and capability check patterns | `references/auth-patterns.md` |
| Nonce generation, verification, and CSRF prevention | `references/nonce-csrf-guide.md` |

**Note**: For standard security reviews, this SKILL.md contains all patterns needed. Load references when you need comprehensive CWE catalogs, edge cases, or platform-specific security requirements (WordPress VIP, etc.).
