# WordPress Output Escaping Guide

Output escaping is the final defense against Cross-Site Scripting (XSS) attacks. This guide covers WordPress's context-specific escaping functions, the late escaping principle, and common mistakes.

## Quick Reference Table

| Function | Output Context | Example | Notes |
|----------|---------------|---------|-------|
| `esc_html()` | HTML body text | `<p><?php echo esc_html( $text ); ?></p>` | Strips/encodes all HTML tags |
| `esc_attr()` | HTML attributes | `<div class="<?php echo esc_attr( $class ); ?>">` | Safe for quoted attributes |
| `esc_url()` | URL in href/src | `<a href="<?php echo esc_url( $url ); ?>">` | Validates scheme, rejects `javascript:` |
| `esc_js()` | Inline JavaScript | `<script>var x = '<?php echo esc_js( $val ); ?>';</script>` | Rare; prefer `wp_localize_script()` |
| `esc_textarea()` | Textarea content | `<textarea><?php echo esc_textarea( $text ); ?></textarea>` | Like `esc_html()` but preserves newlines |
| `esc_url_raw()` | URL in database/redirect | `update_option( 'url', esc_url_raw( $url ) );` | No encoding, validation only |
| `wp_kses()` | HTML with allowed tags | `echo wp_kses( $html, $allowed_tags );` | Whitelist specific tags/attributes |
| `wp_kses_post()` | Post content HTML | `echo wp_kses_post( $content );` | Allows post editor tags (trusted only) |
| `sanitize_text_field()` | Single-line text input | `$name = sanitize_text_field( wp_unslash( $_POST['name'] ) );` | **INPUT sanitization, not output** |

## The Late Escaping Principle

WordPress follows "late escaping" — data is sanitized on input (to ensure safe storage) and escaped on output (to ensure safe display). This allows the same data to be used in multiple contexts.

### Why Late Escaping?

```php
// ❌ BAD: Early escaping prevents reuse in different contexts
function prefix_save_title() {
    $title = esc_html( $_POST['title'] ); // Escaped for HTML context
    update_post_meta( $post_id, 'title', $title ); // Stored with HTML entities
}

function prefix_display_title() {
    $title = get_post_meta( $post_id, 'title', true ); // Contains &lt; &gt; entities

    // In HTML: shows escaped entities (&lt; displays as &amp;lt;)
    echo '<h1>' . esc_html( $title ) . '</h1>'; // Double-escaped!

    // In JSON API: escaping wrong for context
    return array( 'title' => $title ); // Still HTML-escaped!

    // In email: HTML entities in plain text
    wp_mail( $email, $title, $body ); // Subject has &lt; entities
}

// ✅ GOOD: Late escaping allows context-appropriate output
function prefix_save_title() {
    $title = sanitize_text_field( wp_unslash( $_POST['title'] ) ); // Sanitize, don't escape
    update_post_meta( $post_id, 'title', $title ); // Store clean data
}

function prefix_display_title() {
    $title = get_post_meta( $post_id, 'title', true ); // Clean data

    // In HTML: escape for HTML
    echo '<h1>' . esc_html( $title ) . '</h1>';

    // In JSON API: no escaping needed (json_encode handles it)
    return array( 'title' => $title );

    // In email: use as-is
    wp_mail( $email, $title, $body );

    // In attribute: escape for attribute
    echo '<div data-title="' . esc_attr( $title ) . '">';
}
```

### Input vs Output Functions

```php
// INPUT (sanitization) — when receiving data
$title = sanitize_text_field( wp_unslash( $_POST['title'] ) );
$email = sanitize_email( $_POST['email'] );
$url = sanitize_url( $_POST['website'] );
update_post_meta( $post_id, 'data', array(
    'title' => $title,
    'email' => $email,
    'url'   => $url,
) );

// OUTPUT (escaping) — when displaying data
$data = get_post_meta( $post_id, 'data', true );
echo '<h1>' . esc_html( $data['title'] ) . '</h1>';
echo '<a href="mailto:' . esc_attr( $data['email'] ) . '">Email</a>';
echo '<a href="' . esc_url( $data['url'] ) . '">Website</a>';
```

## Context-Specific Escaping Functions

### esc_html() — HTML Body Text

Use for text content within HTML tags. Converts `<`, `>`, `&`, `"`, `'` to HTML entities.

```php
// ❌ BAD: Unescaped output
$user_name = get_user_meta( $user_id, 'display_name', true );
echo '<p>Welcome, ' . $user_name . '</p>'; // XSS if name contains <script>

// ❌ BAD: Wrong context (allows tags to render)
$user_input = $_POST['comment'];
echo '<div>' . esc_attr( $user_input ) . '</div>'; // esc_attr doesn't strip tags!

// ✅ GOOD: Escape HTML context
$user_name = get_user_meta( $user_id, 'display_name', true );
echo '<p>Welcome, ' . esc_html( $user_name ) . '</p>';

// ✅ GOOD: Escaping translated strings
echo '<h1>' . esc_html__( 'Settings', 'textdomain' ) . '</h1>';

// ✅ GOOD: Escaping dynamic content
$post_title = get_the_title();
echo '<h2>' . esc_html( $post_title ) . '</h2>';
```

**What it does:**
- Converts: `<script>` → `&lt;script&gt;`
- Converts: `"Hello"` → `&quot;Hello&quot;`
- Strips: Invalid UTF-8 sequences

### esc_attr() — HTML Attributes

Use for text within HTML attributes (class, id, data-*, etc.).

```php
// ❌ BAD: Unescaped attribute
$user_id = $_GET['user'];
echo '<div data-user="' . $user_id . '">'; // XSS via quote injection: user=1" onload="alert(1)

// ❌ BAD: Wrong context (doesn't validate URL scheme)
$url = $_GET['url'];
echo '<a href="' . esc_attr( $url ) . '">'; // javascript: URLs pass through!

// ✅ GOOD: Escape attribute context
$user_id = absint( $_GET['user'] );
echo '<div data-user="' . esc_attr( $user_id ) . '">';

// ✅ GOOD: CSS class from user input
$class = sanitize_html_class( $_GET['theme'] );
echo '<div class="' . esc_attr( $class ) . '">';

// ✅ GOOD: Data attributes
$status = get_post_meta( $post_id, 'status', true );
echo '<button data-status="' . esc_attr( $status ) . '">Update</button>';
```

**What it does:**
- Converts: `"` → `&quot;`
- Converts: `<` → `&lt;`
- Safe for single/double quoted attributes

**Note:** Always quote attributes: `class="<?php echo esc_attr( $val ); ?>"` not `class=<?php echo esc_attr( $val ); ?>`

### esc_url() — URLs in href/src/action

Use for URLs in href, src, action, and other URL contexts. Validates URL schemes and rejects dangerous ones.

```php
// ❌ BAD: Unescaped URL
$redirect = $_GET['redirect_to'];
echo '<a href="' . $redirect . '">Next</a>'; // XSS via javascript: URL

// ❌ BAD: Wrong context (doesn't validate scheme)
$url = $_GET['url'];
echo '<a href="' . esc_html( $url ) . '">Link</a>'; // javascript: passes!

// ✅ GOOD: Validate and escape URL
$redirect = isset( $_GET['redirect_to'] ) ? $_GET['redirect_to'] : home_url();
echo '<a href="' . esc_url( $redirect ) . '">Next</a>';

// ✅ GOOD: User-submitted website
$website = get_user_meta( $user_id, 'website', true );
echo '<a href="' . esc_url( $website ) . '">' . esc_html( $website ) . '</a>';

// ✅ GOOD: Image src
$avatar_url = get_user_meta( $user_id, 'avatar', true );
echo '<img src="' . esc_url( $avatar_url ) . '" alt="Avatar">';

// ✅ GOOD: With allowed protocols
$custom_url = get_option( 'custom_protocol_url' );
echo '<a href="' . esc_url( $custom_url, array( 'http', 'https', 'ftp' ) ) . '">Download</a>';
```

**What it does:**
- Validates URL structure
- Allows: `http`, `https`, `ftp`, `ftps`, `mailto`, `news`, `irc`, `gopher`, `nntp`, `feed`, `telnet`
- Rejects: `javascript:`, `data:`, `vbscript:`, etc.
- Encodes special characters

**Note:** For database storage or `wp_safe_redirect()`, use `esc_url_raw()` instead (no encoding).

### esc_js() — Inline JavaScript Strings (Rare)

Use for inline JavaScript string literals. **PREFER `wp_localize_script()` for passing data to JavaScript.**

```php
// ❌ BAD: Unescaped JS
$user_name = get_user_meta( $user_id, 'name', true );
echo '<script>var name = "' . $user_name . '";</script>'; // XSS via quote escape

// ❌ BAD: Using esc_html in JS context
echo '<script>var name = "' . esc_html( $user_name ) . '";</script>'; // Wrong escaping!

// ⚠️ ACCEPTABLE: Using esc_js() for inline script
$user_name = get_user_meta( $user_id, 'name', true );
echo '<script>var name = "' . esc_js( $user_name ) . '";</script>';

// ✅ GOOD: Use wp_localize_script() instead
wp_localize_script( 'my-script', 'myData', array(
    'userName' => get_user_meta( $user_id, 'name', true ),
) );
// In JS file: console.log( myData.userName );

// ✅ GOOD: JSON encode for complex data
$user_data = get_user_meta( $user_id, 'data', true );
?>
<script>
var userData = <?php echo wp_json_encode( $user_data ); ?>;
</script>
```

**When to use:**
- Only for inline `<script>` tags when `wp_localize_script()` isn't feasible
- Prefer `wp_json_encode()` for complex data

### esc_textarea() — Textarea Content

Use for content within `<textarea>` tags. Similar to `esc_html()` but preserves line breaks.

```php
// ❌ BAD: Unescaped textarea
$bio = get_user_meta( $user_id, 'bio', true );
echo '<textarea>' . $bio . '</textarea>'; // XSS via </textarea> injection

// ✅ GOOD: Escape textarea
$bio = get_user_meta( $user_id, 'bio', true );
echo '<textarea name="bio">' . esc_textarea( $bio ) . '</textarea>';

// ✅ GOOD: With translation
echo '<textarea placeholder="' . esc_attr__( 'Enter bio...', 'textdomain' ) . '">';
echo esc_textarea( get_the_author_meta( 'description' ) );
echo '</textarea>';
```

**What it does:**
- Like `esc_html()` but keeps `\r\n` line breaks
- Escapes `</textarea>` to prevent tag injection

### esc_url_raw() — URLs for Database/Redirects

Use when storing URLs or passing to `wp_safe_redirect()`. Validates but doesn't encode.

```php
// ❌ BAD: Storing escaped URL
$website = esc_url( $_POST['website'] ); // Encoded for output
update_user_meta( $user_id, 'website', $website ); // Stores encoded URL!

// ✅ GOOD: Store raw, display escaped
$website = esc_url_raw( $_POST['website'] ); // Validates, no encoding
update_user_meta( $user_id, 'website', $website );

$stored_url = get_user_meta( $user_id, 'website', true );
echo '<a href="' . esc_url( $stored_url ) . '">'; // Encode for output

// ✅ GOOD: wp_safe_redirect() needs raw URL
$redirect_to = esc_url_raw( $_GET['redirect_to'] );
wp_safe_redirect( $redirect_to );
exit;
```

**Difference from esc_url():**
- `esc_url()`: Validates + encodes → for HTML output
- `esc_url_raw()`: Validates only → for storage/redirects

## wp_kses() Family — HTML Whitelist Filtering

For allowing specific HTML tags while stripping everything else.

### wp_kses() — Custom Allowed Tags

```php
// ❌ BAD: Allowing all HTML
$bio = $_POST['bio'];
update_user_meta( $user_id, 'bio', $bio ); // Stores <script>, <iframe>, etc.!

// ❌ BAD: Strip all HTML when some is needed
$bio = sanitize_text_field( $_POST['bio'] );
update_user_meta( $user_id, 'bio', $bio ); // Removes even <em>, <strong>

// ✅ GOOD: Allow specific formatting tags
$allowed_tags = array(
    'a' => array(
        'href'  => array(),
        'title' => array(),
    ),
    'br'     => array(),
    'em'     => array(),
    'strong' => array(),
    'p'      => array(),
);

$bio = wp_kses( wp_unslash( $_POST['bio'] ), $allowed_tags );
update_user_meta( $user_id, 'bio', $bio );

// Output: already filtered, just echo
echo '<div class="bio">' . $bio . '</div>';
```

**Allowed tag structure:**
```php
$allowed_tags = array(
    'tagname' => array(
        'attribute_name' => array(), // Allow attribute
        'class' => array(),          // Allow class attribute
        'id'    => array(),          // Allow id attribute
        'data-*' => true,            // Allow all data- attributes (WP 5.5+)
    ),
);
```

**Common allowed sets:**
```php
// Basic formatting
$basic_tags = array(
    'strong' => array(),
    'em'     => array(),
    'b'      => array(),
    'i'      => array(),
    'br'     => array(),
);

// Links and formatting
$formatted_tags = array(
    'a' => array( 'href' => array(), 'title' => array(), 'target' => array() ),
    'strong' => array(),
    'em'     => array(),
    'br'     => array(),
    'p'      => array(),
);

// Rich text (user bios, comments)
$rich_tags = array(
    'a' => array( 'href' => array(), 'title' => array() ),
    'br' => array(),
    'em' => array(),
    'strong' => array(),
    'p' => array(),
    'ul' => array(),
    'ol' => array(),
    'li' => array(),
    'blockquote' => array(),
);
```

### wp_kses_post() — Post Content HTML

Allows tags that the post editor allows. **Use only for content from trusted users (editors/admins).**

```php
// ❌ BAD: Using wp_kses_post() on arbitrary user input
$user_comment = wp_kses_post( $_POST['comment'] ); // Too permissive for comments!
wp_insert_comment( array( 'comment_content' => $user_comment ) );

// ❌ BAD: On untrusted user roles
$user_bio = wp_kses_post( $_POST['bio'] ); // Subscribers can inject <iframe>!
update_user_meta( $user_id, 'bio', $user_bio );

// ✅ GOOD: Only for trusted roles with post editing capability
if ( current_user_can( 'edit_posts' ) ) {
    $custom_content = wp_kses_post( wp_unslash( $_POST['content'] ) );
    update_post_meta( $post_id, 'custom_content', $custom_content );
}

// ✅ GOOD: For actual post content
$post_content = wp_kses_post( wp_unslash( $_POST['post_content'] ) );
wp_update_post( array(
    'ID'           => $post_id,
    'post_content' => $post_content,
) );
```

**What wp_kses_post() allows:**
- All post editor tags: `<p>`, `<h1-h6>`, `<blockquote>`, `<code>`, `<pre>`
- Media: `<img>`, `<audio>`, `<video>`, `<embed>`, `<iframe>` (from allowed domains)
- Formatting: `<strong>`, `<em>`, `<a>`, `<ul>`, `<ol>`, `<li>`, `<table>`

**When to use:** Only for content from users with `edit_posts` capability or higher.

## Escaping in Different Contexts

### Template Output

```php
// ❌ BAD: Multiple escaping mistakes in one template
<div class="user-card" data-id="<?php echo $user_id; ?>">
    <h3><?php echo $user_name; ?></h3>
    <a href="<?php echo $website; ?>">Visit Website</a>
    <p><?php echo $bio; ?></p>
</div>

// ✅ GOOD: Context-appropriate escaping
<div class="user-card" data-id="<?php echo esc_attr( $user_id ); ?>">
    <h3><?php echo esc_html( $user_name ); ?></h3>
    <a href="<?php echo esc_url( $website ); ?>"><?php echo esc_html__( 'Visit Website', 'textdomain' ); ?></a>
    <p><?php echo wp_kses_post( $bio ); ?></p>
</div>
```

### printf() / sprintf() Patterns

```php
// ❌ BAD: Escaping after printf
printf( '<a href="%s">%s</a>', $url, $text );
// String built before escaping!

// ✅ GOOD: Escape before printf
printf(
    '<a href="%s">%s</a>',
    esc_url( $url ),
    esc_html( $text )
);

// ✅ GOOD: With translation
printf(
    /* translators: %s: user name */
    esc_html__( 'Welcome, %s!', 'textdomain' ),
    esc_html( $user_name )
);
```

### Heredoc / Nowdoc Syntax

```php
// ❌ BAD: Unescaped heredoc
$output = <<<HTML
<div class="card">
    <h3>$title</h3>
    <p>$description</p>
</div>
HTML;
echo $output;

// ✅ GOOD: Escape within heredoc
$title_escaped = esc_html( $title );
$desc_escaped = esc_html( $description );
$output = <<<HTML
<div class="card">
    <h3>$title_escaped</h3>
    <p>$desc_escaped</p>
</div>
HTML;
echo $output;

// ✅ BETTER: Nowdoc with concatenation
$output = <<<'HTML'
<div class="card">
    <h3>
HTML;
$output .= esc_html( $title );
$output .= <<<'HTML'
</h3>
    <p>
HTML;
$output .= esc_html( $description );
$output .= <<<'HTML'
</p>
</div>
HTML;
echo $output;
```

### JavaScript Localization

```php
// ❌ BAD: Inline data without escaping
<script>
var config = {
    apiUrl: "<?php echo get_option( 'api_url' ); ?>",
    userName: "<?php echo $user_name; ?>"
};
</script>

// ✅ GOOD: wp_localize_script()
wp_localize_script( 'my-script', 'myConfig', array(
    'apiUrl'   => get_option( 'api_url' ),
    'userName' => $user_name,
    'nonce'    => wp_create_nonce( 'my_action' ),
) );

// In my-script.js:
// console.log( myConfig.apiUrl );
// console.log( myConfig.userName );

// ✅ GOOD: wp_json_encode() for complex data
<script>
var userData = <?php echo wp_json_encode( array(
    'id'    => get_current_user_id(),
    'name'  => wp_get_current_user()->display_name,
    'email' => wp_get_current_user()->user_email,
) ); ?>;
</script>
```

### REST API Responses

```php
// ❌ BAD: Escaping in REST response (breaks JSON)
function prefix_get_user( $request ) {
    $user = get_user_by( 'id', $request['id'] );

    return array(
        'name' => esc_html( $user->display_name ), // Don't escape here!
        'bio'  => esc_html( $user->description ),
    );
}

// ✅ GOOD: Return raw data, escape on frontend
function prefix_get_user( $request ) {
    $user = get_user_by( 'id', $request['id'] );

    return array(
        'name' => $user->display_name, // Raw data
        'bio'  => $user->description,  // Frontend escapes when rendering
    );
}

// Frontend JavaScript:
// fetch('/wp-json/myapp/v1/user/1')
//   .then(res => res.json())
//   .then(data => {
//     document.getElementById('name').textContent = data.name; // textContent escapes
//   });
```

### Shortcode Output

```php
// ❌ BAD: Unescaped shortcode attributes
function prefix_user_shortcode( $atts ) {
    $user_id = $atts['id'];
    $user = get_userdata( $user_id );

    return '<div class="user">' . $user->display_name . '</div>'; // XSS!
}
add_shortcode( 'user', 'prefix_user_shortcode' );

// ✅ GOOD: Sanitize attributes, escape output
function prefix_user_shortcode( $atts ) {
    $atts = shortcode_atts( array(
        'id' => 0,
    ), $atts, 'user' );

    $user_id = absint( $atts['id'] );
    $user = get_userdata( $user_id );

    if ( ! $user ) {
        return '';
    }

    return '<div class="user">' . esc_html( $user->display_name ) . '</div>';
}
add_shortcode( 'user', 'prefix_user_shortcode' );
```

## Common Escaping Mistakes

### 1. Concatenation Before Escaping

```php
// ❌ BAD: Building string before escaping
$output = '<h1>' . $title . '</h1>';
echo esc_html( $output ); // Escapes the HTML tags too!

// ✅ GOOD: Escape data, not markup
echo '<h1>' . esc_html( $title ) . '</h1>';
```

### 2. Wrong Function for Context

```php
// ❌ BAD: HTML escaping for URL
echo '<a href="' . esc_html( $url ) . '">'; // Doesn't validate scheme!

// ❌ BAD: Attribute escaping for HTML body
echo '<p>' . esc_attr( $text ) . '</p>'; // Doesn't strip tags!

// ✅ GOOD: Use correct function
echo '<a href="' . esc_url( $url ) . '">' . esc_html( $text ) . '</a>';
```

### 3. Escaping Before Storage

```php
// ❌ BAD: Escaping input
$title = esc_html( $_POST['title'] );
update_post_meta( $post_id, 'title', $title ); // Can't reuse in different context!

// ✅ GOOD: Sanitize input, escape output
$title = sanitize_text_field( wp_unslash( $_POST['title'] ) );
update_post_meta( $post_id, 'title', $title );

// Later, escape for context
echo '<h1>' . esc_html( get_post_meta( $post_id, 'title', true ) ) . '</h1>';
```

### 4. Double-Escaping

```php
// ❌ BAD: Escaping already-escaped data
$title = esc_html( $_POST['title'] );
update_post_meta( $post_id, 'title', $title );

$stored_title = get_post_meta( $post_id, 'title', true );
echo '<h1>' . esc_html( $stored_title ) . '</h1>'; // &amp;lt; instead of &lt;

// ✅ GOOD: Escape once, at output
$title = sanitize_text_field( wp_unslash( $_POST['title'] ) );
update_post_meta( $post_id, 'title', $title );

$stored_title = get_post_meta( $post_id, 'title', true );
echo '<h1>' . esc_html( $stored_title ) . '</h1>'; // Escaped correctly
```

### 5. Missing Escaping in Loops

```php
// ❌ BAD: Forgetting to escape in loop
foreach ( $users as $user ) {
    echo '<li>' . $user->display_name . '</li>'; // XSS!
}

// ✅ GOOD: Escape each item
foreach ( $users as $user ) {
    echo '<li>' . esc_html( $user->display_name ) . '</li>';
}
```

### 6. Trusting "Safe" Data Sources

```php
// ❌ BAD: Assuming post titles are safe
$post = get_post( $post_id );
echo '<h1>' . $post->post_title . '</h1>'; // Can contain HTML if editor inserted it!

// ✅ GOOD: Always escape output
$post = get_post( $post_id );
echo '<h1>' . esc_html( $post->post_title ) . '</h1>';

// Exception: When you need to allow HTML, use wp_kses_post()
echo '<div>' . wp_kses_post( $post->post_content ) . '</div>';
```

## WordPress i18n + Escaping

Translated strings must also be escaped. WordPress provides combined functions.

### Translation + Escaping Functions

```php
// ❌ BAD: Translating without escaping
echo '<h1>' . __( 'Welcome', 'textdomain' ) . '</h1>'; // If translation contains HTML...

// ❌ BAD: Escaping before translation
echo esc_html( '<h1>' . __( 'Welcome', 'textdomain' ) . '</h1>' ); // Escapes markup!

// ✅ GOOD: Use combined functions
echo '<h1>' . esc_html__( 'Welcome', 'textdomain' ) . '</h1>';
echo '<h1>' . esc_html_e( 'Welcome', 'textdomain' ) . '</h1>'; // Echoes directly

// ✅ GOOD: Attribute context
echo '<input type="text" placeholder="' . esc_attr__( 'Enter name...', 'textdomain' ) . '">';

// ✅ GOOD: With variables
printf(
    '<p>' . esc_html__( 'Hello, %s!', 'textdomain' ) . '</p>',
    esc_html( $user_name )
);
```

**Available combined functions:**
- `esc_html__()` — Translate and escape for HTML
- `esc_html_e()` — Translate, escape, and echo for HTML
- `esc_html_x()` — Translate with context and escape for HTML
- `esc_attr__()` — Translate and escape for attributes
- `esc_attr_e()` — Translate, escape, and echo for attributes
- `esc_attr_x()` — Translate with context and escape for attributes

### Translated Strings with HTML

```php
// ❌ BAD: HTML in translation that gets escaped
echo '<p>' . esc_html__( 'Please <a href="/terms">read terms</a>', 'textdomain' ) . '</p>';
// Outputs: Please &lt;a href="/terms"&gt;read terms&lt;/a&gt;

// ✅ GOOD: Use wp_kses() with allowed tags
$allowed = array( 'a' => array( 'href' => array() ) );
echo '<p>' . wp_kses( __( 'Please <a href="/terms">read terms</a>', 'textdomain' ), $allowed ) . '</p>';

// ✅ GOOD: sprintf with escaping
printf(
    /* translators: %s: terms URL */
    wp_kses( __( 'Please <a href="%s">read terms</a>', 'textdomain' ), array( 'a' => array( 'href' => array() ) ) ),
    esc_url( get_permalink( $terms_page_id ) )
);

// ✅ BETTER: Separate link from translation
printf(
    /* translators: %s: link to terms page */
    esc_html__( 'Please %s.', 'textdomain' ),
    '<a href="' . esc_url( get_permalink( $terms_page_id ) ) . '">' . esc_html__( 'read terms', 'textdomain' ) . '</a>'
);
```

## Summary

Key principles for WordPress output escaping:

1. **Late escaping** — Sanitize input, escape output
2. **Context-specific** — Use correct function for output context:
   - HTML body: `esc_html()`
   - Attributes: `esc_attr()`
   - URLs: `esc_url()`
   - JavaScript: `wp_localize_script()` or `wp_json_encode()`
   - Allowed HTML: `wp_kses()` with whitelist
3. **Escape every output** — Even "trusted" data from database
4. **Never escape before storage** — Prevents reuse in different contexts
5. **Combine i18n + escaping** — Use `esc_html__()`, `esc_attr_e()`, etc.

Cross-references:
- Input sanitization → sanitization-guide.md
- XSS vulnerability patterns → vulnerability-patterns.md
- Complete security review → SKILL.md
