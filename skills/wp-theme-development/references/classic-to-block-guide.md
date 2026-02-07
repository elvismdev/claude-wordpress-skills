# WordPress Classic to Block Theme Migration Guide

This reference provides step-by-step guidance for migrating classic PHP-based WordPress themes to modern block themes with Full Site Editing support. Covers functions.php to theme.json mapping, template conversion (PHP to HTML block markup), sidebar migration, menu conversion, Customizer to Global Styles, and incremental adoption strategies for hybrid themes.

## Quick Reference Table

### Classic Feature to Block Equivalent

| Classic Feature | Block Theme Equivalent | Complexity | Migration Priority |
|-----------------|------------------------|------------|-------------------|
| **add_theme_support('align-wide')** | theme.json settings.layout | Low | High (fundamental) |
| **add_theme_support('custom-line-height')** | theme.json settings.typography.lineHeight | Low | High |
| **add_theme_support('editor-color-palette')** | theme.json settings.color.palette | Low | High |
| **add_theme_support('editor-font-sizes')** | theme.json settings.typography.fontSizes | Low | High |
| **register_nav_menus()** | wp:navigation block | Medium | High (user-visible) |
| **register_sidebar()** | Template parts (parts/*.html) | Medium | High (layout) |
| **header.php / footer.php** | parts/header.html, parts/footer.html | Medium | High (structure) |
| **index.php, single.php, etc.** | templates/*.html | High | High (core templates) |
| **Customizer API** | Global Styles (Site Editor) | High | Medium (optional) |
| **get_template_part()** | wp:template-part block | Low | Medium |
| **Dynamic sidebars / widgets** | wp:group blocks or Query Loop | Medium | Medium |
| **the_title(), the_content()** | wp:post-title, wp:post-content blocks | Medium | High |

## Migration Strategy Overview

### Three Approaches

**1. Full Conversion (Big Bang)**

**What:** Convert entire classic theme to block theme at once

**When to use:**
- Theme has <10 template files
- No complex custom functionality
- Clean slate acceptable

**Pros:**
- Clean break, modern architecture
- No hybrid maintenance burden

**Cons:**
- High upfront effort
- All-or-nothing risk

---

**2. Hybrid Theme (Gradual)**

**What:** Classic theme with theme.json for block editor settings

**When to use:**
- Large existing theme
- Active sites can't have downtime
- Team needs training time

**Pros:**
- Incremental adoption
- Backwards compatible
- Learn as you go

**Cons:**
- Dual system maintenance
- theme.json doesn't affect classic templates (only editor)

---

**3. Create Block Theme Plugin**

**What:** Use WordPress plugin to export Site Editor customizations

**When to use:**
- Converting heavily customized site
- Want to preserve user customizations
- Starting from existing classic theme

**Pros:**
- Automated template generation
- Preserves existing work
- Export user Global Styles

**Cons:**
- Requires Site Editor familiarity
- Output may need cleanup

**Recommended approach:** Start with hybrid (#2), progressively convert templates, finish with full block theme.

## functions.php to theme.json Mapping

### Theme Support Equivalents

**Classic functions.php:**

```php
<?php
function mytheme_setup() {
	// Wide alignment
	add_theme_support( 'align-wide' );

	// Custom line height
	add_theme_support( 'custom-line-height' );

	// Custom spacing
	add_theme_support( 'custom-spacing' );

	// Editor color palette
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
		array(
			'name'  => 'Foreground',
			'slug'  => 'foreground',
			'color' => '#333333',
		),
		array(
			'name'  => 'Background',
			'slug'  => 'background',
			'color' => '#ffffff',
		),
	) );

	// Editor font sizes
	add_theme_support( 'editor-font-sizes', array(
		array(
			'name' => 'Small',
			'size' => 14,
			'slug' => 'small',
		),
		array(
			'name' => 'Medium',
			'size' => 16,
			'slug' => 'medium',
		),
		array(
			'name' => 'Large',
			'size' => 24,
			'slug' => 'large',
		),
		array(
			'name' => 'Extra Large',
			'size' => 36,
			'slug' => 'x-large',
		),
	) );

	// Responsive embeds
	add_theme_support( 'responsive-embeds' );

	// Editor styles
	add_theme_support( 'editor-styles' );

	// Custom units
	add_theme_support( 'custom-units', 'px', 'rem', 'em', 'vh', 'vw' );
}
add_action( 'after_setup_theme', 'mytheme_setup' );
```

**Block theme theme.json equivalent:**

```json
{
	"$schema": "https://schemas.wp.org/trunk/theme.json",
	"version": 3,
	"settings": {
		"layout": {
			"contentSize": "640px",
			"wideSize": "1200px"
		},
		"typography": {
			"lineHeight": true,
			"defaultFontSizes": false,
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
					"size": "2.25rem",
					"name": "Extra Large"
				}
			]
		},
		"spacing": {
			"padding": true,
			"margin": true,
			"blockGap": true,
			"units": ["px", "rem", "em", "vh", "vw", "%"]
		},
		"color": {
			"defaultPalette": false,
			"custom": true,
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
			]
		}
	}
}
```

**Key differences:**

- Font sizes: px → rem units (more accessible, scales with browser settings)
- Add `defaultFontSizes: false` in v3 to prevent WordPress defaults
- Add `defaultPalette: false` to hide WordPress default colors
- `align-wide` becomes `layout.contentSize` and `layout.wideSize`

### add_theme_support() Still Valid in Block Themes

**Some add_theme_support calls remain useful:**

```php
<?php
function mytheme_setup() {
	// Block styles (optional styling)
	add_theme_support( 'wp-block-styles' );

	// Responsive embeds
	add_theme_support( 'responsive-embeds' );

	// Editor styles
	add_theme_support( 'editor-styles' );

	// Post thumbnails
	add_theme_support( 'post-thumbnails' );

	// Custom logo
	add_theme_support( 'custom-logo', array(
		'height'      => 100,
		'width'       => 400,
		'flex-height' => true,
		'flex-width'  => true,
	) );

	// HTML5 support
	add_theme_support( 'html5', array(
		'search-form',
		'comment-form',
		'comment-list',
		'gallery',
		'caption',
		'style',
		'script',
	) );

	// Automatic feed links
	add_theme_support( 'automatic-feed-links' );

	// Title tag
	add_theme_support( 'title-tag' );

	// Load text domain
	load_theme_textdomain( 'mytheme', get_template_directory() . '/languages' );
}
add_action( 'after_setup_theme', 'mytheme_setup' );
```

**What stays in functions.php:**

- Post thumbnails
- Custom logo
- HTML5 support
- Feed links
- Title tag
- Text domain loading
- Custom post types
- Custom taxonomies

**What moves to theme.json:**

- Color palette
- Font sizes
- Layout settings
- Spacing controls
- Typography controls
- Border controls

## Template Conversion (PHP to HTML)

### header.php to parts/header.html

**Classic header.php:**

```php
<!DOCTYPE html>
<html <?php language_attributes(); ?>>
<head>
	<meta charset="<?php bloginfo( 'charset' ); ?>">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<?php wp_head(); ?>
</head>

<body <?php body_class(); ?>>
<?php wp_body_open(); ?>

<div id="page" class="site">
	<header id="masthead" class="site-header">
		<div class="site-branding">
			<?php
			if ( has_custom_logo() ) {
				the_custom_logo();
			} else {
				?>
				<h1 class="site-title">
					<a href="<?php echo esc_url( home_url( '/' ) ); ?>" rel="home">
						<?php bloginfo( 'name' ); ?>
					</a>
				</h1>
				<?php
			}

			$description = get_bloginfo( 'description', 'display' );
			if ( $description ) :
				?>
				<p class="site-description"><?php echo esc_html( $description ); ?></p>
			<?php endif; ?>
		</div>

		<nav id="site-navigation" class="main-navigation">
			<?php
			wp_nav_menu( array(
				'theme_location' => 'primary',
				'menu_id'        => 'primary-menu',
			) );
			?>
		</nav>
	</header>
```

**Block theme parts/header.html:**

```html
<!-- wp:group {"align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|40","bottom":"var:preset|spacing|40"}}},"layout":{"type":"constrained"}} -->
<div class="wp-block-group alignfull">
	<!-- wp:group {"layout":{"type":"flex","justifyContent":"space-between","flexWrap":"nowrap"}} -->
	<div class="wp-block-group">
		<!-- wp:group {"layout":{"type":"flex","flexWrap":"nowrap"}} -->
		<div class="wp-block-group">
			<!-- wp:site-logo {"width":60} /-->
			<!-- wp:site-title {"level":0} /-->
		</div>
		<!-- /wp:group -->

		<!-- wp:navigation {"layout":{"type":"flex","orientation":"horizontal"},"style":{"spacing":{"blockGap":"var:preset|spacing|40"}}} /-->
	</div>
	<!-- /wp:group -->
</div>
<!-- /wp:group -->
```

**Key conversions:**

| Classic PHP | Block Equivalent |
|-------------|------------------|
| `the_custom_logo()` | `wp:site-logo` block |
| `bloginfo('name')` | `wp:site-title` block |
| `get_bloginfo('description')` | `wp:site-tagline` block |
| `wp_nav_menu()` | `wp:navigation` block |
| `<div class="site-header">` | `wp:group` block with className |

**DOCTYPE, <html>, <head>, <body>:**

WordPress auto-generates these in block themes — don't include in template parts.

### footer.php to parts/footer.html

**Classic footer.php:**

```php
	<footer id="colophon" class="site-footer">
		<div class="site-info">
			<p>&copy; <?php echo esc_html( date( 'Y' ) ); ?> <?php bloginfo( 'name' ); ?></p>
			<p>
				<a href="<?php echo esc_url( __( 'https://wordpress.org/', 'mytheme' ) ); ?>">
					<?php
					/* translators: %s: CMS name (WordPress) */
					printf( esc_html__( 'Proudly powered by %s', 'mytheme' ), 'WordPress' );
					?>
				</a>
			</p>
		</div>
	</footer>
</div><!-- #page -->

<?php wp_footer(); ?>

</body>
</html>
```

**Block theme parts/footer.html:**

```html
<!-- wp:group {"align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|50","bottom":"var:preset|spacing|50"}}},"backgroundColor":"contrast","textColor":"base","layout":{"type":"constrained"}} -->
<div class="wp-block-group alignfull has-contrast-background-color has-base-color has-text-color has-background">
	<!-- wp:paragraph {"align":"center"} -->
	<p class="has-text-align-center">&copy; 2024 <?php echo esc_html( get_bloginfo( 'name' ) ); ?></p>
	<!-- /wp:paragraph -->

	<!-- wp:paragraph {"align":"center"} -->
	<p class="has-text-align-center">
		<a href="https://wordpress.org"><?php esc_html_e( 'Proudly powered by WordPress', 'mytheme' ); ?></a>
	</p>
	<!-- /wp:paragraph -->

	<!-- wp:social-links {"iconColor":"base","iconColorValue":"#ffffff","className":"is-style-logos-only","layout":{"type":"flex","justifyContent":"center"}} -->
	<ul class="wp-block-social-links has-icon-color is-style-logos-only">
		<!-- wp:social-link {"url":"#","service":"twitter"} /-->
		<!-- wp:social-link {"url":"#","service":"facebook"} /-->
		<!-- wp:social-link {"url":"#","service":"instagram"} /-->
	</ul>
	<!-- /wp:social-links -->
</div>
<!-- /wp:group -->
```

**wp_footer() equivalent:**

WordPress auto-calls this in block themes — no manual inclusion needed.

### single.php to templates/single.html

**Classic single.php:**

```php
<?php
get_header();
?>

<main id="primary" class="site-main">

	<?php
	while ( have_posts() ) :
		the_post();
		?>

		<article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
			<header class="entry-header">
				<?php the_title( '<h1 class="entry-title">', '</h1>' ); ?>

				<div class="entry-meta">
					<span class="posted-on"><?php echo esc_html( get_the_date() ); ?></span>
					<span class="byline">
						<?php
						printf(
							esc_html__( 'by %s', 'mytheme' ),
							'<a href="' . esc_url( get_author_posts_url( get_the_author_meta( 'ID' ) ) ) . '">' . esc_html( get_the_author() ) . '</a>'
						);
						?>
					</span>
				</div>
			</header>

			<?php
			if ( has_post_thumbnail() ) {
				the_post_thumbnail( 'large' );
			}
			?>

			<div class="entry-content">
				<?php the_content(); ?>
			</div>

			<footer class="entry-footer">
				<?php
				the_category( ', ' );
				the_tags( '<span class="tags-links">', ', ', '</span>' );
				?>
			</footer>
		</article>

		<?php
		if ( comments_open() || get_comments_number() ) {
			comments_template();
		}
	endwhile;
	?>

</main>

<?php
get_sidebar();
get_footer();
```

**Block theme templates/single.html:**

```html
<!-- wp:template-part {"slug":"header","area":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
	<!-- wp:post-title {"level":1} /-->

	<!-- wp:group {"layout":{"type":"flex","flexWrap":"nowrap"},"style":{"spacing":{"blockGap":"var:preset|spacing|30"}}} -->
	<div class="wp-block-group">
		<!-- wp:post-author {"showAvatar":true,"showBio":false} /-->
		<!-- wp:post-date /-->
	</div>
	<!-- /wp:group -->

	<!-- wp:post-featured-image {"align":"wide"} /-->

	<!-- wp:post-content {"layout":{"type":"constrained"}} /-->

	<!-- wp:group {"layout":{"type":"flex","flexWrap":"wrap"}} -->
	<div class="wp-block-group">
		<!-- wp:post-terms {"term":"category","prefix":"Posted in: "} /-->
		<!-- wp:post-terms {"term":"post_tag","prefix":"Tagged: "} /-->
	</div>
	<!-- /wp:group -->

	<!-- wp:comments -->
	<div class="wp-block-comments">
		<!-- wp:comments-title /-->
		<!-- wp:comment-template -->
			<!-- wp:comment-author-name /-->
			<!-- wp:comment-date /-->
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

**Template tag to block mapping:**

| Classic Template Tag | Block Equivalent |
|---------------------|------------------|
| `the_title()` | `wp:post-title` |
| `the_content()` | `wp:post-content` |
| `the_post_thumbnail()` | `wp:post-featured-image` |
| `get_the_date()` | `wp:post-date` |
| `get_the_author()` | `wp:post-author` |
| `the_category()` | `wp:post-terms {"term":"category"}` |
| `the_tags()` | `wp:post-terms {"term":"post_tag"}` |
| `comments_template()` | `wp:comments` |
| `get_header()` | `wp:template-part {"slug":"header"}` |
| `get_footer()` | `wp:template-part {"slug":"footer"}` |
| `get_sidebar()` | `wp:template-part {"slug":"sidebar"}` |

### archive.php to templates/archive.html

**Classic archive.php:**

```php
<?php
get_header();
?>

<main id="primary" class="site-main">

	<?php if ( have_posts() ) : ?>

		<header class="page-header">
			<?php
			the_archive_title( '<h1 class="page-title">', '</h1>' );
			the_archive_description( '<div class="archive-description">', '</div>' );
			?>
		</header>

		<?php
		while ( have_posts() ) :
			the_post();
			?>

			<article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
				<header class="entry-header">
					<?php the_title( '<h2 class="entry-title"><a href="' . esc_url( get_permalink() ) . '">', '</a></h2>' ); ?>
				</header>

				<?php
				if ( has_post_thumbnail() ) {
					?>
					<div class="post-thumbnail">
						<a href="<?php echo esc_url( get_permalink() ); ?>">
							<?php the_post_thumbnail(); ?>
						</a>
					</div>
					<?php
				}
				?>

				<div class="entry-excerpt">
					<?php the_excerpt(); ?>
				</div>
			</article>

		<?php endwhile; ?>

		<?php the_posts_pagination(); ?>

	<?php else : ?>

		<p><?php esc_html_e( 'No posts found.', 'mytheme' ); ?></p>

	<?php endif; ?>

</main>

<?php
get_sidebar();
get_footer();
```

**Block theme templates/archive.html:**

```html
<!-- wp:template-part {"slug":"header","area":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
	<!-- wp:query-title {"type":"archive"} /-->
	<!-- wp:term-description /-->

	<!-- wp:query {"query":{"perPage":10,"pages":0,"offset":0,"postType":"post","order":"desc","orderBy":"date","author":"","search":"","exclude":[],"sticky":"","inherit":true}} -->
	<div class="wp-block-query">
		<!-- wp:post-template {"layout":{"type":"grid","columnCount":3}} -->
			<!-- wp:post-featured-image {"isLink":true} /-->

			<!-- wp:post-title {"isLink":true} /-->

			<!-- wp:post-excerpt /-->

			<!-- wp:post-date /-->
		<!-- /wp:post-template -->

		<!-- wp:query-pagination -->
			<!-- wp:query-pagination-previous /-->
			<!-- wp:query-pagination-numbers /-->
			<!-- wp:query-pagination-next /-->
		<!-- /wp:query-pagination -->

		<!-- wp:query-no-results -->
			<!-- wp:paragraph -->
			<p><?php esc_html_e( 'No posts found.', 'mytheme' ); ?></p>
			<!-- /wp:paragraph -->
		<!-- /wp:query-no-results -->
	</div>
	<!-- /wp:query -->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","area":"footer"} /-->
```

**Loop to Query block mapping:**

| Classic Loop | Block Equivalent |
|--------------|------------------|
| `have_posts()` | `wp:query` with inherit:true |
| `while (have_posts())` | `wp:post-template` (loops automatically) |
| `the_post()` | Automatic in post-template context |
| `the_posts_pagination()` | `wp:query-pagination` |
| Archive title | `wp:query-title {"type":"archive"}` |
| Archive description | `wp:term-description` |
| No results message | `wp:query-no-results` |

**Cross-reference:** See wp-block-development skill for Query Loop block patterns.

## Sidebar to Template Part Migration

### Classic Sidebar Pattern

**functions.php:**

```php
<?php
function mytheme_widgets_init() {
	register_sidebar( array(
		'name'          => __( 'Sidebar', 'mytheme' ),
		'id'            => 'sidebar-1',
		'description'   => __( 'Add widgets here.', 'mytheme' ),
		'before_widget' => '<section id="%1$s" class="widget %2$s">',
		'after_widget'  => '</section>',
		'before_title'  => '<h2 class="widget-title">',
		'after_title'   => '</h2>',
	) );
}
add_action( 'widgets_init', 'mytheme_widgets_init' );
```

**sidebar.php:**

```php
<?php
if ( ! is_active_sidebar( 'sidebar-1' ) ) {
	return;
}
?>

<aside id="secondary" class="widget-area">
	<?php dynamic_sidebar( 'sidebar-1' ); ?>
</aside>
```

**Usage in templates:**

```php
<?php get_sidebar(); ?>
```

### Block Theme Equivalent

**No register_sidebar()** — Block themes don't use widget areas.

**parts/sidebar.html:**

```html
<!-- wp:group {"tagName":"aside","layout":{"type":"constrained"}} -->
<aside class="wp-block-group">
	<!-- wp:heading -->
	<h2><?php esc_html_e( 'Recent Posts', 'mytheme' ); ?></h2>
	<!-- /wp:heading -->

	<!-- wp:latest-posts {"postsToShow":5,"displayPostDate":true} /-->

	<!-- wp:heading -->
	<h2><?php esc_html_e( 'Categories', 'mytheme' ); ?></h2>
	<!-- /wp:heading -->

	<!-- wp:categories /-->

	<!-- wp:heading -->
	<h2><?php esc_html_e( 'Tags', 'mytheme' ); ?></h2>
	<!-- /wp:heading -->

	<!-- wp:tag-cloud /-->

	<!-- wp:heading -->
	<h2><?php esc_html_e( 'Search', 'mytheme' ); ?></h2>
	<!-- /wp:heading -->

	<!-- wp:search {"label":"Search","showLabel":false,"buttonText":"Search"} /-->
</aside>
<!-- /wp:group -->
```

**Usage in templates:**

```html
<!-- wp:template-part {"slug":"sidebar","area":"uncategorized"} /-->
```

**Widget to block mapping:**

| Classic Widget | Block Equivalent |
|----------------|------------------|
| Recent Posts widget | `wp:latest-posts` |
| Categories widget | `wp:categories` |
| Tag Cloud widget | `wp:tag-cloud` |
| Search widget | `wp:search` |
| Archives widget | `wp:archives` |
| Calendar widget | `wp:calendar` |
| RSS widget | `wp:rss` |
| Custom HTML widget | `wp:html` |

**Dynamic content in sidebar:**

Users can edit template parts in Site Editor, adding/removing blocks as needed.

## Menu to Navigation Block Migration

### Classic Menu Pattern

**functions.php:**

```php
<?php
function mytheme_menus() {
	register_nav_menus( array(
		'primary' => __( 'Primary Menu', 'mytheme' ),
		'footer'  => __( 'Footer Menu', 'mytheme' ),
	) );
}
add_action( 'after_setup_theme', 'mytheme_menus' );
```

**header.php:**

```php
<nav id="site-navigation" class="main-navigation">
	<?php
	wp_nav_menu( array(
		'theme_location' => 'primary',
		'menu_id'        => 'primary-menu',
	) );
	?>
</nav>
```

**Users manage:** Appearance > Menus

### Block Theme Equivalent

**No register_nav_menus()** — Block themes use Navigation block.

**parts/header.html:**

```html
<!-- wp:navigation {"layout":{"type":"flex","orientation":"horizontal"}} /-->
```

**Users manage:** Inline editing in Site Editor or template editor.

**Navigation block features:**

- Drag-and-drop menu items
- Add pages, posts, custom links, categories
- Nested submenus
- Mobile menu (overlay) configuration
- Styling via theme.json

**Default menu:**

If no menu exists, Navigation block shows Page List as fallback.

**Cross-reference:** See fse-guide.md for complete Navigation block configuration.

## Customizer to Global Styles Migration

### Classic Customizer Pattern

**functions.php:**

```php
<?php
function mytheme_customize_register( $wp_customize ) {
	// Add setting for header text color
	$wp_customize->add_setting( 'header_textcolor', array(
		'default'           => '#000000',
		'sanitize_callback' => 'sanitize_hex_color',
	) );

	// Add control for header text color
	$wp_customize->add_control( new WP_Customize_Color_Control( $wp_customize, 'header_textcolor', array(
		'label'    => __( 'Header Text Color', 'mytheme' ),
		'section'  => 'colors',
		'settings' => 'header_textcolor',
	) ) );

	// Add setting for primary color
	$wp_customize->add_setting( 'primary_color', array(
		'default'           => '#0073aa',
		'sanitize_callback' => 'sanitize_hex_color',
	) );

	$wp_customize->add_control( new WP_Customize_Color_Control( $wp_customize, 'primary_color', array(
		'label'    => __( 'Primary Color', 'mytheme' ),
		'section'  => 'colors',
		'settings' => 'primary_color',
	) ) );
}
add_action( 'customize_register', 'mytheme_customize_register' );
```

**Output in style.css:**

```php
<?php
function mytheme_customizer_css() {
	$header_textcolor = get_theme_mod( 'header_textcolor', '#000000' );
	$primary_color    = get_theme_mod( 'primary_color', '#0073aa' );
	?>
	<style type="text/css">
		.site-title {
			color: <?php echo esc_attr( $header_textcolor ); ?>;
		}
		.button {
			background-color: <?php echo esc_attr( $primary_color ); ?>;
		}
	</style>
	<?php
}
add_action( 'wp_head', 'mytheme_customizer_css' );
```

**Users manage:** Appearance > Customize

### Block Theme Equivalent

**No Customizer code** — Use theme.json and Global Styles.

**theme.json:**

```json
{
	"version": 3,
	"settings": {
		"color": {
			"palette": [
				{
					"slug": "primary",
					"color": "#0073aa",
					"name": "Primary"
				},
				{
					"slug": "header-text",
					"color": "#000000",
					"name": "Header Text"
				}
			]
		}
	},
	"styles": {
		"elements": {
			"button": {
				"color": {
					"background": "var(--wp--preset--color--primary)"
				}
			}
		}
	}
}
```

**Users manage:** Appearance > Editor > Styles

**User changes:**

When user changes Primary color via Global Styles UI, WordPress saves to wp_global_styles post and applies site-wide.

**Migration tip:** Export Customizer values, add to theme.json palette as default colors.

## Hybrid Theme Pattern (Incremental Adoption)

### What is a Hybrid Theme?

**Definition:** Classic theme (PHP templates) with theme.json for block editor settings.

**File structure:**

```
hybrid-theme/
├── style.css              # Required
├── functions.php          # Classic setup
├── theme.json             # Block editor settings
├── index.php              # Classic templates
├── single.php
├── header.php
└── footer.php
```

**Benefits:**

- Existing classic templates keep working
- Block editor gets modern controls
- Gradual migration path
- Backwards compatible

**Limitations:**

- theme.json only affects block editor, not classic template output
- Dual system complexity
- Some features only work in full block themes

### Hybrid Theme theme.json

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
				{"slug": "medium", "size": "1rem", "name": "Medium"}
			]
		},
		"layout": {
			"contentSize": "640px",
			"wideSize": "1200px"
		}
	}
}
```

**What this provides:**

- Color palette in block editor
- Font size options in block editor
- Wide/full alignment in editor

**What it doesn't do:**

- Change classic template output
- Apply colors to header.php/footer.php
- Replace Customizer

**Making theme.json affect classic templates:**

```php
<?php
function mytheme_enqueue_styles() {
	// Enqueue theme styles
	wp_enqueue_style( 'mytheme-style', get_stylesheet_uri() );

	// Add inline CSS using theme.json CSS variables
	$custom_css = "
		.site-header {
			background-color: var(--wp--preset--color--primary);
		}
		.button {
			background-color: var(--wp--preset--color--secondary);
		}
	";
	wp_add_inline_style( 'mytheme-style', $custom_css );
}
add_action( 'wp_enqueue_scripts', 'mytheme_enqueue_styles' );
```

### Progressive Migration Steps

**Phase 1: Add theme.json**

1. Create theme.json with settings
2. Test block editor settings
3. Keep all classic templates

**Phase 2: Convert template parts**

1. Convert header.php → parts/header.html
2. Convert footer.php → parts/footer.html
3. Use `wp:template-part` in classic templates (requires add_theme_support('block-template-parts'))

**Phase 3: Convert main templates**

1. Convert index.php → templates/index.html
2. Convert single.php → templates/single.html
3. Convert archive.php → templates/archive.html
4. Test each template conversion

**Phase 4: Remove classic templates**

1. Delete all .php templates
2. Verify templates/ directory covers all cases
3. Remove functions.php hooks replaced by theme.json
4. Now full block theme

## Common Migration Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Missing useRootPaddingAwareAlignments** | Full-width blocks don't reach edges | Add `"useRootPaddingAwareAlignments": true` with root padding |
| **Hardcoded hex values in templates** | Users can't customize colors | Define in theme.json palette, use CSS variables |
| **Expecting theme.json to affect classic templates** | Hybrid theme classic templates unchanged | Manually apply CSS variables in enqueued styles |
| **Missing defaultFontSizes: false** | Duplicate font sizes after v3 upgrade | Explicitly set `"defaultFontSizes": false` |
| **Template part name with .html extension** | Template part not found | Use `"header"` not `"header.html"` in theme.json |
| **Navigation block with no fallback** | Empty header if no menu created | Navigation auto-falls back to Page List |
| **Over-locking blocks in templates** | Users can't edit templates | Use block locking sparingly |
| **Not testing with Theme Check** | WordPress.org rejection | Run Theme Check plugin before submission |

## Create Block Theme Plugin Workflow

### Using the Plugin

**Install:** Plugins > Add New > Search "Create Block Theme"

**Workflow:**

1. **Start with classic theme**
2. **Activate theme**
3. **Use Site Editor to customize** (Appearance > Editor)
   - Edit templates visually
   - Customize styles
   - Create patterns
4. **Tools > Create Block Theme > Create Child Theme**
5. **Export generates:**
   - theme.json (from Global Styles)
   - templates/*.html (from edited templates)
   - parts/*.html (from template parts)
   - patterns/*.php (from created patterns)
6. **Review and clean up** generated files
7. **Activate new block theme**

**Benefits:**

- Preserves existing customizations
- Generates valid block theme structure
- Exports user Global Styles
- Creates patterns from variations

**Limitations:**

- Generated code may need cleanup
- Complex PHP logic doesn't convert
- Manual review required

## Cross-References

- **theme.json v3 complete reference:** See theme-json-guide.md
- **Template structure and hierarchy:** See template-patterns.md
- **Full Site Editing features:** See fse-guide.md
- **Block development patterns:** See wp-block-development skill
- **Template security and escaping:** See wp-security-review skill
- **Query Loop optimization:** See wp-performance-review skill
