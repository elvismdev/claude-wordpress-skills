# WordPress Input Sanitization Guide

Input sanitization ensures user-provided data is safe to store and process. This guide covers WordPress's type-specific sanitization functions, the distinction between sanitization and validation, and the critical `wp_unslash()` requirement.

## Quick Reference Table

| Function | Input Type | Returns | When to Use |
|----------|-----------|---------|-------------|
| `sanitize_text_field()` | Single-line text | String | Text inputs, titles, single-line fields |
| `sanitize_textarea_field()` | Multi-line text | String | Textareas, descriptions, multi-line content |
| `sanitize_email()` | Email address | String or empty | Email inputs, returns valid email or empty |
| `sanitize_url()` | URL | String | URL inputs, removes invalid characters |
| `esc_url_raw()` | URL (strict) | String | URL storage, validates scheme, no encoding |
| `sanitize_file_name()` | Filename | String | File uploads, removes special chars |
| `sanitize_key()` | Option names, keys | String | Option keys, meta keys, lowercase alphanumeric |
| `sanitize_title()` | Slug-friendly text | String | Post slugs, URL-safe strings |
| `sanitize_html_class()` | CSS class name | String | CSS class inputs, alphanumeric + hyphen/underscore |
| `absint()` | Integer (positive) | Integer | IDs, counts, non-negative numbers |
| `intval()` | Integer (any) | Integer | Signed integers |
| `floatval()` | Float/decimal | Float | Prices, percentages, decimals |
| `wp_unslash()` | Any POST/GET/COOKIE | String/Array | **ALWAYS call first** to remove magic quotes |
| `wp_kses()` | HTML with allowed tags | String | Rich text with specific HTML tags allowed |
| `wp_kses_post()` | Post content HTML | String | Trusted user content (editors/admins) |

## Sanitization vs Validation

**Sanitization** transforms input to make it safe, removing or encoding dangerous content.
**Validation** checks if input meets requirements and rejects invalid input.

Use **sanitization** when you want to accept input and make it safe.
Use **validation** when input must meet strict requirements or be rejected.

```php
// ❌ BAD: Validation when sanitization needed
$title = $_POST['title'];
if ( preg_match( '/^[a-zA-Z0-9 ]+$/', $title ) ) { // Too strict! Rejects many valid titles
    update_post_meta( $post_id, 'title', $title );
} else {
    wp_die( 'Invalid title' );
}

// ✅ GOOD: Sanitization accepts and cleans input
$title = sanitize_text_field( wp_unslash( $_POST['title'] ) );
update_post_meta( $post_id, 'title', $title ); // Stores safe version

// ❌ BAD: Sanitization when validation needed
$email = sanitize_email( $_POST['email'] ); // Returns empty if invalid
if ( empty( $email ) ) {
    // Too late! Lost original input for error message
    wp_die( 'Invalid email' );
}

// ✅ GOOD: Validation first, then sanitization
$email_raw = isset( $_POST['email'] ) ? wp_unslash( $_POST['email'] ) : '';
if ( ! is_email( $email_raw ) ) {
    wp_die( 'Please enter a valid email address: ' . esc_html( $email_raw ) );
}
$email = sanitize_email( $email_raw );
update_user_meta( $user_id, 'email', $email );
```

### When to Use Each

**Use sanitization for:**
- User-generated content (titles, descriptions, bios)
- Optional fields (can be empty or contain various characters)
- Content that should be cleaned and accepted

**Use validation for:**
- Required fields
- Specific formats (email, phone, date)
- Business logic constraints (stock > 0, price > minimum)
- Access control (checking permissions before allowing action)

```php
// Sanitization example: Accept and clean user bio
$bio = sanitize_textarea_field( wp_unslash( $_POST['bio'] ) );
update_user_meta( $user_id, 'bio', $bio ); // Always succeeds

// Validation example: Require valid email
$email = isset( $_POST['email'] ) ? wp_unslash( $_POST['email'] ) : '';
if ( ! is_email( $email ) ) {
    return new WP_Error( 'invalid_email', 'Please provide a valid email' );
}
update_user_meta( $user_id, 'contact_email', sanitize_email( $email ) );
```

## Type-Specific Sanitization Functions

### sanitize_text_field() — Single-Line Text

Use for text inputs, titles, names, and any single-line text field. Strips tags, line breaks, octets, and extra whitespace.

```php
// ❌ BAD: Using raw $_POST data
$name = $_POST['name'];
update_user_meta( $user_id, 'display_name', $name ); // Stores HTML tags, scripts!

// ❌ BAD: Forgetting wp_unslash()
$name = sanitize_text_field( $_POST['name'] ); // Magic quotes still present!
update_user_meta( $user_id, 'display_name', $name );

// ✅ GOOD: wp_unslash() + sanitize_text_field()
$name = sanitize_text_field( wp_unslash( $_POST['name'] ) );
update_user_meta( $user_id, 'display_name', $name );

// ✅ GOOD: With empty check
$title = isset( $_POST['title'] ) ? sanitize_text_field( wp_unslash( $_POST['title'] ) ) : '';
if ( ! empty( $title ) ) {
    update_post_meta( $post_id, 'custom_title', $title );
}

// ✅ GOOD: Array of text fields
$fields = isset( $_POST['fields'] ) ? (array) $_POST['fields'] : array();
$fields = array_map( function( $field ) {
    return sanitize_text_field( wp_unslash( $field ) );
}, $fields );
update_option( 'prefix_fields', $fields );
```

**What it does:**
- Removes all HTML tags
- Removes line breaks (`\n`, `\r`)
- Removes octets (invalid UTF-8)
- Removes extra whitespace
- Trims leading/trailing spaces

**Use for:**
- Text inputs: `<input type="text">`
- Post titles
- User names
- Product names
- Short descriptions

### sanitize_textarea_field() — Multi-Line Text

Use for textarea content. Like `sanitize_text_field()` but preserves line breaks.

```php
// ❌ BAD: Using sanitize_text_field() on textarea
$description = sanitize_text_field( wp_unslash( $_POST['description'] ) );
// Line breaks removed! Multi-paragraph text becomes single line.

// ✅ GOOD: Use sanitize_textarea_field()
$description = sanitize_textarea_field( wp_unslash( $_POST['description'] ) );
update_post_meta( $post_id, 'description', $description );

// ✅ GOOD: User bio with line breaks
$bio = isset( $_POST['bio'] ) ? sanitize_textarea_field( wp_unslash( $_POST['bio'] ) ) : '';
update_user_meta( $user_id, 'bio', $bio );

// ✅ GOOD: Admin notes field
$notes = sanitize_textarea_field( wp_unslash( $_POST['admin_notes'] ) );
update_option( 'prefix_admin_notes', $notes );
```

**What it does:**
- Removes all HTML tags
- Preserves line breaks (`\n`, `\r`)
- Removes octets
- Trims leading/trailing spaces

**Use for:**
- Textareas: `<textarea>`
- Multi-line descriptions
- User bios
- Admin notes
- Any text that needs line breaks

### sanitize_email() — Email Addresses

Use for email inputs. Returns valid email or empty string.

```php
// ❌ BAD: Storing unvalidated email
$email = $_POST['email'];
update_user_meta( $user_id, 'contact_email', $email ); // Could be invalid!

// ❌ BAD: Not checking return value
$email = sanitize_email( $_POST['email'] ); // Returns empty if invalid
update_user_meta( $user_id, 'contact_email', $email ); // Stores empty string!

// ✅ GOOD: Validate before storing
$email = isset( $_POST['email'] ) ? sanitize_email( wp_unslash( $_POST['email'] ) ) : '';
if ( empty( $email ) || ! is_email( $email ) ) {
    return new WP_Error( 'invalid_email', 'Please provide a valid email address' );
}
update_user_meta( $user_id, 'contact_email', $email );

// ✅ GOOD: Optional email field
$email = isset( $_POST['secondary_email'] ) ? sanitize_email( wp_unslash( $_POST['secondary_email'] ) ) : '';
if ( ! empty( $email ) ) {
    // Only store if valid email provided
    update_user_meta( $user_id, 'secondary_email', $email );
}
```

**What it does:**
- Removes all characters except letters, digits, `@._-`
- Returns empty string if result is not a valid email

**Use with:**
- `is_email()` for validation
- Check return value before storing

### sanitize_url() and esc_url_raw() — URLs

Use `esc_url_raw()` for storing URLs (validates scheme, no encoding).
Use `sanitize_url()` as alias (both call `esc_url_raw()`).

```php
// ❌ BAD: Storing unvalidated URL
$website = $_POST['website'];
update_user_meta( $user_id, 'website', $website ); // Could be javascript: URL!

// ❌ BAD: Using esc_url() for storage
$website = esc_url( $_POST['website'] ); // Encoded for output, wrong for storage!
update_user_meta( $user_id, 'website', $website );

// ✅ GOOD: esc_url_raw() for storage
$website = esc_url_raw( wp_unslash( $_POST['website'] ) );
update_user_meta( $user_id, 'website', $website );

// ✅ GOOD: With empty check
$website = isset( $_POST['website'] ) ? esc_url_raw( wp_unslash( $_POST['website'] ) ) : '';
if ( ! empty( $website ) ) {
    update_option( 'prefix_external_url', $website );
}

// ✅ GOOD: Multiple URLs
$social_links = array(
    'twitter'  => esc_url_raw( wp_unslash( $_POST['twitter'] ) ),
    'facebook' => esc_url_raw( wp_unslash( $_POST['facebook'] ) ),
    'linkedin' => esc_url_raw( wp_unslash( $_POST['linkedin'] ) ),
);
update_user_meta( $user_id, 'social_links', $social_links );
```

**What it does:**
- Validates URL structure
- Validates scheme (http, https, ftp, etc.)
- Rejects dangerous schemes (javascript:, data:, vbscript:)
- No encoding (safe for storage)

**Use for:**
- URL inputs
- Social media links
- External website URLs
- Redirect URLs

### sanitize_file_name() — Filenames

Use for file uploads and user-provided filenames. Removes special characters, spaces to dashes.

```php
// ❌ BAD: Using raw filename
$filename = $_FILES['upload']['name'];
$target = WP_CONTENT_DIR . '/uploads/' . $filename; // Path traversal! ../../evil.php

// ❌ BAD: Only basename() without sanitization
$filename = basename( $_FILES['upload']['name'] ); // Still allows special chars
$target = WP_CONTENT_DIR . '/uploads/' . $filename;

// ✅ GOOD: sanitize_file_name()
$filename = sanitize_file_name( $_FILES['upload']['name'] );
$target = WP_CONTENT_DIR . '/uploads/' . $filename;
// Still use wp_handle_upload() for complete security!

// ✅ GOOD: With unique suffix
$filename = sanitize_file_name( $_FILES['upload']['name'] );
$unique_filename = wp_unique_filename( WP_CONTENT_DIR . '/uploads/', $filename );
$target = WP_CONTENT_DIR . '/uploads/' . $unique_filename;

// ✅ GOOD: Custom export filename from user input
$export_name = sanitize_file_name( $_POST['export_name'] );
if ( empty( $export_name ) ) {
    $export_name = 'export';
}
$filename = $export_name . '-' . date( 'Y-m-d' ) . '.csv';
```

**What it does:**
- Removes special characters: `<>:"/\|?*`
- Converts spaces to dashes
- Preserves extension
- Removes path components

**Use for:**
- File upload names
- User-provided export filenames
- Attachment filenames
- Any filename from user input

### sanitize_key() — Option Names and Keys

Use for option names, meta keys, transient keys. Returns lowercase alphanumeric with underscores and dashes.

```php
// ❌ BAD: User input as option name
$setting_name = $_POST['setting_name'];
update_option( $setting_name, $value ); // Could overwrite core options!

// ✅ GOOD: Sanitize + prefix
$setting_name = sanitize_key( $_POST['setting_name'] );
update_option( 'prefix_' . $setting_name, $value );

// ✅ GOOD: Dynamic meta key
$field_key = sanitize_key( $_POST['field_key'] );
$field_value = sanitize_text_field( wp_unslash( $_POST['field_value'] ) );
update_post_meta( $post_id, 'prefix_' . $field_key, $field_value );

// ✅ GOOD: Transient key
$cache_key = sanitize_key( $_GET['query'] );
$cached = get_transient( 'prefix_search_' . $cache_key );
if ( false === $cached ) {
    // Generate and cache
    set_transient( 'prefix_search_' . $cache_key, $results, HOUR_IN_SECONDS );
}
```

**What it does:**
- Converts to lowercase
- Keeps only: `a-z`, `0-9`, `_`, `-`
- Removes all other characters

**Use for:**
- Dynamic option names (with prefix!)
- Meta keys (with prefix!)
- Transient keys
- Array keys
- Slug-like identifiers

### sanitize_title() — URL-Safe Strings

Use for post slugs and URL-safe strings. Converts to lowercase, replaces spaces/special chars with dashes.

```php
// ❌ BAD: Using slug directly from user input
$slug = $_POST['custom_slug'];
wp_update_post( array( 'ID' => $post_id, 'post_name' => $slug ) ); // Could be invalid!

// ✅ GOOD: sanitize_title() for slugs
$slug = sanitize_title( wp_unslash( $_POST['custom_slug'] ) );
wp_update_post( array( 'ID' => $post_id, 'post_name' => $slug ) );

// ✅ GOOD: Generate slug from title
$title = sanitize_text_field( wp_unslash( $_POST['title'] ) );
$slug = sanitize_title( $title );
update_post_meta( $post_id, 'custom_slug', $slug );

// ✅ GOOD: Category slug
$cat_name = sanitize_text_field( wp_unslash( $_POST['category_name'] ) );
$cat_slug = sanitize_title( $cat_name );
wp_insert_term( $cat_name, 'category', array( 'slug' => $cat_slug ) );
```

**What it does:**
- Converts to lowercase
- Replaces spaces with dashes
- Removes special characters
- Removes accents (configurable)
- URL-safe result

**Use for:**
- Post slugs
- Term slugs
- URL segments
- Permalink components

### sanitize_html_class() — CSS Class Names

Use for CSS class inputs. Returns alphanumeric with hyphens, underscores.

```php
// ❌ BAD: User CSS class without sanitization
$class = $_POST['theme_class'];
echo '<div class="' . $class . '">'; // XSS via quote injection!

// ✅ GOOD: sanitize_html_class()
$class = sanitize_html_class( $_POST['theme_class'] );
if ( ! empty( $class ) ) {
    echo '<div class="' . esc_attr( $class ) . '">';
}

// ✅ GOOD: Multiple classes
$classes = isset( $_POST['classes'] ) ? (array) $_POST['classes'] : array();
$classes = array_map( 'sanitize_html_class', $classes );
$classes = array_filter( $classes ); // Remove empty values
$class_string = implode( ' ', $classes );
echo '<div class="' . esc_attr( $class_string ) . '">';

// ✅ GOOD: Fallback class
$theme = sanitize_html_class( $_POST['theme'] );
if ( empty( $theme ) ) {
    $theme = 'default';
}
echo '<body class="theme-' . esc_attr( $theme ) . '">';
```

**What it does:**
- Keeps only: `a-z`, `A-Z`, `0-9`, `_`, `-`
- Removes all other characters
- Returns empty string if result is invalid

**Use for:**
- User-selected CSS classes
- Theme class inputs
- Dynamic class names
- Body/wrapper classes

### absint() — Positive Integers

Use for IDs, counts, and any non-negative integer. Returns absolute value as integer.

```php
// ❌ BAD: Using intval() for IDs
$post_id = intval( $_POST['post_id'] ); // Could be negative!
wp_delete_post( $post_id ); // Negative IDs can cause issues

// ✅ GOOD: Use absint() for IDs
$post_id = absint( $_POST['post_id'] );
if ( $post_id > 0 ) {
    wp_delete_post( $post_id );
}

// ✅ GOOD: User ID
$user_id = absint( $_GET['user_id'] );
$user = get_userdata( $user_id );

// ✅ GOOD: Count/quantity
$quantity = absint( $_POST['quantity'] );
if ( $quantity > 0 ) {
    update_post_meta( $post_id, '_stock', $quantity );
}

// ✅ GOOD: Page number
$paged = isset( $_GET['paged'] ) ? absint( $_GET['paged'] ) : 1;
if ( $paged < 1 ) {
    $paged = 1;
}
```

**What it does:**
- Converts to integer
- Returns absolute value (always positive)
- Returns 0 for non-numeric input

**Use for:**
- Post IDs, user IDs, term IDs
- Page numbers
- Counts and quantities
- Any positive integer

### intval() — Signed Integers

Use for integers that can be negative.

```php
// ❌ BAD: Using absint() for values that can be negative
$offset = absint( $_POST['offset'] ); // Loses negative offsets!

// ✅ GOOD: Use intval() for signed integers
$offset = intval( $_POST['offset'] );
$query_args = array( 'offset' => $offset );

// ✅ GOOD: Temperature (can be negative)
$temperature = intval( $_POST['temperature'] );
update_option( 'prefix_temperature', $temperature );

// ✅ GOOD: Relative position
$position = intval( $_POST['position'] );
update_post_meta( $post_id, 'relative_position', $position );
```

**What it does:**
- Converts to integer
- Preserves sign (negative values allowed)
- Returns 0 for non-numeric input

**Use for:**
- Offsets
- Relative positions
- Temperature/measurements that can be negative
- Signed numeric values

### floatval() — Decimal Numbers

Use for prices, percentages, measurements.

```php
// ❌ BAD: Using intval() for prices
$price = intval( $_POST['price'] ); // $19.99 becomes 19!

// ✅ GOOD: Use floatval() for decimals
$price = floatval( $_POST['price'] );
update_post_meta( $product_id, '_price', $price );

// ✅ GOOD: Percentage
$discount = floatval( $_POST['discount'] );
if ( $discount > 0 && $discount <= 100 ) {
    update_option( 'prefix_discount_percent', $discount );
}

// ✅ GOOD: Weight/measurements
$weight = floatval( $_POST['weight'] );
update_post_meta( $product_id, '_weight', $weight );

// ✅ GOOD: With validation
$rating = floatval( $_POST['rating'] );
if ( $rating < 0 || $rating > 5 ) {
    return new WP_Error( 'invalid_rating', 'Rating must be between 0 and 5' );
}
update_post_meta( $post_id, '_rating', $rating );
```

**What it does:**
- Converts to float
- Preserves decimal places
- Returns 0.0 for non-numeric input

**Use for:**
- Prices
- Percentages
- Weights/dimensions
- Ratings/scores
- Any decimal number

## The wp_unslash() Requirement

WordPress may add slashes to `$_POST`, `$_GET`, `$_COOKIE`, and `$_REQUEST` data (magic quotes behavior). **Always call `wp_unslash()` before sanitizing user input.**

```php
// ❌ BAD: Sanitizing without wp_unslash()
$title = sanitize_text_field( $_POST['title'] );
// If title is: O'Reilly
// Stored as: O\'Reilly (backslash remains!)

// ✅ GOOD: wp_unslash() before sanitizing
$title = sanitize_text_field( wp_unslash( $_POST['title'] ) );
// Stored correctly as: O'Reilly

// ❌ BAD: Multiple fields without wp_unslash()
$data = array(
    'name'  => sanitize_text_field( $_POST['name'] ),
    'email' => sanitize_email( $_POST['email'] ),
    'bio'   => sanitize_textarea_field( $_POST['bio'] ),
);
// All may have unwanted backslashes!

// ✅ GOOD: wp_unslash() entire array first
$post_data = wp_unslash( $_POST );
$data = array(
    'name'  => sanitize_text_field( $post_data['name'] ),
    'email' => sanitize_email( $post_data['email'] ),
    'bio'   => sanitize_textarea_field( $post_data['bio'] ),
);

// ✅ GOOD: Per-field with wp_unslash()
$name = sanitize_text_field( wp_unslash( $_POST['name'] ) );
$email = sanitize_email( wp_unslash( $_POST['email'] ) );
$bio = sanitize_textarea_field( wp_unslash( $_POST['bio'] ) );
```

**Why it's needed:**
- WordPress adds slashes to preserve quotes/backslashes
- Must be removed before sanitizing
- Otherwise stored data contains `\'`, `\"`, `\\`

**When to use:**
- **Always** with `$_POST`, `$_GET`, `$_COOKIE`, `$_REQUEST`
- Before any sanitization function
- Can unslash entire array or individual values

## Sanitizing Arrays and Complex Data

### Array of Text Fields

```php
// ❌ BAD: Storing array without sanitization
$tags = $_POST['tags']; // Array of user input
update_post_meta( $post_id, 'custom_tags', $tags ); // Unsafe!

// ✅ GOOD: array_map with sanitization
$tags = isset( $_POST['tags'] ) ? (array) $_POST['tags'] : array();
$tags = array_map( function( $tag ) {
    return sanitize_text_field( wp_unslash( $tag ) );
}, $tags );
update_post_meta( $post_id, 'custom_tags', $tags );

// ✅ GOOD: Filter empty values
$tags = isset( $_POST['tags'] ) ? (array) $_POST['tags'] : array();
$tags = array_map( 'sanitize_text_field', array_map( 'wp_unslash', $tags ) );
$tags = array_filter( $tags ); // Remove empty strings
update_post_meta( $post_id, 'custom_tags', $tags );
```

### Nested Arrays

```php
// ❌ BAD: Sanitizing only top level
$settings = $_POST['settings']; // Multi-dimensional array
update_option( 'prefix_settings', $settings ); // Nested values unsafe!

// ✅ GOOD: Recursive sanitization
function prefix_sanitize_array( $array ) {
    foreach ( $array as $key => $value ) {
        if ( is_array( $value ) ) {
            $array[ $key ] = prefix_sanitize_array( $value );
        } else {
            $array[ $key ] = sanitize_text_field( wp_unslash( $value ) );
        }
    }
    return $array;
}

$settings = isset( $_POST['settings'] ) ? $_POST['settings'] : array();
$settings = prefix_sanitize_array( $settings );
update_option( 'prefix_settings', $settings );

// ✅ GOOD: Type-specific nested sanitization
$user_profile = array(
    'name'    => sanitize_text_field( wp_unslash( $_POST['name'] ) ),
    'email'   => sanitize_email( wp_unslash( $_POST['email'] ) ),
    'social'  => array(
        'twitter'  => esc_url_raw( wp_unslash( $_POST['twitter'] ) ),
        'facebook' => esc_url_raw( wp_unslash( $_POST['facebook'] ) ),
    ),
);
update_user_meta( $user_id, 'profile', $user_profile );
```

### JSON Data

```php
// ❌ BAD: Storing JSON without validation
$json_data = $_POST['data'];
update_option( 'prefix_json_data', $json_data ); // Could be malformed!

// ✅ GOOD: Decode, sanitize, re-encode
$json_raw = isset( $_POST['data'] ) ? wp_unslash( $_POST['data'] ) : '';
$data = json_decode( $json_raw, true );

if ( json_last_error() !== JSON_ERROR_NONE ) {
    return new WP_Error( 'invalid_json', 'Invalid JSON data' );
}

// Sanitize decoded data
$data = array_map( 'sanitize_text_field', $data );

// Store as JSON
update_option( 'prefix_json_data', wp_json_encode( $data ) );
```

### Checkbox and Select Inputs (Whitelist)

```php
// ❌ BAD: Trusting select value
$status = $_POST['status'];
update_post_meta( $post_id, 'status', $status ); // User can submit any value!

// ✅ GOOD: Whitelist allowed values
$allowed_statuses = array( 'draft', 'pending', 'published', 'archived' );
$status = isset( $_POST['status'] ) ? sanitize_key( $_POST['status'] ) : 'draft';

if ( ! in_array( $status, $allowed_statuses, true ) ) {
    $status = 'draft'; // Fallback to default
}

update_post_meta( $post_id, 'status', $status );

// ✅ GOOD: Checkbox (boolean)
$enabled = isset( $_POST['enabled'] ) && '1' === $_POST['enabled'];
update_option( 'prefix_feature_enabled', $enabled );

// ✅ GOOD: Multiple checkboxes
$selected_items = isset( $_POST['items'] ) ? (array) $_POST['items'] : array();
$allowed_items = array( 'item1', 'item2', 'item3', 'item4' );
$selected_items = array_map( 'sanitize_key', $selected_items );
$selected_items = array_intersect( $selected_items, $allowed_items ); // Only allowed values
update_option( 'prefix_selected_items', $selected_items );
```

## When NOT to Sanitize

### Post Content (Use wp_kses instead)

```php
// ❌ BAD: sanitize_textarea_field() on post content
$content = sanitize_textarea_field( wp_unslash( $_POST['post_content'] ) );
wp_update_post( array( 'ID' => $post_id, 'post_content' => $content ) );
// Removes ALL HTML! <p>, <strong>, <img> all stripped!

// ✅ GOOD: Use wp_kses_post() for post content
if ( current_user_can( 'edit_posts' ) ) {
    $content = wp_kses_post( wp_unslash( $_POST['post_content'] ) );
    wp_update_post( array( 'ID' => $post_id, 'post_content' => $content ) );
}
```

### Binary Data

```php
// ❌ BAD: Sanitizing image data
$image_data = sanitize_text_field( $_POST['image_base64'] ); // Corrupts binary!

// ✅ GOOD: Validate format, don't sanitize content
$image_data = isset( $_POST['image_base64'] ) ? $_POST['image_base64'] : '';
if ( ! preg_match( '/^data:image\/(png|jpg|jpeg|gif);base64,/', $image_data ) ) {
    return new WP_Error( 'invalid_image', 'Invalid image format' );
}
// Store as-is or decode and validate
```

### Already Sanitized WordPress Function Output

```php
// ❌ BAD: Re-sanitizing WordPress function output
$user_email = get_the_author_meta( 'user_email' ); // Already sanitized
$user_email = sanitize_email( $user_email ); // Unnecessary

// ✅ GOOD: Trust WordPress function output
$user_email = get_the_author_meta( 'user_email' );
// Use directly (escape on output if needed)
```

## Complete Input Handling Example

```php
/**
 * Handle form submission with proper sanitization
 */
function prefix_handle_profile_update() {
    // 1. Verify nonce
    if ( ! isset( $_POST['_nonce'] ) || ! wp_verify_nonce( $_POST['_nonce'], 'update_profile' ) ) {
        wp_die( 'Invalid nonce' );
    }

    // 2. Check capability
    if ( ! current_user_can( 'edit_users' ) ) {
        wp_die( 'Insufficient permissions' );
    }

    // 3. Get user ID
    $user_id = absint( $_POST['user_id'] );
    if ( $user_id <= 0 ) {
        return new WP_Error( 'invalid_user', 'Invalid user ID' );
    }

    // 4. Sanitize text fields
    $display_name = sanitize_text_field( wp_unslash( $_POST['display_name'] ) );
    $bio = sanitize_textarea_field( wp_unslash( $_POST['bio'] ) );

    // 5. Validate and sanitize email
    $email = isset( $_POST['email'] ) ? wp_unslash( $_POST['email'] ) : '';
    if ( ! is_email( $email ) ) {
        return new WP_Error( 'invalid_email', 'Please provide a valid email' );
    }
    $email = sanitize_email( $email );

    // 6. Sanitize URL
    $website = esc_url_raw( wp_unslash( $_POST['website'] ) );

    // 7. Whitelist select value
    $allowed_roles = array( 'subscriber', 'contributor', 'author' );
    $role = sanitize_key( $_POST['role'] );
    if ( ! in_array( $role, $allowed_roles, true ) ) {
        $role = 'subscriber';
    }

    // 8. Boolean checkbox
    $newsletter = isset( $_POST['newsletter'] ) && '1' === $_POST['newsletter'];

    // 9. Array of sanitized values
    $interests = isset( $_POST['interests'] ) ? (array) $_POST['interests'] : array();
    $interests = array_map( 'sanitize_text_field', array_map( 'wp_unslash', $interests ) );
    $interests = array_filter( $interests );

    // 10. Update user data
    wp_update_user( array(
        'ID'           => $user_id,
        'display_name' => $display_name,
        'user_email'   => $email,
        'user_url'     => $website,
        'role'         => $role,
    ) );

    // 11. Update user meta
    update_user_meta( $user_id, 'bio', $bio );
    update_user_meta( $user_id, 'newsletter_opt_in', $newsletter );
    update_user_meta( $user_id, 'interests', $interests );

    return true;
}
```

## Summary

Key principles for WordPress input sanitization:

1. **Always wp_unslash() first** — Before any sanitization
2. **Use type-specific functions** — `sanitize_text_field()`, `sanitize_email()`, `absint()`, etc.
3. **Sanitize on input** — Make data safe when receiving it
4. **Escape on output** — See escaping-guide.md
5. **Validate when needed** — Check requirements before sanitizing
6. **Whitelist for enums** — Don't trust select/radio values
7. **Recursively sanitize arrays** — Don't forget nested values
8. **Don't over-sanitize** — Post content needs `wp_kses_post()`, not `sanitize_text_field()`

Cross-references:
- Output escaping → escaping-guide.md
- Security vulnerabilities → vulnerability-patterns.md
- Complete security review → SKILL.md
