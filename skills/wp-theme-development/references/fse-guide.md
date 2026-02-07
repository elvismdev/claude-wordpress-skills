# WordPress Full Site Editing (FSE) Complete Guide

This reference provides comprehensive coverage of Full Site Editing features for WordPress block themes (WordPress 6.6+) — the system that enables users to customize entire sites through the block editor interface. Covers global styles, style variations, block patterns in themes, navigation configuration, font management, layout system, and Site Editor workflows.

## Quick Reference Table

### FSE Features Overview

| Feature | Required Files | theme.json Section | WP Version | User Access |
|---------|---------------|--------------------|-----------| ------------|
| **Global Styles** | theme.json | settings, styles | 5.9+ | Appearance > Editor > Styles |
| **Style Variations** | styles/*.json | N/A (standalone files) | 6.0+ | Styles panel > Browse styles |
| **Block Patterns** | patterns/*.php | N/A (auto-registered) | 6.3+ | Block inserter > Patterns tab |
| **Navigation** | templates/*.html with wp:navigation | N/A | 5.9+ | Appearance > Editor > Navigation |
| **Template Editing** | templates/*.html | customTemplates (optional) | 5.9+ | Appearance > Editor > Templates |
| **Template Parts** | parts/*.html | templateParts | 5.9+ | Appearance > Editor > Patterns > Template Parts |
| **Font Management** | theme.json, assets/fonts/* | settings.typography.fontFamilies | 6.0+ | Styles panel > Typography |
| **Custom CSS** | theme.json | styles.css | 6.2+ | Styles panel > Additional CSS |

## Full Site Editing Overview

### What is FSE?

Full Site Editing enables users to customize their entire WordPress site through the block editor, without writing code or touching theme files.

**FSE includes:**

- **Site Editor:** Visual interface for editing templates, template parts, patterns, and styles
- **Global Styles:** Centralized system for site-wide colors, typography, spacing
- **Style Variations:** One-click theme style switchers
- **Template Editing:** Block-based template creation and modification
- **Navigation Block:** Visual menu editor replacing classic Appearance > Menus
- **Block Patterns:** Pre-designed block layouts users can insert

**Key insight:** FSE shifts control from theme developers (code) to site users (visual editor).

### Site Editor Interface

**Access:** Appearance > Editor

**Main sections:**

1. **Templates** — Edit HTML templates (single.html, page.html, etc.)
2. **Template Parts** — Edit reusable parts (header.html, footer.html)
3. **Patterns** — Create custom block patterns
4. **Styles** — Global styles and style variations
5. **Navigation** — Edit navigation menus
6. **Pages** — Quick access to page editing

**Canvas vs List View:**

- **Canvas:** Visual template editing
- **List View:** Block hierarchy tree view

### Relationship to theme.json

```
theme.json (developer) → Global Styles UI (user) → Customizations (database)
```

**Flow:**

1. Developer defines defaults in theme.json
2. User customizes via Site Editor > Styles
3. WordPress saves user changes to database
4. Frontend applies: theme.json defaults + user customizations

**User customizations stored in:**
- Custom Post Type: `wp_global_styles`
- Merged with theme.json at runtime

## Global Styles System

### How theme.json Maps to Global Styles UI

**theme.json:**

```json
{
	"version": 3,
	"settings": {
		"color": {
			"palette": [
				{"slug": "primary", "color": "#0073aa", "name": "Primary"},
				{"slug": "secondary", "color": "#005177", "name": "Secondary"}
			]
		},
		"typography": {
			"fontSizes": [
				{"slug": "small", "size": "0.875rem", "name": "Small"},
				{"slug": "large", "size": "1.5rem", "name": "Large"}
			]
		}
	},
	"styles": {
		"color": {
			"background": "var(--wp--preset--color--background)",
			"text": "var(--wp--preset--color--foreground)"
		},
		"typography": {
			"fontSize": "var(--wp--preset--font-size--medium)"
		}
	}
}
```

**Global Styles UI shows:**

- **Colors panel:** Primary, Secondary (from palette)
- **Typography panel:** Small, Large size options (from fontSizes)
- **Preview:** Background and text color applied (from styles)

**User changes:**

When user changes background color to "Primary" via UI, WordPress saves:

```json
{
	"version": 3,
	"styles": {
		"color": {
			"background": "var(--wp--preset--color--primary)"
		}
	}
}
```

This merges with theme.json, overriding the default background.

### Style Precedence

**Highest to lowest priority:**

1. **Inline styles** (style attribute on blocks) — User applied via block settings
2. **User Global Styles** — Saved in wp_global_styles post type
3. **Theme styles** — From theme.json styles section
4. **Block defaults** — WordPress core block defaults

**Example:**

```
User sets heading color to red via Styles panel (Global Styles)
→ Overrides theme.json styles.elements.heading.color
→ But user can still override per-block via block settings sidebar
```

### Global Styles UI Organization

**Root styles (body element):**

- Background color
- Text color
- Link color
- Font family
- Font size
- Line height
- Padding

**Element styles:**

- Links (:hover, :focus states)
- Headings
- Buttons
- Captions

**Block styles:**

Per-block customization for any registered block.

**Example UI path:**

1. Appearance > Editor > Styles
2. Click "Typography" in sidebar
3. Select "Heading" element
4. Change font size to "Large"

Result: All headings site-wide use Large font size.

## Style Variations

### Creating Style Variations

**Directory:** `styles/`

**File naming:** `{variation-name}.json`

**Example:** `styles/dark.json`

```json
{
	"$schema": "https://schemas.wp.org/trunk/theme.json",
	"version": 3,
	"title": "Dark Mode",
	"settings": {
		"color": {
			"palette": [
				{
					"slug": "foreground",
					"color": "#ffffff",
					"name": "Foreground"
				},
				{
					"slug": "background",
					"color": "#1a1a1a",
					"name": "Background"
				},
				{
					"slug": "primary",
					"color": "#3dadff",
					"name": "Primary"
				},
				{
					"slug": "secondary",
					"color": "#69a3d6",
					"name": "Secondary"
				}
			]
		}
	},
	"styles": {
		"color": {
			"background": "var(--wp--preset--color--background)",
			"text": "var(--wp--preset--color--foreground)"
		},
		"elements": {
			"link": {
				"color": {
					"text": "var(--wp--preset--color--primary)"
				}
			}
		}
	}
}
```

**Required fields:**

- `version` — Must match main theme.json version
- `title` — Display name in UI (optional, uses filename if omitted)

**Optional fields:**

- `$schema` — IDE validation (recommended)
- `settings` — Can override theme settings
- `styles` — Can override theme styles

### What Can Be Overridden in Variations

**Can override:**

- `settings.color.palette` — Different color schemes
- `settings.typography.fontFamilies` — Different font families
- `settings.typography.fontSizes` — Different size scales
- `styles` — All style properties (colors, typography, spacing, etc.)

**Cannot override:**

- `settings.layout.contentSize` — Content width locked to main theme.json
- `settings.layout.wideSize` — Wide width locked to main theme.json
- `customTemplates` — Template definitions locked
- `templateParts` — Template part definitions locked

**Why layout locked:** Prevents style variations from breaking site structure.

### Style Variation Naming

**File name becomes variation slug:**

```
styles/dark.json → "Dark" variation (auto-generated title)
styles/minimal-typography.json → "Minimal Typography"
```

**Override with title property:**

```json
{
	"title": "Dark Mode with Blue Accents"
}
```

File: `styles/dark.json`
UI shows: "Dark Mode with Blue Accents"

### User Selecting Variations

**Path:** Appearance > Editor > Styles > Browse Styles

**What happens:**

1. User clicks variation preview
2. WordPress applies variation settings + styles
3. Saved to wp_global_styles post
4. User can further customize from that starting point

**Variation as starting point:** Variations aren't "locked" — users can customize after selecting.

## Block Patterns in Themes

### Pattern File Structure

**Directory:** `patterns/`

**File naming:** `{pattern-slug}.php`

**Example:** `patterns/hero.php`

```php
<?php
/**
 * Title: Hero Section
 * Slug: mytheme/hero
 * Categories: featured, banner
 * Keywords: hero, banner, header, cover
 * Block Types: core/cover
 * Viewport Width: 1400
 * Description: Full-width hero section with heading and call-to-action
 * Inserter: yes
 */
?>
<!-- wp:cover {"url":"<?php echo esc_url( get_theme_file_uri( 'assets/images/hero.jpg' ) ); ?>","dimRatio":50,"align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}}}} -->
<div class="wp-block-cover alignfull">
	<span aria-hidden="true" class="wp-block-cover__background has-background-dim"></span>
	<img class="wp-block-cover__image-background" alt="" src="<?php echo esc_url( get_theme_file_uri( 'assets/images/hero.jpg' ) ); ?>" data-object-fit="cover" />

	<div class="wp-block-cover__inner-container">
		<!-- wp:heading {"textAlign":"center","level":1,"style":{"typography":{"fontSize":"3rem"}}} -->
		<h1 class="has-text-align-center" style="font-size:3rem"><?php echo esc_html_x( 'Welcome to Our Site', 'Pattern placeholder text', 'mytheme' ); ?></h1>
		<!-- /wp:heading -->

		<!-- wp:paragraph {"align":"center"} -->
		<p class="has-text-align-center"><?php echo esc_html_x( 'Discover amazing content and services', 'Pattern placeholder text', 'mytheme' ); ?></p>
		<!-- /wp:paragraph -->

		<!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
		<div class="wp-block-buttons">
			<!-- wp:button {"className":"is-style-fill"} -->
			<div class="wp-block-button is-style-fill">
				<a class="wp-block-button__link wp-element-button"><?php echo esc_html_x( 'Get Started', 'Pattern placeholder text', 'mytheme' ); ?></a>
			</div>
			<!-- /wp:button -->
		</div>
		<!-- /wp:buttons -->
	</div>
</div>
<!-- /wp:cover -->
```

### Pattern Header Fields

| Field | Required | Description | Example |
|-------|----------|-------------|---------|
| `Title` | Yes | Human-readable pattern name | `Hero Section` |
| `Slug` | Yes | Unique identifier (namespace/slug) | `mytheme/hero` |
| `Categories` | No | Pattern categories (comma-separated) | `featured, banner` |
| `Keywords` | No | Search keywords | `hero, banner, header` |
| `Block Types` | No | Related block types | `core/cover, core/group` |
| `Viewport Width` | No | Preview width in pixels | `1400` |
| `Description` | No | Pattern description | `Full-width hero section` |
| `Inserter` | No | Show in inserter (yes/no) | `yes` (default) |

**Slug format:** Must be namespaced `themename/pattern-name` to avoid conflicts.

### Pattern Categories

**Built-in categories:**

- `featured` — Featured patterns
- `text` — Text patterns
- `gallery` — Gallery patterns
- `call-to-action` — CTA patterns
- `banner` — Banner patterns
- `header` — Header patterns
- `footer` — Footer patterns
- `query` — Query loop patterns

**Custom categories:**

Register in functions.php:

```php
add_action( 'init', 'mytheme_register_pattern_categories' );

function mytheme_register_pattern_categories() {
	register_block_pattern_category(
		'mytheme-portfolio',
		array( 'label' => __( 'Portfolio Sections', 'mytheme' ) )
	);
}
```

Then use in pattern:

```php
/**
 * Categories: mytheme-portfolio
 */
```

### Auto-Registration

**How it works:**

1. WordPress scans `patterns/` directory on theme activation
2. Reads PHP file headers
3. Auto-registers patterns with `register_block_pattern()`
4. No manual registration code needed

**Pattern availability:**

- Block inserter > Patterns tab
- Page/post editor
- Site Editor template editing

### Dynamic Pattern Content

**Use PHP for dynamic values:**

```php
<?php
/**
 * Title: Recent Posts Grid
 * Slug: mytheme/recent-posts
 */

$recent_posts = get_posts( array(
	'numberposts' => 3,
	'post_status' => 'publish',
) );
?>
<!-- wp:group {"layout":{"type":"constrained"}} -->
<div class="wp-block-group">
	<!-- wp:heading -->
	<h2><?php esc_html_e( 'Recent Posts', 'mytheme' ); ?></h2>
	<!-- /wp:heading -->

	<?php foreach ( $recent_posts as $post ) : ?>
		<!-- wp:heading {"level":3} -->
		<h3><?php echo esc_html( get_the_title( $post ) ); ?></h3>
		<!-- /wp:heading -->
	<?php endforeach; ?>
</div>
<!-- /wp:group -->
```

**When to use:**

- Theme-specific asset URLs (`get_theme_file_uri()`)
- Translated text (`esc_html__()`, `esc_html_x()`)
- Dynamic queries (recent posts, featured content)

**CRITICAL: Escape all output**

```php
// BAD: Unescaped
<?php echo get_the_title(); ?>

// GOOD: Escaped
<?php echo esc_html( get_the_title() ); ?>
```

**Cross-reference:** See wp-security-review skill for complete escaping guide.

## Navigation Configuration

### Navigation Block in Templates

**Usage in templates:**

```html
<!-- wp:navigation {"layout":{"type":"flex","orientation":"horizontal"}} /-->
```

**With styling:**

```html
<!-- wp:navigation {"layout":{"type":"flex","orientation":"horizontal"},"style":{"spacing":{"blockGap":"var:preset|spacing|40"}}} /-->
```

### How Users Create Menus

**Classic themes:** Appearance > Menus (separate interface)

**Block themes:** Inline editing in Site Editor or templates

**User workflow:**

1. Insert Navigation block in template/template part
2. Click "+ Add link" in block
3. Select pages, custom links, categories, etc.
4. Block saves menu automatically
5. Menu reusable across templates

**Menu storage:**

- Custom Post Type: `wp_navigation`
- Each menu is a post with navigation items as blocks

### Navigation Block Fallback

**If no menu exists:**

Navigation block shows:

1. **Page List block** (lists all published pages)
2. **Edit prompt** (for logged-in users with permission)

**Customize fallback in theme.json:**

Cannot be configured in theme.json. Fallback is automatic.

**Best practice:** Create default navigation in Site Editor during theme development, export with Create Block Theme plugin.

### Navigation Block Configuration

**Available settings:**

- **Orientation:** Horizontal, vertical
- **Layout:** Flex, justification options
- **Styling:** Background color, text color, spacing
- **Overlay:** Mobile menu overlay settings
- **Responsive:** Mobile menu breakpoint

**Block supports in theme.json:**

```json
{
	"settings": {
		"blocks": {
			"core/navigation": {
				"color": {
					"background": true,
					"text": true
				},
				"spacing": {
					"padding": true,
					"margin": true
				}
			}
		}
	}
}
```

## Font Management

### Local Font Hosting via theme.json

**Directory:** `assets/fonts/`

**theme.json configuration:**

```json
{
	"settings": {
		"typography": {
			"fontFamilies": [
				{
					"fontFamily": "\"Inter\", -apple-system, BlinkMacSystemFont, \"Segoe UI\", sans-serif",
					"name": "Inter",
					"slug": "inter",
					"fontFace": [
						{
							"fontFamily": "Inter",
							"fontWeight": "400",
							"fontStyle": "normal",
							"src": ["file:./assets/fonts/inter-regular.woff2"]
						},
						{
							"fontFamily": "Inter",
							"fontWeight": "700",
							"fontStyle": "normal",
							"src": ["file:./assets/fonts/inter-bold.woff2"]
						},
						{
							"fontFamily": "Inter",
							"fontWeight": "400",
							"fontStyle": "italic",
							"src": ["file:./assets/fonts/inter-italic.woff2"]
						}
					]
				}
			]
		}
	}
}
```

**File structure:**

```
assets/fonts/
├── inter-regular.woff2
├── inter-bold.woff2
└── inter-italic.woff2
```

**Why local hosting:**

- **Privacy:** GDPR compliance (no third-party requests)
- **Performance:** Fewer HTTP requests, no external DNS lookups
- **Reliability:** Works offline, no CDN downtime
- **Control:** Version locking, no unexpected font updates

### Font File Formats

**Recommended:** `.woff2` (Web Open Font Format 2)

- 30-50% smaller than .ttf
- Supported by all modern browsers
- Best compression

**Fallback:** `.woff` for older browser support

**Don't use:** `.ttf`, `.otf` (too large for web)

### Font Face Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `fontFamily` | string | Yes | Font family name (must match fontFamilies.fontFamily) |
| `fontWeight` | string | No | Font weight (100-900, or "normal"/"bold") |
| `fontStyle` | string | No | Font style ("normal", "italic", "oblique") |
| `src` | array | Yes | Font file paths (file:./path/to/font.woff2) |

**Multiple font weights/styles:**

```json
{
	"fontFace": [
		{
			"fontFamily": "Inter",
			"fontWeight": "100 900",
			"fontStyle": "normal",
			"src": ["file:./assets/fonts/inter-variable.woff2"]
		}
	]
}
```

Variable font with weight range 100-900 in single file.

### Generated CSS

**WordPress generates @font-face rules:**

```css
@font-face {
	font-family: "Inter";
	font-weight: 400;
	font-style: normal;
	src: url('/wp-content/themes/mytheme/assets/fonts/inter-regular.woff2') format('woff2');
}

@font-face {
	font-family: "Inter";
	font-weight: 700;
	font-style: normal;
	src: url('/wp-content/themes/mytheme/assets/fonts/inter-bold.woff2') format('woff2');
}
```

**And CSS variables:**

```css
--wp--preset--font-family--inter: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
```

### Font Licensing

**WordPress.org submission requires:**

- GPL-compatible font license
- License information in LICENSE.txt
- Font source attribution

**Common GPL-compatible fonts:**

- Open Sans
- Roboto
- Lato
- Source Sans Pro
- Inter
- Public Sans

**Non-GPL fonts:** Cannot be included in WordPress.org theme submission.

## Layout System

### contentSize and wideSize

**In theme.json:**

```json
{
	"settings": {
		"layout": {
			"contentSize": "640px",
			"wideSize": "1200px"
		}
	}
}
```

**What they control:**

- **contentSize:** Max width for normal content blocks
- **wideSize:** Max width for wide-aligned blocks
- **Full width:** No max width, extends to viewport edges

**Visual representation:**

```
┌─────────────────────────────────────────────┐
│                                             │ ← Viewport
│   ┌─────────────────────────────────────┐   │
│   │                                     │   │ ← wideSize (1200px)
│   │     ┌───────────────────────┐       │   │
│   │     │                       │       │   │ ← contentSize (640px)
│   │     │   Normal content      │       │   │
│   │     └───────────────────────┘       │   │
│   │                                     │   │
│   │  ┌──────────────────────────────┐   │   │
│   │  │  Wide-aligned block          │   │   │
│   │  └──────────────────────────────┘   │   │
│   └─────────────────────────────────────┘   │
├─────────────────────────────────────────────┤
│  Full-width block                           │
└─────────────────────────────────────────────┘
```

### Block Alignment Classes

| Class | Width | Usage |
|-------|-------|-------|
| (no alignment) | contentSize | Default for most blocks |
| `alignwide` | wideSize | Wide images, galleries |
| `alignfull` | 100vw | Covers, full-width sections |

**Applying alignment:**

In block attributes:

```html
<!-- wp:image {"align":"wide"} -->
<!-- Renders with class="alignwide" -->

<!-- wp:cover {"align":"full"} -->
<!-- Renders with class="alignfull" -->
```

### Root Padding and Alignments

**See theme-json-guide.md for complete useRootPaddingAwareAlignments coverage.**

**Quick summary:**

```json
{
	"settings": {
		"layout": {
			"contentSize": "640px",
			"wideSize": "1200px"
		},
		"useRootPaddingAwareAlignments": true
	},
	"styles": {
		"spacing": {
			"padding": {
				"top": "2rem",
				"right": "2rem",
				"bottom": "2rem",
				"left": "2rem"
			}
		}
	}
}
```

**Result:** Full-width blocks extend to viewport edges via negative margins.

## Custom CSS in theme.json

### styles.css Property (WordPress 6.2+)

**Add custom CSS in theme.json:**

```json
{
	"styles": {
		"css": ".my-custom-class { background-color: red; }"
	}
}
```

**WordPress generates:**

```css
.wp-site-blocks .my-custom-class {
	background-color: red;
}
```

**Use cases:**

- Fine-tuned selectors not possible via theme.json
- Pseudo-elements (::before, ::after)
- Complex CSS (calc, clamp, grid)

**Limitations:**

- No Sass/Less preprocessing
- No CSS imports
- Scoped to .wp-site-blocks context

**Alternative:** Enqueue separate CSS file in functions.php for complex styles.

## Block Locking in Templates

### Lock Attribute

**Prevent block removal/movement:**

```html
<!-- wp:group {"lock":{"move":true,"remove":true}} -->
<div class="wp-block-group">
	<!-- wp:site-title /-->
</div>
<!-- /wp:group -->
```

**Lock levels:**

- `"move": true` — Block cannot be moved
- `"remove": true` — Block cannot be deleted

**Use cases:**

- Lock site branding in header
- Lock footer copyright notice
- Prevent users from breaking layout

**User experience:**

Locked blocks show lock icon, drag/delete options disabled.

### templateLock Attribute

**Lock inner blocks:**

```html
<!-- wp:group {"templateLock":"all"} -->
<div class="wp-block-group">
	<!-- Inner blocks locked -->
</div>
<!-- /wp:group -->
```

**Lock levels:**

- `"all"` — Cannot add, remove, or move inner blocks
- `"insert"` — Cannot add new blocks (can move/remove existing)
- `false` — No restrictions (default)

**Use cases:**

- Structured layouts (hero sections with fixed format)
- Template consistency

**Warning:** Excessive locking reduces user flexibility. Use sparingly.

## WordPress.org Theme Submission

### FSE-Specific Requirements

**Required for block theme submission:**

1. **Valid theme.json v3**
2. **At least templates/index.html**
3. **No hardcoded inline styles in templates**
4. **GPL-compatible fonts with license documentation**
5. **Accessible color contrast** (WCAG AA)

**Common rejection reasons:**

- Hardcoded hex values in templates
- Missing font license information
- Non-GPL-compatible fonts
- Inline styles preventing user customization
- Missing required template files

**Best practices for approval:**

- Use theme.json for all colors/typography
- Reference presets via CSS variables
- Include comprehensive readme.txt
- Document all third-party resources
- Test with Theme Check plugin

### Testing Before Submission

**Required testing:**

1. **Theme Unit Test data** — Import sample content
2. **Theme Check plugin** — Automated validation
3. **Multiple style variations** — Test all variations work
4. **Site Editor workflows** — Verify templates editable
5. **Accessibility** — Color contrast, keyboard navigation

**Theme Check command:**

```bash
wp theme check mytheme
```

Or via WordPress admin: Tools > Theme Check

## Anti-Patterns

### BAD: Hardcoded Styles in Templates

```html
<!-- BAD: Defeats FSE purpose -->
<div style="background-color: #333; font-size: 18px;">
	<!-- wp:post-title /-->
</div>
```

**GOOD: Use theme.json presets**

```html
<!-- GOOD: User-customizable via Site Editor -->
<!-- wp:group {"style":{"color":{"background":"var(--wp--preset--color--primary)"},"typography":{"fontSize":"var(--wp--preset--font-size--medium)"}}} -->
<div class="wp-block-group">
	<!-- wp:post-title /-->
</div>
<!-- /wp:group -->
```

### BAD: Google Fonts CDN

```json
// BAD: Privacy concern, external dependency
{
	"fontFace": [
		{
			"src": ["https://fonts.googleapis.com/css2?family=Inter"]
		}
	]
}
```

**GOOD: Local font hosting**

```json
// GOOD: Privacy-compliant, faster, offline-capable
{
	"fontFace": [
		{
			"fontFamily": "Inter",
			"fontWeight": "400",
			"fontStyle": "normal",
			"src": ["file:./assets/fonts/inter-regular.woff2"]
		}
	]
}
```

### BAD: Pattern Without Namespace

```php
<?php
/**
 * Slug: hero
 */
?>
```

**GOOD: Namespaced slug**

```php
<?php
/**
 * Slug: mytheme/hero
 */
?>
```

## Cross-References

- **theme.json v3 complete reference:** See theme-json-guide.md
- **Template structure and hierarchy:** See template-patterns.md
- **Classic to block theme migration:** See classic-to-block-guide.md
- **Block markup in templates:** See wp-block-development skill
- **Template security and escaping:** See wp-security-review skill
