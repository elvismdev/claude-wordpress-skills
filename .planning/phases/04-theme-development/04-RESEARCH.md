# Phase 4: Theme Development - Research

**Researched:** 2026-02-06
**Domain:** WordPress block theme development, Full Site Editing (FSE), theme.json v3
**Confidence:** HIGH

## Summary

Phase 4 research focused on WordPress block theme development for WordPress 6.6+, with theme.json v3 as the primary configuration system. The research reveals that WordPress theme development has undergone a fundamental paradigm shift: block themes use HTML block templates instead of PHP templates, theme.json v3 replaces most `add_theme_support()` calls, and Full Site Editing (FSE) enables site-wide customization through the block editor. Classic themes remain relevant for migration guidance, and hybrid themes provide an incremental adoption path.

**Key findings:**
- theme.json v3 introduces breaking changes around `defaultFontSizes` and `defaultSpacingSizes` settings (both default to `true` in v3, reversing v2 behavior)
- Block themes require only `templates/index.html`, `theme.json`, and `style.css` for WordPress.org submission
- Template hierarchy remains identical between classic (PHP) and block (HTML) themes, with WordPress selecting templates using the same fallback logic
- `useRootPaddingAwareAlignments` is essential for proper full-width block alignment but commonly overlooked
- Style variations (alternate theme.json files in `styles/` directory) and block patterns (PHP files in `patterns/` directory) are first-class theme features
- Hardcoded styles in block themes defeat theme.json's purpose and are flagged by WordPress.org theme reviewers

**Primary recommendation:** Build comprehensive skill covering block themes primarily (WP 6.6+ with theme.json v3), classic themes for migration context, and hybrid themes for incremental adoption. Structure around theme type detection (block/classic/hybrid/child) with context-aware guidance.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Detection scope & depth:**
- **Deep coverage (dedicated sections in SKILL.md):** theme.json v3 structure validation (version, settings, styles, patterns, templateParts, customTemplates), template hierarchy for block themes (templates/ directory with .html files: index.html, single.html, page.html, archive.html, 404.html, search.html, home.html, front-page.html), template parts (parts/ directory: header.html, footer.html, sidebar.html with area designation), FSE templates with block markup, global styles (color palettes, typography scales, spacing presets, layout settings, border and shadow), style variations (styles/ directory with alternate theme.json files), block patterns in themes (patterns/ directory with PHP files and pattern headers), useRootPaddingAwareAlignments and layout system, child theme patterns for block themes
- **Surface-level coverage (scan patterns + brief guidance):** Navigation block configuration for menus, custom template registration via customTemplates in theme.json, block theme fonts management (fontFamilies in theme.json, local font hosting), theme.json v2 vs v3 differences (flag v2 with upgrade path), add_theme_support() calls still valid in block themes (wp-block-styles, responsive-embeds, editor-styles), classic theme conditional tags (is_front_page, is_singular, etc.), classic get_template_part() patterns, widget areas (classic themes only -- block themes use template parts)
- **Context-aware detection:** Distinguish between block theme, classic theme, hybrid theme, child theme, WordPress.org theme directory submission

**Block theme vs classic theme emphasis:**
- Block themes are primary focus (WP 6.6+)
- Classic themes get migration guidance
- Hybrid themes acknowledged
- Auto-detect theme type from file structure

**theme.json depth & version handling:**
- v3 is primary (WP 6.6+)
- v2 flagged with upgrade notes (WARNING)
- v1 flagged as deprecated (CRITICAL)
- Deep coverage of all theme.json sections: settings, styles, templateParts, customTemplates, patterns

**Style system & anti-patterns:**
- Hardcoded styles detection (THM-14)
- Block stylesheets (THM-15)
- useRootPaddingAwareAlignments (THM-20)
- Style variations (THM-19)

**Cross-references to existing skills:**
- wp-security-review: Template escaping, child theme security, sanitization
- wp-plugin-development: Theme hooks, functions.php patterns
- wp-block-development: Block patterns, block markup in templates, block supports
- wp-performance-review: Asset enqueuing, conditional loading, image optimization

**Reference doc structure:**
Four reference docs:
1. theme-json-guide.md — Complete v3 schema reference
2. template-patterns.md — Template hierarchy for classic and block themes
3. fse-guide.md — Full Site Editing comprehensive guide
4. classic-to-block-guide.md — Step-by-step migration guide

**Finding output style:**
- Mirror all prior skills exactly (FILE grouping, severity, BAD/GOOD pairs)
- CRITICAL/WARNING/INFO severity mapping for themes

**Quick scan vs full review boundary:**
- /wp-theme (quick scan): Grep-based pattern detection
- /wp-theme-review (full review): Complete skill workflow

### Claude's Discretion
- Exact grep patterns for quick detection
- Depth of Customizer coverage for classic themes
- How to handle starter themes and theme frameworks
- Block locking patterns in templates
- Whether to include WordPress.org Theme Check plugin alignment details
</user_constraints>

## Standard Stack

The established libraries/tools for WordPress theme development:

### Core Requirements
| Component | Version | Purpose | Why Standard |
|-----------|---------|---------|--------------|
| theme.json | v3 | Global settings and styles | Required for block themes (WP 6.6+), replaces add_theme_support() |
| style.css | - | Theme metadata + minimal styling | Required file with theme header, primary styles come from theme.json in block themes |
| templates/index.html | - | Default template fallback | Required for block themes, uses block markup not PHP |
| functions.php | - | Theme setup and hooks | Optional but common, same role in classic and block themes |

### Block Theme Structure
| Directory | Purpose | When to Use |
|-----------|---------|-------------|
| templates/ | HTML template files | Required directory for block themes (.html files) |
| parts/ | Template parts (header, footer, sidebar) | Reusable sections, replaces PHP get_template_part() |
| patterns/ | Theme-bundled block patterns | PHP files with pattern headers, auto-registered by WordPress |
| styles/ | Style variations | Alternate theme.json files for different color/typography schemes |
| assets/ | Fonts, images, JS/CSS | Theme assets, fonts for local hosting via theme.json fontFamilies |

### Classic Theme Structure (Migration Context)
| File | Purpose | Block Theme Equivalent |
|------|---------|------------------------|
| index.php | Default template | templates/index.html |
| header.php | Header markup | parts/header.html |
| footer.php | Footer markup | parts/footer.html |
| sidebar.php | Sidebar markup | parts/sidebar.html |
| single.php | Single post template | templates/single.html |
| archive.php | Archive template | templates/archive.html |

### Development Tools
| Tool | Purpose | When to Use |
|------|---------|-------------|
| Create Block Theme plugin | Convert classic to block theme | Migration projects, exporting customizations from Site Editor |
| Theme Check plugin | WordPress.org validation | Pre-submission testing, standards compliance |
| Theme Unit Test data | Content testing | Testing template hierarchy with all content types |
| @wordpress/scripts | Build tooling | If theme includes custom blocks or React components |

**Installation:**
```bash
# Block themes are native WordPress, no npm packages required for basic themes
# Only needed if theme includes custom blocks:
npm install @wordpress/scripts --save-dev
```

**Key insight:** Block themes require far less code than classic themes. A minimal block theme is three files (style.css, theme.json, templates/index.html) compared to classic themes requiring dozens of PHP template files.

## Architecture Patterns

### Recommended Block Theme Structure
```
block-theme/
├── style.css                 # Theme metadata (Theme Name, Version, etc.)
├── theme.json                # v3 schema - settings, styles, customTemplates, templateParts
├── functions.php             # Optional - hooks, custom post types, enqueues
├── templates/                # HTML templates using block markup
│   ├── index.html            # Required - fallback template
│   ├── home.html             # Blog homepage
│   ├── front-page.html       # Static front page
│   ├── single.html           # Single post
│   ├── page.html             # Single page
│   ├── archive.html          # Post archives
│   ├── 404.html              # 404 error page
│   └── search.html           # Search results
├── parts/                    # Template parts (header, footer, sidebar)
│   ├── header.html           # Header template part
│   ├── footer.html           # Footer template part
│   └── sidebar.html          # Sidebar template part
├── patterns/                 # Theme-bundled patterns (PHP files)
│   ├── hero.php              # Hero section pattern
│   └── call-to-action.php    # CTA pattern
├── styles/                   # Style variations (alternate theme.json)
│   ├── dark.json             # Dark color scheme
│   └── minimal.json          # Minimal typography variation
└── assets/                   # Fonts, images, CSS, JS
    ├── fonts/                # Local font files (.woff2 recommended)
    └── images/               # Theme images
```

### Pattern 1: theme.json v3 Structure
**What:** Central configuration file defining all theme settings and styles
**When to use:** Every block theme (required for FSE), optional for hybrid classic themes
**Example:**
```json
// Source: https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/
{
	"$schema": "https://schemas.wp.org/trunk/theme.json",
	"version": 3,
	"settings": {
		"typography": {
			"defaultFontSizes": false,
			"fontFamilies": [
				{
					"fontFamily": "\"Inter\", sans-serif",
					"name": "Inter",
					"slug": "inter",
					"fontFace": [
						{
							"fontFamily": "Inter",
							"fontWeight": "400",
							"fontStyle": "normal",
							"src": [ "file:./assets/fonts/inter-regular.woff2" ]
						}
					]
				}
			]
		},
		"color": {
			"palette": [
				{
					"slug": "primary",
					"color": "#0073aa",
					"name": "Primary"
				}
			]
		},
		"layout": {
			"contentSize": "640px",
			"wideSize": "1200px"
		},
		"useRootPaddingAwareAlignments": true
	},
	"styles": {
		"spacing": {
			"padding": {
				"top": "var(--wp--preset--spacing--50)",
				"right": "var(--wp--preset--spacing--50)",
				"bottom": "var(--wp--preset--spacing--50)",
				"left": "var(--wp--preset--spacing--50)"
			}
		},
		"typography": {
			"fontFamily": "var(--wp--preset--font-family--inter)",
			"fontSize": "var(--wp--preset--font-size--medium)"
		}
	},
	"templateParts": [
		{
			"name": "header",
			"title": "Header",
			"area": "header"
		}
	],
	"customTemplates": [
		{
			"name": "page-no-title",
			"title": "Page Without Title",
			"postTypes": [ "page" ]
		}
	]
}
```

### Pattern 2: Block Template Structure
**What:** HTML files containing block markup, not PHP template tags
**When to use:** All templates in block themes
**Example:**
```html
<!-- Source: https://developer.wordpress.org/themes/basics/template-hierarchy/ -->
<!-- wp:template-part {"slug":"header","area":"header"} /-->

<!-- wp:group {"layout":{"type":"constrained"}} -->
<div class="wp-block-group">
	<!-- wp:post-title /-->
	<!-- wp:post-content /-->
</div>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","area":"footer"} /-->
```

**Key difference from classic themes:** No PHP template tags (`the_title()`, `the_content()`), no PHP opening/closing tags, pure block markup.

### Pattern 3: Template Part Registration
**What:** Reusable template sections (header, footer, sidebar) defined in theme.json and parts/ directory
**When to use:** All block themes for modular template structure
**Example:**
```json
// Source: https://developer.wordpress.org/themes/global-settings-and-styles/template-parts/
// In theme.json:
{
	"templateParts": [
		{
			"name": "header",
			"title": "Header",
			"area": "header"
		},
		{
			"name": "footer",
			"title": "Footer",
			"area": "footer"
		},
		{
			"name": "sidebar",
			"title": "Sidebar",
			"area": "uncategorized"
		}
	]
}
```

```html
<!-- In parts/header.html: -->
<!-- wp:group {"layout":{"type":"flex","justifyContent":"space-between"}} -->
<div class="wp-block-group">
	<!-- wp:site-title /-->
	<!-- wp:navigation /-->
</div>
<!-- /wp:group -->
```

**Available areas:** `header`, `footer`, `uncategorized` (default), or custom areas (WP 6.0+).

### Pattern 4: Style Variations
**What:** Alternate theme.json files in styles/ directory providing different color/typography schemes
**When to use:** Offering users visual alternatives without separate theme installations
**Example:**
```json
// Source: https://developer.wordpress.org/themes/global-settings-and-styles/style-variations/
// In styles/dark.json:
{
	"version": 3,
	"title": "Dark Mode",
	"settings": {
		"color": {
			"palette": [
				{
					"slug": "background",
					"color": "#1a1a1a",
					"name": "Background"
				},
				{
					"slug": "foreground",
					"color": "#ffffff",
					"name": "Foreground"
				}
			]
		}
	},
	"styles": {
		"color": {
			"background": "var(--wp--preset--color--background)",
			"text": "var(--wp--preset--color--foreground)"
		}
	}
}
```

**Naming:** File name becomes variation name (dark.json → "Dark" in UI), `title` property overrides.

### Pattern 5: Theme-Bundled Block Patterns
**What:** PHP files in patterns/ directory with pattern headers, auto-registered by WordPress
**When to use:** Providing ready-made block layouts specific to the theme design
**Example:**
```php
<?php
/**
 * Title: Hero Section
 * Slug: mytheme/hero
 * Categories: featured
 * Keywords: hero, banner, header
 * Block Types: core/cover
 * Viewport Width: 1400
 */
?>
<!-- wp:cover {"url":"<?php echo esc_url( get_theme_file_uri( 'assets/images/hero.jpg' ) ); ?>","align":"full"} -->
<div class="wp-block-cover alignfull">
	<span aria-hidden="true" class="wp-block-cover__background"></span>
	<img class="wp-block-cover__image-background" alt="" src="<?php echo esc_url( get_theme_file_uri( 'assets/images/hero.jpg' ) ); ?>" />
	<div class="wp-block-cover__inner-container">
		<!-- wp:heading {"level":1} -->
		<h1><?php echo esc_html_x( 'Welcome', 'Pattern placeholder text', 'mytheme' ); ?></h1>
		<!-- /wp:heading -->
	</div>
</div>
<!-- /wp:cover -->
```

**Key insight:** Source: [Registering Patterns – Theme Handbook](https://developer.wordpress.org/themes/patterns/registering-patterns/). Patterns use PHP for dynamic values (`get_theme_file_uri()`, translation functions), but output is block markup. WordPress only supports PHP files in patterns/, not HTML or JSON.

### Pattern 6: useRootPaddingAwareAlignments
**What:** Essential setting for proper full-width block alignment with root padding
**When to use:** Any block theme using root-level padding (most themes)
**Example:**
```json
// Source: https://developer.wordpress.org/themes/global-settings-and-styles/settings/use-root-padding-aware-alignments/
{
	"version": 3,
	"settings": {
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

**Why critical:** Without this, full-width blocks (`alignfull` class) overlap root padding. With it, WordPress generates negative margins to pull full-width blocks to viewport edges. Must use object notation for padding (not CSS shorthand like `"2rem"`).

### Pattern 7: Theme Type Detection
**What:** Identifying whether theme is block, classic, hybrid, or child theme
**When to use:** First step of any theme review for context-aware guidance
**Example:**
```php
// Block theme detection:
// - Has templates/index.html
// - Has theme.json
// - style.css exists
if ( file_exists( $theme_path . '/templates/index.html' ) &&
     file_exists( $theme_path . '/theme.json' ) ) {
	// Block theme
}

// Classic theme detection:
// - Has index.php
// - No templates/index.html
elseif ( file_exists( $theme_path . '/index.php' ) &&
         ! file_exists( $theme_path . '/templates/index.html' ) ) {
	// Classic theme
}

// Hybrid theme detection:
// - Has both index.php AND theme.json
// - No templates directory (uses PHP templates with theme.json for block editor settings)
elseif ( file_exists( $theme_path . '/index.php' ) &&
         file_exists( $theme_path . '/theme.json' ) &&
         ! file_exists( $theme_path . '/templates' ) ) {
	// Hybrid theme
}

// Child theme detection:
// - style.css has "Template:" header pointing to parent theme
$style_contents = file_get_contents( $theme_path . '/style.css' );
if ( preg_match( '/^Template:/mi', $style_contents ) ) {
	// Child theme (can be block child or classic child)
}
```

### Anti-Patterns to Avoid

- **Hardcoded inline styles in block templates:** Defeats theme.json purpose, prevents user customization, flagged by WordPress.org reviewers
  ```html
  <!-- BAD: Inline styles in template -->
  <div style="color: #333; font-size: 18px;">Content</div>

  <!-- GOOD: Use theme.json presets via CSS variables -->
  <div style="color: var(--wp--preset--color--foreground); font-size: var(--wp--preset--font-size--medium);">Content</div>
  ```

- **Hardcoded color/font values in style.css:** Users can't change via Site Editor, breaks global styles system
  ```css
  /* BAD: Hardcoded values */
  body {
  	color: #333333;
  	font-size: 16px;
  }

  /* GOOD: Use CSS variables from theme.json */
  body {
  	color: var(--wp--preset--color--foreground);
  	font-size: var(--wp--preset--font-size--medium);
  }
  ```

- **add_theme_support() instead of theme.json settings:** Classic approach still works but deprecated for block themes
  ```php
  // BAD: Classic approach in block theme
  add_theme_support( 'custom-line-height' );
  add_theme_support( 'editor-color-palette', array( /* colors */ ) );

  // GOOD: Use theme.json for block themes
  // In theme.json:
  {
  	"settings": {
  		"typography": {
  			"lineHeight": true
  		},
  		"color": {
  			"palette": [ /* colors */ ]
  		}
  	}
  }
  ```

- **Missing useRootPaddingAwareAlignments with root padding:** Full-width blocks break layout
  ```json
  // BAD: Root padding without useRootPaddingAwareAlignments
  {
  	"styles": {
  		"spacing": {
  			"padding": { "left": "2rem", "right": "2rem" }
  		}
  	}
  	// Missing: "settings": { "useRootPaddingAwareAlignments": true }
  }
  ```

- **CSS shorthand for root padding with useRootPaddingAwareAlignments:** Must use object notation
  ```json
  // BAD: CSS shorthand doesn't work
  {
  	"settings": {
  		"useRootPaddingAwareAlignments": true
  	},
  	"styles": {
  		"spacing": {
  			"padding": "2rem"  // Won't work!
  		}
  	}
  }

  // GOOD: Object notation required
  {
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

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Menu navigation | Custom nav markup in header.html | Navigation block | Handles mobile menus, accessibility, user menu editing via Site Editor, menu fallback to Page List |
| Font loading | Manual @font-face CSS + wp_enqueue_style | theme.json fontFamilies with fontFace | WordPress handles enqueuing, editor preview, GDPR compliance (local hosting), .woff2 optimization |
| Color palette UI | Custom color picker or hardcoded colors | theme.json settings.color.palette | Generates CSS variables, Site Editor UI, user can customize via global styles, consistent across all blocks |
| Responsive typography | Media query CSS for font sizes | theme.json settings.typography with fluid typography | WordPress calculates clamp() values, scales smoothly between breakpoints, editor preview matches frontend |
| Template hierarchy logic | Custom PHP routing or template switching | WordPress native template hierarchy | Works identically for classic (.php) and block (.html) templates, respects child theme overrides, plugin-compatible |
| Style variations | Multiple theme installations or custom CSS switcher | styles/ directory with alternate theme.json files | Native Site Editor UI, instant preview, user-customizable per variation, no code required |
| Classic to block theme migration | Manual rewrite of all templates | Create Block Theme plugin | Exports existing customizations, converts templates, generates theme.json from current settings, preserves content |
| Block pattern library | Custom shortcodes or template includes | patterns/ directory with PHP files | Auto-registered by WordPress, searchable in inserter, proper block markup, translatable, theme-specific |

**Key insight:** Block themes invert the classic theme paradigm. Classic themes are "code-first" (PHP generates HTML). Block themes are "content-first" (block markup in templates, styling in theme.json, minimal PHP). Don't fight this by adding unnecessary PHP.

## Common Pitfalls

### Pitfall 1: theme.json v2 to v3 Migration Breaks Font/Spacing Defaults
**What goes wrong:** After updating theme.json version from 2 to 3, WordPress default font sizes and spacing presets suddenly appear alongside theme-defined sizes, or theme sizes stop overriding defaults
**Why it happens:** v3 changed default behavior for `settings.typography.defaultFontSizes` and `settings.spacing.defaultSpacingSizes`. Both now default to `true` (show WordPress defaults and prevent theme override), whereas v2 behavior was to hide defaults when theme defined custom sizes
**How to avoid:** Source: [Migrating Theme.json to Newer Versions](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-migrations/). When updating to v3, explicitly set:
```json
{
	"version": 3,
	"settings": {
		"typography": {
			"defaultFontSizes": false,  // If theme defines fontSizes
			"fontSizes": [ /* custom sizes */ ]
		},
		"spacing": {
			"defaultSpacingSizes": false,  // If theme defines spacingSizes or spacingScale
			"spacingSizes": [ /* custom sizes */ ]
		}
	}
}
```
**Warning signs:** After v3 update, Site Editor shows duplicate size options, or theme size names appear but apply WordPress default values

### Pitfall 2: Missing Required Files for WordPress.org Submission
**What goes wrong:** Theme appears complete locally but gets rejected during WordPress.org submission for missing required files
**Why it happens:** WordPress.org has stricter requirements than WordPress core. Block themes only need `templates/index.html` and `style.css` to function, but submission requires additional files
**How to avoid:** Source: [Required Theme Files – Theme Handbook](https://developer.wordpress.org/themes/releasing-your-theme/required-theme-files/). For WordPress.org submission, ensure:
- `style.css` with complete header (Theme Name, Theme URI, Author, Author URI, Description, Version, Requires at least, Tested up to, Requires PHP, License, License URI, Text Domain)
- `templates/index.html`
- `theme.json`
- `readme.txt` with theme description, installation instructions, changelog, credits
- `screenshot.png` or `screenshot.jpg` (max 1200x900px)
- All code/images GPL-compatible or GPL-compatible licensed
**Warning signs:** Local theme works perfectly but submission system returns "missing required files" error

### Pitfall 3: Hardcoded Styles in Block Themes Prevent User Customization
**What goes wrong:** Theme looks great initially but users can't change colors/typography via Site Editor, or changes don't apply consistently
**Why it happens:** Inline styles in templates or hardcoded values in CSS override theme.json settings, breaking the global styles system
**How to avoid:** Source: [Troubleshooting block themes](https://fullsiteediting.com/lessons/troubleshooting-block-themes/). Never use inline `style=""` attributes in templates, never hardcode color hex values or pixel font sizes in theme CSS. Always reference theme.json presets via CSS variables:
```html
<!-- BAD: Hardcoded inline style -->
<div style="background-color: #0073aa; padding: 2rem;">

<!-- GOOD: Block attributes using theme.json preset slugs -->
<!-- wp:group {"style":{"color":{"background":"var(--wp--preset--color--primary)"},"spacing":{"padding":"var(--wp--preset--spacing--50)"}}} -->
```
**Warning signs:** Site Editor global styles panel shows correct values but template doesn't reflect changes, specificity wars in browser DevTools, !important rules needed

### Pitfall 4: useRootPaddingAwareAlignments Missing Breaks Full-Width Layouts
**What goes wrong:** Full-width blocks (alignfull) don't reach viewport edges, or overflow causes horizontal scroll
**Why it happens:** Source: [Use Root Padding Aware Alignments](https://developer.wordpress.org/themes/global-settings-and-styles/settings/use-root-padding-aware-alignments/). When root-level padding is applied to `<body>` (via `styles.spacing.padding` in theme.json), full-width blocks need negative margins to extend beyond that padding. `useRootPaddingAwareAlignments: true` tells WordPress to generate those negative margins automatically
**How to avoid:** Any time you set root-level padding in theme.json, also set:
```json
{
	"settings": {
		"useRootPaddingAwareAlignments": true
	},
	"styles": {
		"spacing": {
			"padding": {
				"top": "var(--wp--preset--spacing--50)",
				"right": "var(--wp--preset--spacing--50)",
				"bottom": "var(--wp--preset--spacing--50)",
				"left": "var(--wp--preset--spacing--50)"
			}
		}
	}
}
```
And use object notation (not CSS shorthand) for padding values.
**Warning signs:** Full-width Cover blocks have white space on left/right edges, Group blocks with "Full width" alignment don't touch viewport edges

### Pitfall 5: Classic Theme Patterns in patterns/ Directory Don't Work
**What goes wrong:** PHP pattern files added to patterns/ directory don't appear in block inserter
**Why it happens:** Source: [Registering Patterns – Theme Handbook](https://developer.wordpress.org/themes/patterns/registering-patterns/). WordPress auto-registers patterns from patterns/ directory ONLY for themes (not plugins), and pattern files must have valid PHP file headers with at minimum `Title` and `Slug` fields
**How to avoid:** Ensure every pattern file starts with:
```php
<?php
/**
 * Title: Pattern Name
 * Slug: themename/pattern-slug
 * Categories: featured
 */
?>
<!-- wp:group -->
<!-- Block markup here -->
<!-- /wp:group -->
```
Pattern slug must be namespaced (themename/pattern-slug). Available headers: Title, Slug, Categories, Description, Keywords, Block Types, Post Types, Template Types, Viewport Width, Inserter.
**Warning signs:** Pattern files exist in patterns/ but don't appear in inserter, or appear without preview thumbnail

### Pitfall 6: Child Theme Doesn't Override Parent Template Parts
**What goes wrong:** Child theme has parts/header.html but parent theme header still displays
**Why it happens:** Source: [Full site editing child themes](https://fullsiteediting.com/lessons/child-themes/). Template parts in child themes must match parent theme file names exactly, and child theme.json doesn't automatically override parent theme.json templateParts registration
**How to avoid:** Child theme must:
1. Copy parent's templateParts definition to child theme.json and modify, or
2. Keep identical file names (parts/header.html overrides parent parts/header.html by filename)
3. Use functions.php with `unregister_block_pattern()` if removing parent patterns
**Warning signs:** Child theme template parts created but parent versions still render, child theme.json changes ignored

### Pitfall 7: Hybrid Theme theme.json Doesn't Apply to Classic Templates
**What goes wrong:** Color palette defined in theme.json shows in editor but doesn't affect frontend classic template output
**Why it happens:** Source: [Adding full site editing features to classic themes](https://fullsiteediting.com/lessons/adding-full-site-editing-features-to-classic-themes/). theme.json settings only generate CSS variables and affect blocks. Classic PHP templates using `<?php the_content(); ?>` or custom markup don't automatically adopt theme.json styling unless theme manually applies CSS variables
**How to avoid:** In hybrid themes, either:
1. Use theme.json for block editor only (set expectations correctly)
2. Manually apply CSS variables from theme.json in theme CSS: `body { color: var(--wp--preset--color--foreground); }`
3. Add `add_theme_support( 'block-template-parts' )` and migrate header/footer to block template parts
**Warning signs:** Editor shows correct colors but frontend uses different colors, block content in posts styled correctly but surrounding theme markup isn't

### Pitfall 8: Local Font Files Not Loading in Block Editor
**What goes wrong:** Custom fonts display on frontend but not in block editor
**Why it happens:** Source: [Loading Fonts using theme.json](https://gutenberg.10up.com/reference/Themes/fonts/). theme.json fontFamilies must include `src` property pointing to font files, and paths must be relative from theme root using `file:` prefix
**How to avoid:**
```json
{
	"settings": {
		"typography": {
			"fontFamilies": [
				{
					"fontFamily": "\"Custom Font\", sans-serif",
					"name": "Custom Font",
					"slug": "custom-font",
					"fontFace": [
						{
							"fontFamily": "Custom Font",
							"fontWeight": "400",
							"fontStyle": "normal",
							"src": [ "file:./assets/fonts/custom-font.woff2" ]
						}
					]
				}
			]
		}
	}
}
```
Prefer .woff2 format (30-50% smaller than .ttf). Without `src`, WordPress doesn't know where to find fonts for editor preview.
**Warning signs:** Frontend shows custom font, editor shows system font fallback, block editor inspector shows font family name but doesn't apply it

## Code Examples

Verified patterns from official sources:

### Example 1: Complete theme.json v3 Structure
```json
// Source: https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/
{
	"$schema": "https://schemas.wp.org/trunk/theme.json",
	"version": 3,
	"settings": {
		"appearanceTools": true,
		"useRootPaddingAwareAlignments": true,
		"color": {
			"defaultPalette": false,
			"palette": [
				{
					"slug": "primary",
					"color": "#0073aa",
					"name": "Primary"
				},
				{
					"slug": "secondary",
					"color": "#005177",
					"name": "Secondary"
				},
				{
					"slug": "foreground",
					"color": "#333333",
					"name": "Foreground"
				},
				{
					"slug": "background",
					"color": "#ffffff",
					"name": "Background"
				}
			],
			"gradients": [],
			"duotone": []
		},
		"typography": {
			"defaultFontSizes": false,
			"fluid": true,
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
							"src": [ "file:./assets/fonts/inter-regular.woff2" ]
						},
						{
							"fontFamily": "Inter",
							"fontWeight": "700",
							"fontStyle": "normal",
							"src": [ "file:./assets/fonts/inter-bold.woff2" ]
						}
					]
				}
			],
			"fontSizes": [
				{
					"slug": "small",
					"size": "0.875rem",
					"name": "Small"
				},
				{
					"slug": "medium",
					"size": "1rem",
					"name": "Medium"
				},
				{
					"slug": "large",
					"size": "1.5rem",
					"name": "Large"
				},
				{
					"slug": "x-large",
					"size": "2rem",
					"name": "Extra Large"
				}
			]
		},
		"spacing": {
			"defaultSpacingSizes": false,
			"spacingSizes": [
				{
					"slug": "30",
					"size": "1rem",
					"name": "Small"
				},
				{
					"slug": "40",
					"size": "1.5rem",
					"name": "Medium"
				},
				{
					"slug": "50",
					"size": "2rem",
					"name": "Large"
				},
				{
					"slug": "60",
					"size": "3rem",
					"name": "Extra Large"
				}
			],
			"units": [ "px", "rem", "vh", "vw", "%" ]
		},
		"layout": {
			"contentSize": "640px",
			"wideSize": "1200px"
		},
		"border": {
			"radius": true,
			"color": true,
			"style": true,
			"width": true
		}
	},
	"styles": {
		"color": {
			"background": "var(--wp--preset--color--background)",
			"text": "var(--wp--preset--color--foreground)"
		},
		"typography": {
			"fontFamily": "var(--wp--preset--font-family--inter)",
			"fontSize": "var(--wp--preset--font-size--medium)",
			"lineHeight": "1.6"
		},
		"spacing": {
			"padding": {
				"top": "var(--wp--preset--spacing--50)",
				"right": "var(--wp--preset--spacing--50)",
				"bottom": "var(--wp--preset--spacing--50)",
				"left": "var(--wp--preset--spacing--50)"
			}
		},
		"elements": {
			"link": {
				"color": {
					"text": "var(--wp--preset--color--primary)"
				},
				":hover": {
					"color": {
						"text": "var(--wp--preset--color--secondary)"
					}
				}
			},
			"heading": {
				"typography": {
					"fontWeight": "700",
					"lineHeight": "1.2"
				}
			}
		},
		"blocks": {
			"core/button": {
				"color": {
					"background": "var(--wp--preset--color--primary)",
					"text": "var(--wp--preset--color--background)"
				},
				"border": {
					"radius": "4px"
				}
			}
		}
	},
	"customTemplates": [
		{
			"name": "page-no-title",
			"title": "Page Without Title",
			"postTypes": [ "page" ]
		},
		{
			"name": "single-product",
			"title": "Product Page",
			"postTypes": [ "product" ]
		}
	],
	"templateParts": [
		{
			"name": "header",
			"title": "Header",
			"area": "header"
		},
		{
			"name": "footer",
			"title": "Footer",
			"area": "footer"
		},
		{
			"name": "sidebar",
			"title": "Sidebar",
			"area": "uncategorized"
		}
	]
}
```

### Example 2: Block Theme Template (templates/single.html)
```html
<!-- Source: https://developer.wordpress.org/themes/basics/template-hierarchy/ -->
<!-- wp:template-part {"slug":"header","area":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
	<!-- wp:post-title {"level":1} /-->

	<!-- wp:group {"layout":{"type":"flex","flexWrap":"nowrap"}} -->
	<div class="wp-block-group">
		<!-- wp:post-author {"showAvatar":true,"showBio":false} /-->
		<!-- wp:post-date /-->
		<!-- wp:post-terms {"term":"category"} /-->
	</div>
	<!-- /wp:group -->

	<!-- wp:post-featured-image {"align":"wide"} /-->

	<!-- wp:post-content {"layout":{"type":"constrained"}} /-->

	<!-- wp:post-terms {"term":"post_tag"} /-->

	<!-- wp:comments -->
	<div class="wp-block-comments">
		<!-- wp:comments-title /-->
		<!-- wp:comment-template -->
			<!-- wp:comment-author-name /-->
			<!-- wp:comment-content /-->
			<!-- wp:comment-reply-link /-->
		<!-- /wp:comment-template -->
		<!-- wp:comments-pagination /-->
		<!-- wp:post-comments-form /-->
	</div>
	<!-- /wp:comments -->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","area":"footer"} /-->
```

### Example 3: Template Part (parts/header.html)
```html
<!-- Source: https://developer.wordpress.org/themes/global-settings-and-styles/template-parts/ -->
<!-- wp:group {"align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|40","bottom":"var:preset|spacing|40"}}},"layout":{"type":"constrained"}} -->
<div class="wp-block-group alignfull">
	<!-- wp:group {"layout":{"type":"flex","justifyContent":"space-between","flexWrap":"nowrap"}} -->
	<div class="wp-block-group">
		<!-- wp:site-logo {"width":60} /-->
		<!-- wp:site-title {"level":0} /-->

		<!-- wp:navigation {"layout":{"type":"flex","orientation":"horizontal"},"style":{"spacing":{"blockGap":"var:preset|spacing|40"}}} /-->
	</div>
	<!-- /wp:group -->
</div>
<!-- /wp:group -->
```

### Example 4: Block Pattern in themes (patterns/hero.php)
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
 */
?>
<!-- wp:cover {"url":"<?php echo esc_url( get_theme_file_uri( 'assets/images/hero.jpg' ) ); ?>","dimRatio":50,"align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}}}} -->
<div class="wp-block-cover alignfull">
	<span aria-hidden="true" class="wp-block-cover__background has-background-dim"></span>
	<img class="wp-block-cover__image-background" alt="" src="<?php echo esc_url( get_theme_file_uri( 'assets/images/hero.jpg' ) ); ?>" data-object-fit="cover" />

	<div class="wp-block-cover__inner-container">
		<!-- wp:heading {"textAlign":"center","level":1,"style":{"typography":{"fontSize":"3rem"}}} -->
		<h1 class="has-text-align-center"><?php echo esc_html_x( 'Welcome to Our Site', 'Pattern placeholder text', 'mytheme' ); ?></h1>
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

### Example 5: Style Variation (styles/dark.json)
```json
// Source: https://developer.wordpress.org/themes/global-settings-and-styles/style-variations/
{
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

### Example 6: functions.php for Block Themes
```php
<?php
/**
 * Theme setup and initialization
 * Source: https://developer.wordpress.org/themes/basics/theme-functions/
 */

// Security: Exit if accessed directly
if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

/**
 * Theme setup
 */
function mytheme_setup() {
	// Add theme support (most done via theme.json in block themes)
	add_theme_support( 'wp-block-styles' );
	add_theme_support( 'responsive-embeds' );
	add_theme_support( 'editor-styles' );

	// Load editor stylesheet (for classic editor blocks, optional)
	add_editor_style( 'style.css' );

	// Internationalization
	load_theme_textdomain( 'mytheme', get_template_directory() . '/languages' );
}
add_action( 'after_setup_theme', 'mytheme_setup' );

/**
 * Enqueue additional scripts/styles (block themes auto-load most assets)
 */
function mytheme_enqueue_assets() {
	// Custom JS for interactions (if needed)
	wp_enqueue_script(
		'mytheme-interactions',
		get_theme_file_uri( '/assets/js/interactions.js' ),
		array(),
		wp_get_theme()->get( 'Version' ),
		true
	);
}
add_action( 'wp_enqueue_scripts', 'mytheme_enqueue_assets' );

/**
 * Register custom block patterns categories (optional)
 */
function mytheme_register_pattern_categories() {
	register_block_pattern_category(
		'mytheme-featured',
		array( 'label' => __( 'Featured Sections', 'mytheme' ) )
	);
}
add_action( 'init', 'mytheme_register_pattern_categories' );
```

### Example 7: Classic Theme Migration - add_theme_support() to theme.json Mapping
```php
// BAD: Classic theme approach (deprecated for block themes)
function classictheme_setup() {
	add_theme_support( 'align-wide' );
	add_theme_support( 'custom-line-height' );
	add_theme_support( 'custom-spacing' );
	add_theme_support( 'editor-color-palette', array(
		array(
			'name'  => 'Primary',
			'slug'  => 'primary',
			'color' => '#0073aa',
		),
		array(
			'name'  => 'Secondary',
			'slug'  => 'secondary',
			'color' => '#005177',
		),
	) );
	add_theme_support( 'editor-font-sizes', array(
		array(
			'name' => 'Small',
			'size' => 14,
			'slug' => 'small'
		),
		array(
			'name' => 'Medium',
			'size' => 16,
			'slug' => 'medium'
		),
		array(
			'name' => 'Large',
			'size' => 24,
			'slug' => 'large'
		),
	) );
}
add_action( 'after_setup_theme', 'classictheme_setup' );
```

```json
// GOOD: Block theme approach via theme.json
// Source: https://developer.wordpress.org/block-editor/how-to-guides/themes/theme-support/
{
	"version": 3,
	"settings": {
		"layout": {
			"contentSize": "640px",
			"wideSize": "1200px"
		},
		"typography": {
			"lineHeight": true,
			"fontSizes": [
				{
					"slug": "small",
					"size": "0.875rem",
					"name": "Small"
				},
				{
					"slug": "medium",
					"size": "1rem",
					"name": "Medium"
				},
				{
					"slug": "large",
					"size": "1.5rem",
					"name": "Large"
				}
			]
		},
		"spacing": {
			"padding": true,
			"margin": true,
			"blockGap": true
		},
		"color": {
			"palette": [
				{
					"slug": "primary",
					"color": "#0073aa",
					"name": "Primary"
				},
				{
					"slug": "secondary",
					"color": "#005177",
					"name": "Secondary"
				}
			]
		}
	}
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Classic themes with PHP templates | Block themes with HTML templates | WP 5.9 (Jan 2022) | Fundamental architecture shift - content-first vs code-first |
| theme.json v2 | theme.json v3 | WP 6.6 (July 2024) | Breaking changes: defaultFontSizes and defaultSpacingSizes now default to true |
| add_theme_support() in functions.php | settings in theme.json | WP 5.8+ | Declarative configuration replaces imperative code, affects editor and frontend |
| Google Fonts via CDN (@import or wp_enqueue_style) | Local font hosting via theme.json fontFace | WP 6.0+ | Privacy compliance (GDPR), performance (fewer requests), theme.json fontFace property |
| register_nav_menus() in PHP | Navigation block in templates | WP 5.9+ | Users edit menus in Site Editor, no separate Appearance > Menus screen for block themes |
| Customizer API for theme options | Global Styles in Site Editor | WP 5.9+ | User customizations stored as theme variations, no code required for color/typography options |
| Widget areas with register_sidebar() | Template parts (parts/sidebar.html) | WP 5.9+ | Block-based template parts replace widget PHP API |
| Manual @font-face CSS + enqueuing | theme.json fontFamilies with fontFace array | WP 6.0+ | WordPress handles enqueuing, editor preview, optimization |
| Manual color palette CSS + custom controls | theme.json color.palette | WP 5.8+ | Generates CSS variables, Site Editor UI, user-customizable |

**Deprecated/outdated:**
- **theme.json v1:** Deprecated, update to v3 (CRITICAL severity)
- **theme.json v2:** Still works but flag for upgrade (WARNING severity) - v3 has breaking changes requiring migration
- **Underscores (_s) starter theme:** Project archived by Automattic, focus shifted to block themes - still functional for classic themes but not actively maintained
- **Classic theme approach for new development:** Block themes are WordPress's future (stated in Gutenberg plugin roadmap and Make WordPress Core blog)
- **TrueType (.ttf) fonts:** 30-50% larger than .woff2, use .woff2 for web (source: [Managing Fonts in WordPress Block Themes](https://css-tricks.com/managing-fonts-in-wordpress-block-themes/))
- **$content parameter escaping in render callbacks:** From wp-block-development skill - using `wp_kses_post( $content )` on InnerBlocks content breaks embeds, just echo directly

## Open Questions

Things that couldn't be fully resolved:

1. **Exact WordPress.org theme submission requirements for block patterns**
   - What we know: Patterns in patterns/ directory are auto-registered, must have PHP file headers
   - What's unclear: Are there minimum pattern requirements for theme directory approval? Is there a maximum number before "too many patterns" flags?
   - Recommendation: Flag as INFO if theme has no patterns (missed opportunity), don't enforce pattern requirements as CRITICAL unless WordPress.org documentation explicitly requires them

2. **Block locking depth for theme review scope**
   - What we know: Block locking exists (templateLock attribute), can lock blocks from moving/deleting, supported in group/cover/columns/column/navigation blocks
   - What's unclear: How deep should skill review go? Detect presence/absence? Validate lock levels? Check for accessibility issues with overly-locked templates?
   - Recommendation: Surface-level coverage - detect templateLock attribute, flag if used without context (INFO), note that excessive locking reduces user flexibility

3. **Customizer API coverage for classic themes**
   - What we know: Classic themes use Customizer API for theme options, replaced by Global Styles in block themes
   - What's unclear: How much Customizer validation should skill include for classic theme reviews? Security issues only? Or also validate controls, sanitization, partial refresh setup?
   - Recommendation: Minimal coverage - cross-reference to wp-security-review for sanitization, mention Customizer → Global Styles migration opportunity, don't deep-dive Customizer API patterns

4. **Hybrid theme detection vs classic theme with theme.json**
   - What we know: Hybrid themes have both index.php and theme.json, use PHP templates with block editor settings
   - What's unclear: Is there a formal WordPress definition of "hybrid theme"? Or is it community terminology? Should skill use "hybrid theme" label or "classic theme with block editor support"?
   - Recommendation: Use "hybrid theme" terminology (appears in [official WordPress Developer News blog](https://developer.wordpress.org/news/2024/12/bridging-the-gap-hybrid-themes/)), detect by presence of both index.php and theme.json without templates/ directory

5. **Theme.json $schema property necessity**
   - What we know: $schema enables IDE autocomplete and validation, points to https://schemas.wp.org/trunk/theme.json
   - What's unclear: Is $schema required, recommended, or purely optional? Does WordPress.org theme review check for it?
   - Recommendation: Flag missing $schema as INFO (enhancement for DX), not WARNING/CRITICAL

## Sources

### Primary (HIGH confidence)
- [Theme.json Version 3 Reference (latest) – Block Editor Handbook](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/) - Complete v3 schema documentation
- [Migrating Theme.json to Newer Versions – Block Editor Handbook](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-migrations/) - v2 to v3 migration guide with breaking changes
- [Global Settings & Styles (theme.json) – Theme Handbook](https://developer.wordpress.org/themes/global-settings-and-styles/) - Theme.json best practices and structure
- [Template Hierarchy – Theme Handbook](https://developer.wordpress.org/themes/basics/template-hierarchy/) - Classic and block theme template hierarchy
- [Registering Patterns – Theme Handbook](https://developer.wordpress.org/themes/patterns/registering-patterns/) - Pattern file headers and auto-registration
- [Template Parts – Theme Handbook](https://developer.wordpress.org/themes/global-settings-and-styles/template-parts/) - Template part areas and registration
- [Custom Templates – Theme Handbook](https://developer.wordpress.org/themes/global-settings-and-styles/custom-templates/) - customTemplates in theme.json
- [Use Root Padding Aware Alignments – Theme Handbook](https://developer.wordpress.org/themes/global-settings-and-styles/settings/use-root-padding-aware-alignments/) - Full-width alignment handling
- [Style Variations – Theme Handbook](https://developer.wordpress.org/themes/global-settings-and-styles/style-variations/) - styles/ directory structure
- [Required Theme Files – Theme Handbook](https://developer.wordpress.org/themes/releasing-your-theme/required-theme-files/) - WordPress.org submission requirements
- [Theme Support – Block Editor Handbook](https://developer.wordpress.org/block-editor/how-to-guides/themes/theme-support/) - add_theme_support() for block themes
- [Patterns and Block Locking – Theme Handbook](https://developer.wordpress.org/themes/patterns/patterns-and-block-locking/) - templateLock attribute

### Secondary (MEDIUM confidence)
- [Theme.json version 3 frequently asked questions – Make WordPress Themes](https://make.wordpress.org/themes/2024/06/21/theme-json-version-3-frequently-asked-questions/) - Official v3 FAQ, verified migration steps
- [Bridging the gap: Hybrid themes – WordPress Developer Blog](https://developer.wordpress.org/news/2024/12/bridging-the-gap-hybrid-themes/) - Official WordPress blog post defining hybrid themes
- [Full site editing child themes - Full Site Editing](https://fullsiteediting.com/lessons/child-themes/) - Child theme patterns for block themes
- [Troubleshooting block themes - Full Site Editing](https://fullsiteediting.com/lessons/troubleshooting-block-themes/) - Common mistakes and solutions
- [Creating WordPress block themes - Full Site Editing](https://fullsiteediting.com/lessons/creating-block-based-themes/) - Block theme structure and files
- [Loading Fonts using theme.json | 10up - WP Block Editor Best Practices](https://gutenberg.10up.com/reference/Themes/fonts/) - Font loading best practices
- [Managing Fonts in WordPress Block Themes | CSS-Tricks](https://css-tricks.com/managing-fonts-in-wordpress-block-themes/) - .woff2 optimization guidance
- [How to lock blocks and templates - Full Site Editing](https://fullsiteediting.com/how-to-lock-blocks-and-templates/) - Block locking patterns

### Tertiary (LOW confidence - WebSearch only, marked for validation)
- [Kinsta blog: theme.json guide](https://kinsta.com/blog/theme-json/) - Community tutorial, unverified against official docs
- [WPBeginner: theme.json guide](https://www.wpbeginner.com/beginners-guide/what-is-theme-json-file-in-wordpress-and-how-to-use-it/) - Community tutorial, good overview but verify specifics

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - WordPress core features, official documentation
- theme.json v3 schema: HIGH - Official Block Editor Handbook, schema hosted by WordPress.org
- Architecture patterns: HIGH - Verified with official docs and WordPress Developer Blog
- Migration guidance: MEDIUM - Official docs exist but community interpretations vary
- Block locking depth: LOW - Limited official documentation on review best practices
- Pitfalls: HIGH - Sourced from official Make WordPress blogs and Theme Handbook troubleshooting

**Research date:** 2026-02-06
**Valid until:** ~30 days (WordPress 6.7 expected late February/early March 2026, may introduce theme.json changes)

**Key WordPress versions:**
- WP 6.6 (current): Introduced theme.json v3
- WP 6.5: Introduced viewScriptModule for ES modules
- WP 6.4: Introduced blockHooks
- WP 6.3: Block theme patterns in patterns/ directory auto-registration
- WP 6.1: useRootPaddingAwareAlignments introduced
- WP 6.0: Template parts areas, fontFace support
- WP 5.9: Block themes and FSE stable release
- WP 5.8: theme.json v1 introduced

**Cross-skill dependencies:**
- wp-block-development: Block patterns registration, block markup in templates, InnerBlocks patterns
- wp-security-review: Template escaping (classic themes), sanitization in Customizer callbacks
- wp-plugin-development: functions.php patterns, hooks (after_setup_theme, wp_enqueue_scripts, init)
- wp-performance-review: Asset enqueuing, conditional loading via viewScript/viewScriptModule
