# Phase 1: Security Review - Research

**Researched:** 2026-02-06
**Domain:** WordPress security vulnerability detection (code-level)
**Confidence:** HIGH

## Summary

WordPress security code review focuses on detecting XSS, SQL injection, CSRF/nonce verification, privilege escalation, file upload vulnerabilities, and authentication issues in WordPress 6.x+ PHP and JavaScript code. The standard approach follows WordPress's late escaping principle and context-aware detection to minimize false positives while catching real vulnerabilities.

Research confirmed all major security APIs remain stable in WordPress 6.x with extensive official documentation. The skill should mirror the performance skill's 12-section structure but with security-specific content organized by vulnerability category. Each finding maps to CWE references for credibility and includes concrete fix examples following WordPress PHP Coding Standards.

**Primary recommendation:** Structure the skill around WordPress's three-pillar security model: sanitize input early (sanitize_*), validate authorization (current_user_can/nonces), escape output late (esc_*). Detection patterns must be context-aware to avoid false positives in admin-only code, WP-CLI context, and REST API endpoints.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Finding output style:**
- Mirror the performance skill's output format: grouped by severity (Critical/Warning/Info), with line numbers, explanation, and fix per finding
- Security-specific addition: each finding includes a **CWE reference** where applicable (e.g., CWE-79 for XSS) to give findings credibility and searchability
- Severity mapping for security: CRITICAL = exploitable without authentication or leads to data breach, WARNING = exploitable with authentication or requires specific conditions, INFO = defense-in-depth improvement
- Group findings by file (like performance skill), not by vulnerability category — developers fix file by file, not vulnerability-class by vulnerability-class
- Each finding includes a concrete ❌ BAD / ✅ GOOD code pair showing the vulnerable pattern and the fix, following WordPress PHP Coding Standards

**Detection scope & depth:**
- **Deep coverage (dedicated sections in SKILL.md):** XSS (output escaping), SQL injection (wpdb::prepare), CSRF/nonce verification, capability/authorization checks, file upload validation
- **Surface-level coverage (scan patterns + brief guidance):** Object injection (unserialize), path traversal, open redirect, information disclosure, XML/XXE (rare in modern WP)
- **Context-aware detection:** The skill must distinguish execution contexts to avoid false positives:
  - Admin-only code (lower severity for some patterns since authenticated access required)
  - WP-CLI / cron context (nonce verification not applicable)
  - REST API endpoints (different auth model — permission_callback vs nonce)
  - Public-facing code (highest severity for unescaped output)
- **False positive guidance:** Include a "Common Mistakes" table (like performance skill) documenting known false-positive scenarios:
  - Nonces not required in WP-CLI context
  - `wp_kses_post()` is valid output for trusted content (not a missing-escape issue)
  - Admin-only `$wpdb->query()` with hardcoded SQL (no user input = no injection risk)
  - `current_user_can()` checks inside REST permission_callback are correct (not redundant)

**Reference doc tone & structure:**
- Follow the performance skill pattern: reference docs are **deep-dive companions** to the SKILL.md, not standalone guides
- Each reference doc is a **cookbook** — organized by pattern with ❌ BAD / ✅ GOOD examples and brief explanations
- Structure per reference doc:
  1. Quick reference table (pattern → risk → fix) for scanning
  2. Detailed patterns with code examples
  3. Edge cases and exceptions
  4. WordPress-specific nuances (e.g., late escaping philosophy, `wp_kses` family)
- Five reference docs planned:
  - `vulnerability-patterns.md` — comprehensive vulnerability catalog across all categories
  - `escaping-guide.md` — output escaping functions, context-specific escaping, late escaping principle
  - `sanitization-guide.md` — input sanitization functions, validation vs sanitization, type-specific patterns
  - `auth-patterns.md` — capability checks, role hierarchies, meta capabilities, REST permission callbacks
  - `nonce-csrf-guide.md` — nonce lifecycle, form nonces, AJAX nonces, REST nonce header, when nonces aren't needed

**Quick scan vs full review boundary:**
- `/wp-sec` (quick scan): Grep-based pattern detection only — runs bash grep commands against the codebase looking for high-risk signatures (like `$wpdb->query(` without `prepare`, `echo $` without escaping functions, missing `wp_verify_nonce`, `move_uploaded_file`, `unserialize`). Reports matches with file:line and severity. No deep analysis. Takes seconds.
- `/wp-sec-review` (full review): Complete skill workflow — file-type identification, context-aware analysis, severity assessment with CWE references, fix suggestions with code examples, loads reference docs for deeper patterns. Produces structured report.
- The boundary matches the performance skill exactly: quick scan = grep patterns from "Search Patterns for Quick Detection" section, full review = complete Code Review Workflow

### Claude's Discretion

- Exact grep patterns for quick detection (will be tuned during implementation based on false-positive rates)
- Ordering of vulnerability categories within SKILL.md sections
- Depth of edge-case coverage in reference docs (enough to be useful, not encyclopedic)
- Whether to include a "Platform Context" section (like performance skill has for hosting environments) — likely yes, with notes on managed hosts that have additional security layers

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope
</user_constraints>

## Standard Stack

The established security APIs and functions for WordPress security code review:

### Core Escaping Functions
| Function | Context | Purpose | When to Use |
|----------|---------|---------|-------------|
| `esc_html()` | HTML body | Strips/encodes HTML tags and entities | Visible text output within HTML elements |
| `esc_attr()` | HTML attributes | Escapes for use in HTML attributes | Values inside HTML attribute quotes |
| `esc_url()` | URLs | Validates and escapes URLs | All `href`, `src`, `action` attributes |
| `esc_js()` | JavaScript strings | Escapes for use in JavaScript | Inline JavaScript string values |
| `esc_textarea()` | Textarea content | Escapes for textarea elements | Content in `<textarea>` tags |
| `wp_kses()` | HTML with allowed tags | Filters HTML to allowed tags/attributes | User-generated HTML with whitelist |
| `wp_kses_post()` | Post content | Allows post content HTML tags | Trusted/editor content (not untrusted input) |

**Version:** WordPress 6.0+ (stable APIs, unchanged since WP 3.x)

### Core Sanitization Functions
| Function | Input Type | Purpose | Returns |
|----------|-----------|---------|---------|
| `sanitize_text_field()` | Generic text | Strips tags, octets, line breaks, extra whitespace | Plain text string |
| `sanitize_email()` | Email addresses | Validates email format per RFC subset | Valid email or empty |
| `sanitize_url()` | URLs | Removes invalid characters from URL | Clean URL string |
| `sanitize_key()` | Array keys/slugs | Lowercase alphanumeric + underscore/dash | Slug-safe string |
| `sanitize_file_name()` | File names | Removes special characters from filename | Safe filename |
| `absint()` | Integers | Converts to non-negative integer | Positive integer or 0 |
| `intval()` | Integers | Converts to integer (can be negative) | Integer value |
| `floatval()` | Floats | Converts to floating point | Float value |

**Version:** WordPress 6.0+ (stable APIs)

### Core CSRF/Nonce Functions
| Function | Purpose | Returns | Use Case |
|----------|---------|---------|----------|
| `wp_create_nonce()` | Creates nonce for action | Nonce string | Generate nonce before output |
| `wp_verify_nonce()` | Verifies nonce validity | False/1/2 | Manual verification in handlers |
| `wp_nonce_field()` | Adds nonce to form | HTML string | Form nonce hidden field |
| `wp_nonce_url()` | Adds nonce to URL | URL with nonce | Link-based actions |
| `check_ajax_referer()` | Verifies AJAX nonce | Dies or returns true | AJAX wp_ajax_* handlers |
| `check_admin_referer()` | Verifies admin action nonce | Dies or returns true | Admin form processing |

**Nonce validity:** 12-24 hours (two 12-hour "ticks")

### Core Authorization Functions
| Function | Purpose | Use Case |
|----------|---------|----------|
| `current_user_can()` | Check user capability | All privileged operations |
| `is_user_logged_in()` | Check if user authenticated | Conditional feature access |
| `map_meta_cap()` | Map meta caps to primitive caps | Custom capability logic (internal) |
| `get_role()` | Get role object | Role inspection (avoid role checks, use caps) |

**Meta vs Primitive Capabilities:**
- **Meta capabilities:** Context-aware (`edit_post`, `delete_post`, `edit_user`) — require object ID
- **Primitive capabilities:** Role-based (`edit_posts`, `edit_others_posts`, `manage_options`) — static checks

### Core Database Security
| API | Purpose | Security Benefit |
|-----|---------|------------------|
| `$wpdb->prepare()` | Parameterized queries | Prevents SQL injection via placeholders |
| `$wpdb->esc_like()` | Escape LIKE wildcards | Prevents LIKE clause injection |
| Placeholders: `%s` (string), `%d` (integer), `%f` (float) | Type-safe substitution | Forces data type constraints |

**CRITICAL:** Never pass user input to the query side of `->prepare()`. "Double-preparing" undermines security.

### REST API Security
| Mechanism | Purpose | Implementation |
|-----------|---------|----------------|
| `permission_callback` | Authorization check | Required in `register_rest_route()` (WP 5.5+) |
| `X-WP-Nonce` header | Cookie authentication | Set to `wp_create_nonce('wp_rest')` |
| `__return_true` | Public endpoint | Explicitly allows unauthenticated access |
| `current_user_can()` in callback | Capability check | Verify user has required capability |

**Best practice:** Always use `current_user_can()` for authorization checks, not `is_user_logged_in()`.

### File Upload Security
| Function/Hook | Purpose | Use Case |
|--------------|---------|----------|
| `wp_handle_upload()` | Process uploaded files | Standard upload handler |
| `wp_check_filetype_and_ext()` | Validate file type | Extension + MIME validation |
| `upload_mimes` filter | Whitelist allowed types | Restrict allowed file types |
| `wp_handle_upload_prefilter` hook | Pre-upload validation | Custom validation logic |

**Security Note:** WordPress 4.7.1+ includes content-based MIME validation via PHP `finfo` when available.

### Supporting Tools
| Tool | Purpose | When to Use |
|------|---------|-------------|
| `defined('ABSPATH')` check | Prevent direct file access | Top of plugin/theme PHP files |
| `DISALLOW_FILE_EDIT` constant | Disable file editor | Production wp-config.php hardening |
| `FORCE_SSL_ADMIN` constant | Force SSL for admin | Secure admin area in wp-config.php |

**Installation:**
All functions are WordPress core (no additional packages). Security constants go in `wp-config.php`.

## Architecture Patterns

### Recommended Code Review Order
```
1. Critical vulnerabilities first (SQL injection, XSS in public code, missing nonces)
2. Warnings (auth issues, admin-only XSS, file upload gaps)
3. Info (defense-in-depth, security constants)
4. Context-aware severity adjustment based on execution context
```

### Pattern 1: Late Escaping (WordPress Security Principle)
**What:** Escape data at the point of output, not before storage or during processing.

**When to use:** All output contexts (HTML, attributes, URLs, JavaScript).

**Why it works:** Preserves original data for all uses (APIs, formatting, different contexts) while preventing XSS.

**Example:**
```php
// ❌ BAD: Early escaping (before storage)
$title = esc_html( $_POST['title'] );
update_post_meta( $post_id, 'title', $title ); // Loses original for API use

// ✅ GOOD: Late escaping (at output)
$title = sanitize_text_field( $_POST['title'] ); // Sanitize input
update_post_meta( $post_id, 'title', $title );   // Store original

// Later, at output:
echo '<h1>' . esc_html( get_post_meta( $post_id, 'title', true ) ) . '</h1>';
```

**Source:** [WordPress VIP: Validating, sanitizing, and escaping](https://docs.wpvip.com/security/validating-sanitizing-and-escaping/)

### Pattern 2: Context-Specific Escaping
**What:** Choose escaping function based on output context.

**Contexts:**
- HTML body: `esc_html()`
- HTML attributes: `esc_attr()` (must use quotes around attribute)
- URLs (`href`, `src`): `esc_url()` (NOT `esc_attr()`)
- JavaScript strings: `esc_js()`
- Textarea content: `esc_textarea()`
- Rich HTML (whitelisted): `wp_kses()` or `wp_kses_post()`

**Example:**
```php
// ✅ GOOD: Context-specific escaping
<div class="<?php echo esc_attr( $class ); ?>"
     data-title="<?php echo esc_attr( $title ); ?>">
    <a href="<?php echo esc_url( $link ); ?>">
        <?php echo esc_html( $text ); ?>
    </a>
</div>

// ❌ BAD: Wrong function for context
<a href="<?php echo esc_attr( $link ); ?>"> <!-- Wrong: use esc_url() -->
```

**Source:** [WordPress Developer: Escaping Data](https://developer.wordpress.org/apis/security/escaping/)

### Pattern 3: Three-Step Form Security (Nonce + Capability + Sanitize)
**What:** Every form handler verifies nonce, checks capability, then sanitizes input.

**Steps:**
1. Verify nonce → prevents CSRF
2. Check capability → prevents privilege escalation
3. Sanitize input → prevents injection

**Example:**
```php
// ✅ GOOD: Complete form security
add_action( 'admin_post_save_settings', 'prefix_save_settings' );

function prefix_save_settings() {
    // Step 1: Verify nonce (CSRF protection)
    if ( ! isset( $_POST['settings_nonce'] ) ||
         ! wp_verify_nonce( $_POST['settings_nonce'], 'save_settings_action' ) ) {
        wp_die( 'Invalid nonce' ); // CWE-352
    }

    // Step 2: Check capability (authorization)
    if ( ! current_user_can( 'manage_options' ) ) {
        wp_die( 'Insufficient permissions' ); // CWE-863
    }

    // Step 3: Sanitize input
    $value = sanitize_text_field( $_POST['setting_value'] );
    update_option( 'prefix_setting', $value );

    wp_redirect( admin_url( 'admin.php?page=settings&updated=1' ) );
    exit;
}

// Form with nonce field:
<form method="post" action="<?php echo esc_url( admin_url( 'admin-post.php' ) ); ?>">
    <input type="hidden" name="action" value="save_settings">
    <?php wp_nonce_field( 'save_settings_action', 'settings_nonce' ); ?>
    <input type="text" name="setting_value" value="">
    <button type="submit">Save</button>
</form>
```

**Source:** [WordPress Developer: Nonces](https://developer.wordpress.org/apis/security/nonces/)

### Pattern 4: SQL Injection Prevention via wpdb::prepare()
**What:** Use parameterized queries with type-safe placeholders.

**Placeholders:**
- `%s` → strings
- `%d` → integers
- `%f` → floats

**Example:**
```php
global $wpdb;

// ❌ CRITICAL: Direct interpolation (SQL injection - CWE-89)
$user_id = $_GET['user_id'];
$results = $wpdb->get_results( "SELECT * FROM {$wpdb->users} WHERE ID = $user_id" );

// ✅ GOOD: Parameterized with wpdb::prepare()
$user_id = intval( $_GET['user_id'] ); // Sanitize first
$results = $wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM {$wpdb->users} WHERE ID = %d",
    $user_id
) );

// ❌ CRITICAL: Double-preparing (security bypass)
$where = $wpdb->prepare( "WHERE foo = %s", $_GET['data'] );
$query = $wpdb->prepare( "SELECT * FROM table $where LIMIT %d", 10 ); // WRONG!

// ✅ GOOD: Single prepare with all variables
$results = $wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM table WHERE foo = %s LIMIT %d",
    $_GET['data'],
    10
) );

// ❌ WARNING: LIKE with leading wildcard (full table scan)
$search = $_GET['search'];
$results = $wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM {$wpdb->posts} WHERE post_title LIKE %s",
    '%' . $wpdb->esc_like( $search ) . '%' // Full scan
) );

// ✅ GOOD: Trailing wildcard can use index
$results = $wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM {$wpdb->posts} WHERE post_title LIKE %s",
    $wpdb->esc_like( $search ) . '%' // Index scan
) );
```

**Source:** [Patchstack: SQL Injection in WordPress](https://patchstack.com/articles/sql-injection/)

### Pattern 5: REST API Security (permission_callback Required)
**What:** All REST endpoints must define authorization via `permission_callback` (required WP 5.5+).

**Example:**
```php
// ❌ CRITICAL: Missing permission_callback (vulnerable to unauthorized access)
register_rest_route( 'myapp/v1', '/users/(?P<id>\d+)', array(
    'methods'  => 'DELETE',
    'callback' => 'myapp_delete_user',
    // Missing: 'permission_callback'
) );

// ❌ CRITICAL: __return_true for privileged operation
register_rest_route( 'myapp/v1', '/users/(?P<id>\d+)', array(
    'methods'             => 'DELETE',
    'callback'            => 'myapp_delete_user',
    'permission_callback' => '__return_true', // Anyone can delete!
) );

// ✅ GOOD: Public endpoint (read-only)
register_rest_route( 'myapp/v1', '/posts', array(
    'methods'             => 'GET',
    'callback'            => 'myapp_get_posts',
    'permission_callback' => '__return_true', // Explicitly public
) );

// ✅ GOOD: Privileged operation with capability check
register_rest_route( 'myapp/v1', '/users/(?P<id>\d+)', array(
    'methods'             => 'DELETE',
    'callback'            => 'myapp_delete_user',
    'permission_callback' => function( $request ) {
        return current_user_can( 'delete_users' );
    },
) );

// ✅ GOOD: Context-aware authorization (edit own posts)
register_rest_route( 'myapp/v1', '/posts/(?P<id>\d+)', array(
    'methods'             => 'PUT',
    'callback'            => 'myapp_update_post',
    'permission_callback' => function( $request ) {
        $post = get_post( $request['id'] );
        return current_user_can( 'edit_post', $post->ID );
    },
) );
```

**Client-side nonce (JavaScript):**
```javascript
// ✅ GOOD: Set X-WP-Nonce header for cookie auth
fetch( '/wp-json/myapp/v1/posts', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-WP-Nonce': wpApiSettings.nonce // Localized from wp_localize_script()
    },
    body: JSON.stringify( data )
} );
```

**Source:** [WordPress Developer: REST API Authentication](https://developer.wordpress.org/rest-api/using-the-rest-api/authentication/)

### Pattern 6: AJAX Security (wp_ajax Hooks + Nonces)
**What:** AJAX handlers use `wp_ajax_*` hooks with nonce verification and capability checks.

**Hooks:**
- `wp_ajax_{action}` → logged-in users
- `wp_ajax_nopriv_{action}` → non-logged-in users

**Example:**
```php
// Backend handler
add_action( 'wp_ajax_load_more_posts', 'prefix_load_more_posts' );

function prefix_load_more_posts() {
    // Step 1: Verify nonce
    check_ajax_referer( 'load_more_nonce', 'security' );

    // Step 2: Check capability (if needed)
    if ( ! current_user_can( 'read' ) ) {
        wp_send_json_error( 'Insufficient permissions' );
    }

    // Step 3: Sanitize input
    $page = intval( $_POST['page'] );

    // Process request
    $posts = get_posts( array( 'paged' => $page ) );

    wp_send_json_success( $posts );
}
```

**Frontend JavaScript:**
```javascript
// Localize nonce in PHP:
// wp_localize_script( 'my-script', 'myAjax', array(
//     'ajax_url' => admin_url( 'admin-ajax.php' ),
//     'nonce'    => wp_create_nonce( 'load_more_nonce' )
// ) );

jQuery.post( myAjax.ajax_url, {
    action: 'load_more_posts',
    security: myAjax.nonce,
    page: 2
}, function( response ) {
    if ( response.success ) {
        // Handle data
    }
} );
```

**Source:** [MalCare: wp_verify_nonce](https://www.malcare.com/blog/wp_verify_nonce/)

### Pattern 7: File Upload Validation
**What:** Validate file type, size, and content before accepting uploads.

**Example:**
```php
// ✅ GOOD: Whitelist MIME types
add_filter( 'upload_mimes', 'prefix_restrict_mime_types' );

function prefix_restrict_mime_types( $mimes ) {
    // Remove all except images and PDFs
    return array(
        'jpg|jpeg|jpe' => 'image/jpeg',
        'png'          => 'image/png',
        'pdf'          => 'application/pdf',
    );
}

// ✅ GOOD: Pre-upload validation
add_filter( 'wp_handle_upload_prefilter', 'prefix_validate_upload' );

function prefix_validate_upload( $file ) {
    // Validate file type
    $filetype = wp_check_filetype_and_ext( $file['tmp_name'], $file['name'] );

    if ( ! $filetype['ext'] || ! $filetype['type'] ) {
        $file['error'] = 'Invalid file type.';
        return $file;
    }

    // Validate file size (5MB limit)
    if ( $file['size'] > 5 * 1024 * 1024 ) {
        $file['error'] = 'File too large (max 5MB).';
        return $file;
    }

    return $file;
}

// ❌ CRITICAL: Unchecked file upload with move_uploaded_file()
if ( isset( $_FILES['upload'] ) ) {
    move_uploaded_file( $_FILES['upload']['tmp_name'],
                        '/uploads/' . $_FILES['upload']['name'] ); // Path traversal risk!
}

// ✅ GOOD: Use wp_handle_upload() with capability check
if ( ! current_user_can( 'upload_files' ) ) {
    wp_die( 'Insufficient permissions' );
}

$upload = wp_handle_upload( $_FILES['file'], array( 'test_form' => false ) );

if ( isset( $upload['error'] ) ) {
    wp_die( $upload['error'] );
}
```

**Source:** [WordPress VIP: MIME Types](https://wpengine.com/support/mime-types-wordpress/)

### Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Fix |
|--------------|-------------|-----|
| **Using role checks instead of capabilities** | Roles vary by site, custom roles break checks | Always use `current_user_can('capability')` |
| **Verifying nonces in REST API with wp_verify_nonce()** | REST uses different auth model | Use `permission_callback` with `current_user_can()` |
| **Escaping before storage** | Loses original data, may double-escape | Sanitize input, escape output (late escaping) |
| **Using esc_attr() for URLs** | Doesn't validate URL structure | Always use `esc_url()` for href/src |
| **Checking is_user_logged_in() for authorization** | Authentication ≠ authorization | Use `current_user_can('capability')` |
| **Setting permission_callback to __return_true for privileged ops** | Allows unauthenticated access | Check capabilities in callback |
| **Double-preparing SQL** | Breaks parameter substitution security | Single `prepare()` with all variables |

## Don't Hand-Roll

Security problems that have established WordPress solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| **Output escaping** | Custom HTML entity encoding | `esc_html()`, `esc_attr()`, `esc_url()` | Context-aware, handles edge cases, UTF-8 safe |
| **CSRF protection** | Custom token generation/verification | WordPress nonces (`wp_create_nonce()`, `wp_verify_nonce()`) | Time-limited, user-scoped, action-scoped |
| **SQL injection prevention** | Manual escaping with `addslashes()` or `mysqli_real_escape_string()` | `$wpdb->prepare()` with placeholders | Type-safe, prevents double-escape issues |
| **Password hashing** | `md5()`, `sha1()`, or custom hashing | `wp_hash_password()`, `wp_check_password()` | Uses bcrypt, salted, WordPress-compatible |
| **Authorization checks** | Custom role/permission systems | `current_user_can()` with capabilities | Extensible, respects role hierarchy, meta caps |
| **Input sanitization** | Custom regex/strip_tags | `sanitize_text_field()`, `sanitize_email()`, etc. | Type-specific, UTF-8 safe, WordPress-consistent |
| **File upload handling** | Direct `move_uploaded_file()` | `wp_handle_upload()`, MIME filters | Validates type, checks permissions, prevents traversal |
| **Session management** | PHP `$_SESSION` | WordPress cookies, transients, user meta | Cache-friendly, multi-server compatible |
| **Nonce in AJAX** | Custom token in POST | `check_ajax_referer()` | Integrates with wp_ajax hooks, dies on failure |
| **REST API auth** | Custom header tokens | `permission_callback` + `current_user_can()` | Cookie auth, OAuth (plugins), application passwords |

**Key insight:** WordPress security functions are battle-tested across millions of sites. They handle UTF-8, multibyte characters, edge cases, and integrate with WordPress's user system. Custom implementations introduce vulnerabilities and maintenance burden.

## Common Pitfalls

### Pitfall 1: Flagging wp_kses_post() as Missing Escaping (False Positive)
**What goes wrong:** Reviewers flag `echo wp_kses_post( $content )` as unescaped output.

**Why it happens:** `wp_kses_post()` doesn't have "esc" in the name, looks like unsafe output.

**Context:** `wp_kses_post()` **is** an escaping function. It allows post content HTML (same as the post editor) and strips everything else. It's appropriate for trusted content from editors/admins.

**How to avoid:** Recognize `wp_kses()` and `wp_kses_post()` as valid escaping functions. Flag only if used on **untrusted** user input (comments, front-end forms).

**Warning signs:**
- `wp_kses_post()` used on `$_POST` from public form → FLAG (allows too much HTML)
- `wp_kses_post()` used on saved post content → SAFE (trusted content)

**CWE Reference:** CWE-79 (XSS) - but this is a false positive in trusted contexts.

### Pitfall 2: Flagging Missing Nonces in WP-CLI Commands (False Positive)
**What goes wrong:** Code review flags missing `wp_verify_nonce()` in WP-CLI command handlers.

**Why it happens:** Security checklist says "all privileged actions need nonce verification."

**Context:** WP-CLI runs from command line, not HTTP requests. No CSRF risk, no nonces needed. Nonces are for web requests only.

**How to avoid:** Check execution context. If code is in:
- `if ( defined( 'WP_CLI' ) && WP_CLI ) { ... }` → Skip nonce checks
- WP-Cron callback → Skip nonce checks (server-initiated)
- AJAX/form handler → Require nonce checks

**Warning signs:**
- Command-line tools with nonce verification → FLAG (unnecessary, breaks CLI)
- AJAX handlers without nonce verification → FLAG (CSRF vulnerability)

**CWE Reference:** CWE-352 (CSRF) - not applicable in CLI/cron context.

### Pitfall 3: Flagging Admin-Only $wpdb->query() with Hardcoded SQL (False Positive)
**What goes wrong:** Code review flags `$wpdb->query( "CREATE TABLE..." )` in admin-only code as SQL injection risk.

**Why it happens:** Rule says "always use wpdb::prepare() for queries."

**Context:** Hardcoded SQL with zero user input has no injection vector. If the query is in plugin activation, admin settings, or otherwise not processing user input, it's safe.

**How to avoid:**
1. Check if query contains variables from user input (`$_GET`, `$_POST`, `$_REQUEST`)
2. If hardcoded string with no interpolation → SAFE
3. If uses variables → Must use `->prepare()` or validated/sanitized

**Warning signs:**
- `$wpdb->query( "CREATE TABLE {$table_name}..." )` with validated `$table_name` → SAFE (if `$table_name` is hardcoded prefix)
- `$wpdb->query( "SELECT * FROM posts WHERE ID = $id" )` with user `$id` → FLAG (SQL injection)

**CWE Reference:** CWE-89 (SQL Injection) - not applicable without user input.

### Pitfall 4: Flagging current_user_can() Inside REST permission_callback as Redundant (False Positive)
**What goes wrong:** Reviewer sees `permission_callback` AND `current_user_can()` inside callback, flags as "redundant check."

**Why it happens:** Misunderstanding that `permission_callback` is the **place** to check capabilities, not a capability check itself.

**Context:** `permission_callback` is a **function** that runs authorization logic. It should contain `current_user_can()` checks. This is the correct pattern, not redundant.

**How to avoid:** Understand that:
- `permission_callback => '__return_true'` → Public endpoint
- `permission_callback => function() { return current_user_can('...'); }` → Capability-protected endpoint

Both are correct patterns for their use cases.

**Warning signs:**
- `permission_callback` with `current_user_can()` inside → SAFE (correct pattern)
- Missing `permission_callback` entirely → FLAG (required WP 5.5+)
- `permission_callback => '__return_true'` for write operations → FLAG (authorization bypass)

**CWE Reference:** CWE-863 (Incorrect Authorization) - but this is correct usage.

### Pitfall 5: Missing Context-Specific Escaping (Real Vulnerability)
**What goes wrong:** Developer uses `esc_html()` for all output, including URLs and attributes.

**Why it happens:** "Escape all output" rule applied without context awareness.

**Context:** Different output contexts require different escaping functions:
- HTML body: `esc_html()`
- HTML attributes: `esc_attr()`
- URLs: `esc_url()` (NOT `esc_attr()`)
- JavaScript: `esc_js()`

**How to avoid:** Match escaping function to output context. Using `esc_html()` for a URL in `href` doesn't validate the URL scheme, allowing `javascript:` XSS.

**Warning signs:**
```php
// ❌ BAD: esc_html() used in href (doesn't validate URL)
<a href="<?php echo esc_html( $url ); ?>">

// ❌ BAD: esc_attr() used in href (doesn't validate URL scheme)
<a href="<?php echo esc_attr( $url ); ?>">

// ✅ GOOD: esc_url() validates and escapes URLs
<a href="<?php echo esc_url( $url ); ?>">
```

**CWE Reference:** CWE-79 (XSS) via improper escaping.

### Pitfall 6: Late Escaping Breaking on Concatenation (Real Vulnerability)
**What goes wrong:** Developer concatenates strings, then escapes the result.

**Why it happens:** Misunderstanding of when to escape - escaping happens on each variable individually, not on concatenated output.

**Context:** Concatenating unescaped strings creates XSS window. Each variable must be escaped before concatenation.

**How to avoid:** Escape each variable **before** concatenation or use template functions.

**Warning signs:**
```php
// ❌ BAD: Concatenate first, escape later
$output = '<div class="' . $class . '">' . $content . '</div>';
echo esc_html( $output ); // esc_html strips the HTML tags!

// ✅ GOOD: Escape each variable in context
echo '<div class="' . esc_attr( $class ) . '">' . esc_html( $content ) . '</div>';

// ✅ BETTER: Use template function
printf( '<div class="%s">%s</div>', esc_attr( $class ), esc_html( $content ) );
```

**CWE Reference:** CWE-79 (XSS) via improper escaping order.

### Pitfall 7: Unserialize on User Input (Real Vulnerability)
**What goes wrong:** Code calls `unserialize()` on data from `$_POST`, `$_GET`, or database that users can modify.

**Why it happens:** Developer uses `serialize()` to store complex data structures, then deserializes without validation.

**Context:** PHP object injection (CWE-502) allows attackers to instantiate arbitrary classes and trigger magic methods (`__wakeup`, `__destruct`) with attacker-controlled properties. Can lead to RCE via gadget chains.

**How to avoid:**
1. Use `json_encode()` / `json_decode()` instead of `serialize()` / `unserialize()`
2. If `unserialize()` required, set `allowed_classes => false` option
3. Never unserialize user input directly

**Warning signs:**
```php
// ❌ CRITICAL: Unserialize user input (object injection)
$data = unserialize( $_POST['data'] );

// ❌ CRITICAL: Unserialize from database modified by users
$user_data = get_user_meta( $user_id, 'preferences', true );
$prefs = unserialize( $user_data ); // User can modify their meta

// ✅ GOOD: Use JSON instead
$data = json_decode( $_POST['data'], true );

// ✅ ACCEPTABLE: Unserialize with allowed_classes = false
$data = unserialize( $_POST['data'], array( 'allowed_classes' => false ) );
```

**CWE Reference:** CWE-502 (Deserialization of Untrusted Data)

### Pitfall 8: Path Traversal in File Includes (Real Vulnerability)
**What goes wrong:** Code includes files based on user input without validation.

**Why it happens:** Dynamic includes seem convenient for loading templates or modules.

**Context:** Attackers use `../` sequences to escape intended directory and read sensitive files (`wp-config.php`, `.env`, `/etc/passwd`).

**How to avoid:**
1. Whitelist allowed filenames (never allow user to specify arbitrary paths)
2. Use `basename()` to strip directory components
3. Validate against allowed values before including

**Warning signs:**
```php
// ❌ CRITICAL: Direct user input in include (path traversal)
include( $_GET['template'] . '.php' );
// Attacker: ?template=../../../../wp-config

// ❌ CRITICAL: Insufficient validation
$file = str_replace( '..', '', $_GET['file'] );
include( $file ); // Can bypass with ....// (becomes ../ after replace)

// ✅ GOOD: Whitelist allowed templates
$allowed = array( 'header', 'footer', 'sidebar' );
$template = $_GET['template'];
if ( in_array( $template, $allowed, true ) ) {
    include( $template . '.php' );
}

// ✅ GOOD: Use basename to strip paths
$template = basename( $_GET['template'] ); // Removes ../
include( __DIR__ . '/templates/' . $template . '.php' );
```

**CWE Reference:** CWE-22 (Path Traversal)

## Code Examples

Verified patterns from official sources and research:

### XSS Prevention (Output Escaping)
```php
// Source: https://developer.wordpress.org/apis/security/escaping/

// ✅ GOOD: Context-specific escaping
<h1><?php echo esc_html( get_the_title() ); ?></h1>
<img src="<?php echo esc_url( $image_url ); ?>"
     alt="<?php echo esc_attr( $image_alt ); ?>">
<div class="<?php echo esc_attr( $css_class ); ?>">
    <?php echo wp_kses_post( get_the_content() ); ?>
</div>

// ❌ BAD: No escaping (XSS - CWE-79)
<h1><?php echo get_the_title(); ?></h1>

// ❌ BAD: Wrong function for context (XSS - CWE-79)
<a href="<?php echo esc_attr( $url ); ?>"> <!-- Use esc_url() -->

// ❌ BAD: Escaping too early
$title = esc_html( $_POST['title'] );
update_post_meta( $id, 'title', $title ); // Loses original for API
```

### SQL Injection Prevention
```php
// Source: https://patchstack.com/articles/sql-injection/

global $wpdb;

// ✅ GOOD: Parameterized query with wpdb::prepare()
$user_id = intval( $_GET['user_id'] );
$status  = sanitize_text_field( $_GET['status'] );

$results = $wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM {$wpdb->posts} WHERE post_author = %d AND post_status = %s",
    $user_id,
    $status
) );

// ❌ CRITICAL: Direct interpolation (SQL injection - CWE-89)
$user_id = $_GET['user_id'];
$results = $wpdb->get_results(
    "SELECT * FROM {$wpdb->posts} WHERE post_author = $user_id"
);

// ❌ CRITICAL: Concatenation with user input (SQL injection - CWE-89)
$search = $_GET['search'];
$sql = "SELECT * FROM {$wpdb->posts} WHERE post_title LIKE '%" . $search . "%'";
$results = $wpdb->get_results( $sql );

// ✅ GOOD: LIKE escaping
$search = sanitize_text_field( $_GET['search'] );
$results = $wpdb->get_results( $wpdb->prepare(
    "SELECT * FROM {$wpdb->posts} WHERE post_title LIKE %s",
    $wpdb->esc_like( $search ) . '%'
) );
```

### CSRF Prevention (Nonces)
```php
// Source: https://developer.wordpress.org/apis/security/nonces/

// ✅ GOOD: Form with nonce
<form method="post" action="<?php echo esc_url( admin_url( 'admin-post.php' ) ); ?>">
    <input type="hidden" name="action" value="save_settings">
    <?php wp_nonce_field( 'save_settings_action', 'settings_nonce' ); ?>
    <input type="text" name="setting_value">
    <button type="submit">Save</button>
</form>

// Handler
add_action( 'admin_post_save_settings', 'prefix_save_settings' );

function prefix_save_settings() {
    // ✅ GOOD: Verify nonce
    if ( ! isset( $_POST['settings_nonce'] ) ||
         ! wp_verify_nonce( $_POST['settings_nonce'], 'save_settings_action' ) ) {
        wp_die( 'Invalid nonce' );
    }

    if ( ! current_user_can( 'manage_options' ) ) {
        wp_die( 'Insufficient permissions' );
    }

    $value = sanitize_text_field( $_POST['setting_value'] );
    update_option( 'prefix_setting', $value );

    wp_redirect( admin_url( 'admin.php?page=settings&updated=1' ) );
    exit;
}

// ❌ CRITICAL: No nonce verification (CSRF - CWE-352)
function prefix_save_settings_bad() {
    update_option( 'prefix_setting', $_POST['setting_value'] );
    wp_redirect( admin_url( 'admin.php?page=settings' ) );
    exit;
}
```

### Authorization Checks
```php
// Source: https://developer.wordpress.org/plugins/security/checking-user-capabilities/

// ✅ GOOD: Capability check before privileged operation
if ( ! current_user_can( 'edit_post', $post_id ) ) {
    wp_die( 'You do not have permission to edit this post.' );
}

// Update the post...
wp_update_post( array( 'ID' => $post_id, 'post_content' => $content ) );

// ❌ CRITICAL: No capability check (privilege escalation - CWE-862)
function prefix_delete_post_bad() {
    $post_id = intval( $_POST['post_id'] );
    wp_delete_post( $post_id, true ); // Anyone can delete!
}

// ❌ WARNING: Role check instead of capability (brittle)
if ( in_array( 'administrator', wp_get_current_user()->roles ) ) {
    // Do admin thing
}

// ✅ GOOD: Capability check (respects custom roles)
if ( current_user_can( 'manage_options' ) ) {
    // Do admin thing
}

// ✅ GOOD: Meta capability (context-aware)
if ( current_user_can( 'edit_post', $post_id ) ) {
    // Checks if user is author OR has edit_others_posts
}
```

### REST API Security
```php
// Source: https://developer.wordpress.org/rest-api/extending-the-rest-api/adding-custom-endpoints/

// ✅ GOOD: Public read endpoint
register_rest_route( 'myapp/v1', '/posts', array(
    'methods'             => 'GET',
    'callback'            => 'myapp_get_posts',
    'permission_callback' => '__return_true',
) );

// ✅ GOOD: Authenticated write endpoint
register_rest_route( 'myapp/v1', '/posts', array(
    'methods'             => 'POST',
    'callback'            => 'myapp_create_post',
    'permission_callback' => function() {
        return current_user_can( 'publish_posts' );
    },
) );

// ✅ GOOD: Context-aware authorization
register_rest_route( 'myapp/v1', '/posts/(?P<id>\d+)', array(
    'methods'             => 'DELETE',
    'callback'            => 'myapp_delete_post',
    'permission_callback' => function( $request ) {
        $post = get_post( $request['id'] );
        return current_user_can( 'delete_post', $post->ID );
    },
) );

// ❌ CRITICAL: Missing permission_callback (WP 5.5+ throws warning)
register_rest_route( 'myapp/v1', '/posts', array(
    'methods'  => 'POST',
    'callback' => 'myapp_create_post',
) );

// ❌ CRITICAL: __return_true for privileged operation
register_rest_route( 'myapp/v1', '/users/(?P<id>\d+)', array(
    'methods'             => 'DELETE',
    'callback'            => 'myapp_delete_user',
    'permission_callback' => '__return_true', // Anyone can delete users!
) );
```

### AJAX Security
```php
// Source: https://www.malcare.com/blog/wp_verify_nonce/

// Backend
add_action( 'wp_ajax_load_more_posts', 'prefix_load_more_posts' );

function prefix_load_more_posts() {
    // ✅ GOOD: Verify nonce
    check_ajax_referer( 'load_more_nonce', 'security' );

    // ✅ GOOD: Check capability
    if ( ! current_user_can( 'read' ) ) {
        wp_send_json_error( 'Insufficient permissions' );
    }

    // ✅ GOOD: Sanitize input
    $page = intval( $_POST['page'] );

    $posts = get_posts( array( 'paged' => $page ) );

    wp_send_json_success( $posts );
}

// Frontend JavaScript
// (Assuming nonce localized via wp_localize_script())
jQuery.post( myAjax.ajax_url, {
    action: 'load_more_posts',
    security: myAjax.nonce,
    page: 2
}, function( response ) {
    if ( response.success ) {
        // Handle data
    }
} );

// ❌ CRITICAL: No nonce verification (CSRF - CWE-352)
add_action( 'wp_ajax_delete_post', 'prefix_delete_post_bad' );
function prefix_delete_post_bad() {
    $post_id = intval( $_POST['post_id'] );
    wp_delete_post( $post_id, true ); // CSRF + no auth check!
}
```

### File Upload Validation
```php
// Source: https://wpengine.com/support/mime-types-wordpress/

// ✅ GOOD: Restrict allowed MIME types
add_filter( 'upload_mimes', 'prefix_restrict_mime_types' );

function prefix_restrict_mime_types( $mimes ) {
    return array(
        'jpg|jpeg|jpe' => 'image/jpeg',
        'png'          => 'image/png',
        'gif'          => 'image/gif',
        'pdf'          => 'application/pdf',
    );
}

// ✅ GOOD: Pre-upload validation
add_filter( 'wp_handle_upload_prefilter', 'prefix_validate_upload' );

function prefix_validate_upload( $file ) {
    // Check capability
    if ( ! current_user_can( 'upload_files' ) ) {
        $file['error'] = 'You do not have permission to upload files.';
        return $file;
    }

    // Validate file type
    $filetype = wp_check_filetype_and_ext( $file['tmp_name'], $file['name'] );

    if ( ! $filetype['ext'] || ! $filetype['type'] ) {
        $file['error'] = 'Invalid file type. Only images and PDFs allowed.';
        return $file;
    }

    // Validate file size (5MB)
    if ( $file['size'] > 5 * 1024 * 1024 ) {
        $file['error'] = 'File too large. Maximum size is 5MB.';
        return $file;
    }

    return $file;
}

// ❌ CRITICAL: Unchecked move_uploaded_file (path traversal + unrestricted upload)
if ( isset( $_FILES['upload'] ) ) {
    move_uploaded_file(
        $_FILES['upload']['tmp_name'],
        '/uploads/' . $_FILES['upload']['name']
    );
}
```

### Direct File Access Prevention
```php
// Source: https://www.pontikis.net/blog/what-is-abspath-in-wordpress-and-how-to-use-for-security

// ✅ GOOD: ABSPATH check at top of plugin/theme files
if ( ! defined( 'ABSPATH' ) ) {
    exit; // Exit if accessed directly
}

// Plugin code here...

// ❌ BAD: No ABSPATH check (information disclosure if accessed directly)
// Plugin code without protection...
```

## State of the Art

| Old Approach | Current Approach (WP 6.x) | When Changed | Impact |
|--------------|---------------------------|--------------|--------|
| `permission_callback` optional | `permission_callback` **required** in REST routes | WP 5.5 (2020) | Throws PHP warning if missing, forces explicit auth |
| Nonces valid 12 hours | Nonces valid 12-24 hours (two ticks) | WP 3.0 | More user-friendly, same security |
| `wp_remote_get()` no SSL verification | SSL verification enabled by default | WP 4.0 | Prevents MITM attacks on API calls |
| MIME type detection via extension | Content-based MIME validation | WP 4.7.1 (2017) | Prevents disguised malicious files |
| `serialize()` / `unserialize()` common | `json_encode()` / `json_decode()` preferred | Ongoing shift | Prevents object injection (CWE-502) |
| Role checks (`in_array($user->roles)`) | Capability checks (`current_user_can()`) | Always recommended | Respects custom roles, meta caps |
| admin-ajax.php for all AJAX | REST API preferred for new code | WP 4.7+ (2016) | Leaner bootstrap, better caching |

**Deprecated/outdated:**
- **`addslashes()` for SQL:** Use `$wpdb->prepare()` instead. `addslashes()` doesn't handle multibyte character exploits.
- **`strip_tags()` for XSS prevention:** Use `wp_kses()` or `esc_html()`. `strip_tags()` is not context-aware.
- **Role checks for authorization:** Use `current_user_can()` with capabilities. Roles vary per site.
- **`__return_true` for all REST endpoints:** Explicitly set for public endpoints only. Forces conscious security decision.

## Open Questions

Things that couldn't be fully resolved:

1. **Grep pattern false-positive rates for quick scan**
   - What we know: Common patterns like `echo $` will have high false positives (many legitimate uses with inline escaping)
   - What's unclear: Optimal balance between coverage and noise for the `/wp-sec` quick scan
   - Recommendation: Start with high-confidence patterns (missing `->prepare()`, `unserialize($_`, `move_uploaded_file`) and add more aggressive patterns in later iterations based on user feedback

2. **Context detection reliability for automated reviews**
   - What we know: Context matters (admin vs public, WP-CLI vs web) but detecting context programmatically is complex
   - What's unclear: How reliably can Claude detect execution context from code snippets without full codebase analysis
   - Recommendation: Include context-checking guidance in "Common Mistakes" table and instruct reviewers to check for `is_admin()`, `defined('WP_CLI')`, etc.

3. **CWE mapping for WordPress-specific patterns**
   - What we know: Standard vulnerabilities map to CWEs (XSS→79, SQLi→89, CSRF→352)
   - What's unclear: CWE mapping for WordPress-specific issues (missing `permission_callback`, ABSPATH bypass)
   - Recommendation: Use closest general CWE + WordPress-specific explanation. E.g., missing `permission_callback` → CWE-862 (Missing Authorization) + "WordPress 5.5+ requires explicit permission_callback"

4. **Platform Context section depth**
   - What we know: Managed WordPress hosts (WP VIP, WP Engine) have additional security layers
   - What's unclear: How much platform-specific guidance to include without making research outdated
   - Recommendation: Include high-level Platform Context section noting managed hosts may have additional protections, but avoid platform-specific API recommendations (users can check host docs)

## Sources

### Primary (HIGH confidence)

**WordPress Official Documentation:**
- [Escaping Data – Common APIs Handbook](https://developer.wordpress.org/apis/security/escaping/)
- [Sanitizing Data – Common APIs Handbook](https://developer.wordpress.org/apis/security/sanitizing/)
- [Nonces – Common APIs Handbook](https://developer.wordpress.org/apis/security/nonces/)
- [Checking User Capabilities – Plugin Handbook](https://developer.wordpress.org/plugins/security/checking-user-capabilities/)
- [REST API Authentication](https://developer.wordpress.org/rest-api/using-the-rest-api/authentication/)
- [Adding Custom REST Endpoints](https://developer.wordpress.org/rest-api/extending-the-rest-api/adding-custom-endpoints/)
- [WordPress Hardening – Advanced Administration](https://developer.wordpress.org/advanced-administration/security/hardening/)

**WordPress Function References:**
- [current_user_can()](https://developer.wordpress.org/reference/functions/current_user_can/)
- [esc_html()](https://developer.wordpress.org/reference/functions/esc_html/)
- [esc_attr()](https://developer.wordpress.org/reference/functions/esc_attr/)
- [esc_url()](https://developer.wordpress.org/reference/functions/esc_url/)
- [sanitize_text_field()](https://developer.wordpress.org/reference/functions/sanitize_text_field/)
- [sanitize_email()](https://developer.wordpress.org/reference/functions/sanitize_email/)
- [wp_kses()](https://developer.wordpress.org/reference/functions/wp_kses/)
- [wp_kses_post()](https://developer.wordpress.org/reference/functions/wp_kses_post/)
- [check_ajax_referer()](https://developer.wordpress.org/reference/functions/check_ajax_referer/)
- [register_rest_route()](https://developer.wordpress.org/reference/functions/register_rest_route/)
- [map_meta_cap()](https://developer.wordpress.org/reference/functions/map_meta_cap/)
- [wpdb class](https://developer.wordpress.org/reference/classes/wpdb/)

**CWE References (MITRE):**
- [CWE-79: Cross-site Scripting (XSS)](https://cwe.mitre.org/data/definitions/79.html)
- [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)
- [CWE-352: Cross-Site Request Forgery (CSRF)](https://cwe.mitre.org/data/definitions/352.html)
- [CWE-22: Path Traversal](https://cwe.mitre.org/data/definitions/22.html)
- [CWE-502: Deserialization of Untrusted Data](https://cwe.mitre.org/data/definitions/502.html)
- [CWE-862: Missing Authorization](https://cwe.mitre.org/data/definitions/862.html)
- [CWE-863: Incorrect Authorization](https://cwe.mitre.org/data/definitions/863.html)

### Secondary (MEDIUM confidence)

**WordPress VIP Documentation:**
- [Validating, sanitizing, and escaping](https://docs.wpvip.com/security/validating-sanitizing-and-escaping/)

**Security Analysis Articles (verified against official docs):**
- [Patchstack: SQL Injection in WordPress](https://patchstack.com/articles/sql-injection/)
- [MalCare: wp_verify_nonce](https://www.malcare.com/blog/wp_verify_nonce/)
- [MalCare: WordPress CSRF](https://www.malcare.com/blog/wordpress-csrf/)
- [WP VIP Blog: CSRF Attacks with WordPress Nonces](https://wpvip.com/blog/how-to-protect-against-csrf-attacks-with-wordpress-nonces/)

**WordPress Developer Blog:**
- [Understand and use WordPress nonces properly](https://developer.wordpress.org/news/2023/08/understand-and-use-wordpress-nonces-properly/)

**Security Research (cross-referenced):**
- [Plugin Vulnerabilities: WordPress REST API permission_callback](https://www.pluginvulnerabilities.com/2022/12/13/how-to-properly-restrict-access-to-wordpress-rest-api-routes/)
- [SonarSource: WordPress Object Injection Vulnerability](https://www.sonarsource.com/blog/wordpress-object-injection-vulnerability/)
- [OWASP: PHP Object Injection](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection)

### Tertiary (LOW confidence - requires validation)

**General WordPress Security Articles:**
- Various 2026 WordPress security best practices guides (used for current trends, not authoritative API info)
- Community tutorials on escaping/sanitization (verified against official docs before including)

## Metadata

**Confidence breakdown:**
- Standard stack (escaping/sanitizing/nonce functions): **HIGH** - Official WordPress documentation, stable APIs since WP 3.x
- Architecture patterns (late escaping, three-step form security): **HIGH** - Official WordPress VIP and Developer docs
- CWE mappings: **HIGH** - MITRE official CWE database
- Grep patterns for quick detection: **MEDIUM** - Based on research and security articles, needs field testing for false-positive rates
- Context-aware detection guidance: **MEDIUM** - Based on WordPress patterns, but automated context detection complexity not fully resolved
- Platform-specific security: **MEDIUM** - General guidance available, but platform-specific details beyond research scope

**Research date:** 2026-02-06
**Valid until:** 30-60 days (WordPress security APIs stable, but vulnerability landscape evolves; CWE references evergreen)
