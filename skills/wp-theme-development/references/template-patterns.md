# WordPress Template Hierarchy and Patterns Reference

This reference catalogs the complete WordPress template hierarchy for both block themes (HTML templates) and classic themes (PHP templates), template parts, conditional tags, and template selection logic. Each pattern includes block markup examples for block themes and PHP template tag examples for classic themes following WordPress PHP Coding Standards.

## Quick Reference Table

### Template Hierarchy Side-by-Side

| Purpose | Classic Theme (.php) | Block Theme (.html) | Fallback Chain |
|---------|---------------------|---------------------|----------------|
| **Homepage (blog)** | `home.php` | `templates/home.html` | index.php / index.html |
| **Static front page** | `front-page.php` | `templates/front-page.html` | home.php > index.php |
| **Single post** | `single-{post-type}-{slug}.php`<br>`single-{post-type}.php`<br>`single.php` | `templates/single-{post-type}-{slug}.html`<br>`templates/single-{post-type}.html`<br>`templates/single.html` | singular.php > index.php |
| **Single page** | `page-{slug}.php`<br>`page-{id}.php`<br>`page.php` | `templates/page-{slug}.html`<br>`templates/page-{id}.html`<br>`templates/page.html` | singular.php > index.php |
| **Category archive** | `category-{slug}.php`<br>`category-{id}.php`<br>`category.php` | `templates/category-{slug}.html`<br>`templates/category-{id}.html`<br>`templates/category.html` | archive.php > index.php |
| **Tag archive** | `tag-{slug}.php`<br>`tag-{id}.php`<br>`tag.php` | `templates/tag-{slug}.html`<br>`templates/tag-{id}.html`<br>`templates/tag.html` | archive.php > index.php |
| **Author archive** | `author-{nicename}.php`<br>`author-{id}.php`<br>`author.php` | `templates/author-{nicename}.html`<br>`templates/author-{id}.html`<br>`templates/author.html` | archive.php > index.php |
| **Date archive** | `date.php` | `templates/date.html` | archive.php > index.php |
| **Custom post type archive** | `archive-{post-type}.php` | `templates/archive-{post-type}.html` | archive.php > index.php |
| **Taxonomy archive** | `taxonomy-{taxonomy}-{term}.php`<br>`taxonomy-{taxonomy}.php`<br>`taxonomy.php` | `templates/taxonomy-{taxonomy}-{term}.html`<br>`templates/taxonomy-{taxonomy}.html`<br>`templates/taxonomy.html` | archive.php > index.php |
| **Search results** | `search.php` | `templates/search.html` | index.php / index.html |
| **404 error** | `404.php` | `templates/404.html` | index.php / index.html |
| **Attachment** | `attachment.php` | `templates/attachment.html` | single.php > singular.php > index.php |
| **Embed template** | `embed.php` | `templates/embed.html` | index.php / index.html |

## Block Theme Template Structure

### Required Files

**Minimal block theme:**

```
block-theme/
├── style.css              # Required: Theme metadata
├── theme.json             # Required: Settings and styles
└── templates/
    └── index.html         # Required: Fallback template
```

**Complete block theme:**

```
block-theme/
├── style.css
├── theme.json
├── functions.php          # Optional: Hooks and setup
├── templates/
│   ├── index.html         # Fallback
│   ├── home.html          # Blog homepage
│   ├── front-page.html    # Static front page
│   ├── single.html        # Single post
│   ├── page.html          # Single page
│   ├── archive.html       # Archives
│   ├── category.html      # Category archives
│   ├── tag.html           # Tag archives
│   ├── author.html        # Author archives
│   ├── date.html          # Date archives
│   ├── search.html        # Search results
│   ├── 404.html           # 404 error
│   └── singular.html      # Fallback for single post/page
└── parts/
    ├── header.html        # Header template part
    ├── footer.html        # Footer template part
    └── sidebar.html       # Sidebar template part
```

### Block Template Anatomy

**templates/single.html:**

```html
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

**Key blocks for templates:**

| Block | Purpose | Usage |
|-------|---------|-------|
| `wp:template-part` | Include template part | `{"slug":"header","area":"header"}` |
| `wp:post-title` | Display post title | Auto-populated from post |
| `wp:post-content` | Display post content | The Loop equivalent |
| `wp:post-featured-image` | Display featured image | Conditional on image existence |
| `wp:post-author` | Display post author | With avatar, bio options |
| `wp:post-date` | Display post date | Configurable format |
| `wp:post-terms` | Display taxonomy terms | `{"term":"category"}` or `{"term":"post_tag"}` |
| `wp:comments` | Display comments section | Contains comment-template |
| `wp:query` | Custom post query loop | Archive template queries |
| `wp:site-title` | Display site title | From Settings > General |
| `wp:site-logo` | Display site logo | From Site Editor |
| `wp:navigation` | Display navigation menu | User-editable in Site Editor |

### Block Markup in Templates

**BAD: Inline styles in template**

```html
<!-- BAD: Hardcoded styles prevent user customization -->
<div style="background-color: #333; padding: 2rem;">
	<!-- wp:post-title /-->
</div>
```

**GOOD: Block attributes using theme.json presets**

```html
<!-- GOOD: References theme.json settings, user-customizable -->
<!-- wp:group {"style":{"color":{"background":"var(--wp--preset--color--primary)"},"spacing":{"padding":"var(--wp--preset--spacing--50)"}}} -->
<div class="wp-block-group">
	<!-- wp:post-title /-->
</div>
<!-- /wp:group -->
```

**GOOD: Class-based styling via theme.json**

```html
<!-- GOOD: Apply styles via theme.json blocks configuration -->
<!-- wp:group {"className":"hero-section"} -->
<div class="wp-block-group hero-section">
	<!-- wp:post-title /-->
</div>
<!-- /wp:group -->
```

Then in theme.json:

```json
{
	"styles": {
		"blocks": {
			"core/group": {
				"variations": {
					"hero-section": {
						"color": {
							"background": "var(--wp--preset--color--primary)"
						}
					}
				}
			}
		}
	}
}
```

## Classic Theme Template Structure

### Required Files

**Minimal classic theme:**

```
classic-theme/
├── style.css              # Required: Theme metadata
├── index.php              # Required: Fallback template
├── screenshot.png         # Required for WordPress.org
└── functions.php          # Common: Theme setup
```

**Complete classic theme:**

```
classic-theme/
├── style.css
├── functions.php
├── index.php              # Fallback
├── home.php               # Blog homepage
├── front-page.php         # Static front page
├── single.php             # Single post
├── single-{post-type}.php # Custom post type single
├── page.php               # Single page
├── archive.php            # Archives
├── category.php           # Category archives
├── tag.php                # Tag archives
├── author.php             # Author archives
├── date.php               # Date archives
├── search.php             # Search results
├── 404.php                # 404 error
├── header.php             # Header template part
├── footer.php             # Footer template part
├── sidebar.php            # Sidebar template part
├── comments.php           # Comments template
└── template-parts/        # Reusable components
    ├── content.php        # Post content partial
    └── content-page.php   # Page content partial
```

### Classic Template Anatomy

**single.php:**

```php
<?php
/**
 * Template for displaying single posts
 */

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
					<span class="posted-on">
						<?php echo esc_html( get_the_date() ); ?>
					</span>
					<span class="byline">
						<?php
						/* translators: %s: post author */
						printf( esc_html__( 'by %s', 'mytheme' ), '<a href="' . esc_url( get_author_posts_url( get_the_author_meta( 'ID' ) ) ) . '">' . esc_html( get_the_author() ) . '</a>' );
						?>
					</span>
				</div>
			</header>

			<?php if ( has_post_thumbnail() ) : ?>
				<div class="post-thumbnail">
					<?php the_post_thumbnail( 'large' ); ?>
				</div>
			<?php endif; ?>

			<div class="entry-content">
				<?php
				the_content();

				wp_link_pages( array(
					'before' => '<div class="page-links">' . esc_html__( 'Pages:', 'mytheme' ),
					'after'  => '</div>',
				) );
				?>
			</div>

			<footer class="entry-footer">
				<?php
				$categories_list = get_the_category_list( esc_html__( ', ', 'mytheme' ) );
				if ( $categories_list ) {
					/* translators: %s: list of categories */
					printf( '<span class="cat-links">' . esc_html__( 'Posted in %s', 'mytheme' ) . '</span>', $categories_list );
				}

				$tags_list = get_the_tag_list( '', esc_html_x( ', ', 'list item separator', 'mytheme' ) );
				if ( $tags_list ) {
					/* translators: %s: list of tags */
					printf( '<span class="tags-links">' . esc_html__( 'Tagged %s', 'mytheme' ) . '</span>', $tags_list );
				}
				?>
			</footer>
		</article>

		<?php
		if ( comments_open() || get_comments_number() ) {
			comments_template();
		}

		the_post_navigation( array(
			'prev_text' => '<span class="nav-subtitle">' . esc_html__( 'Previous:', 'mytheme' ) . '</span> <span class="nav-title">%title</span>',
			'next_text' => '<span class="nav-subtitle">' . esc_html__( 'Next:', 'mytheme' ) . '</span> <span class="nav-title">%title</span>',
		) );

	endwhile;
	?>

</main>

<?php
get_sidebar();
get_footer();
```

**CRITICAL: All output must be escaped**

```php
// BAD: Unescaped output (XSS vulnerability)
<h1><?php echo get_the_title(); ?></h1>

// GOOD: Escaped output
<h1><?php echo esc_html( get_the_title() ); ?></h1>
```

**Cross-reference:** See wp-security-review skill for complete escaping guide.

### The Loop in Classic Themes

**Standard Loop:**

```php
<?php
if ( have_posts() ) :
	while ( have_posts() ) :
		the_post();

		// Template content here
		the_title( '<h2>', '</h2>' );
		the_content();

	endwhile;

	the_posts_pagination();

else :

	get_template_part( 'template-parts/content', 'none' );

endif;
?>
```

**Custom Loop with WP_Query:**

```php
<?php
$args = array(
	'post_type'      => 'post',
	'posts_per_page' => 10,
	'orderby'        => 'date',
	'order'          => 'DESC',
);

$query = new WP_Query( $args );

if ( $query->have_posts() ) :
	while ( $query->have_posts() ) :
		$query->the_post();

		// Template content here

	endwhile;
	wp_reset_postdata();
else :
	// No posts found
endif;
?>
```

**Cross-reference:** See wp-performance-review skill for WP_Query optimization patterns.

### Common Template Tags

| Function | Purpose | Escaping Required |
|----------|---------|-------------------|
| `the_title()` | Output post title | Yes (esc_html) |
| `the_content()` | Output post content | No (already escaped) |
| `the_excerpt()` | Output post excerpt | No (already escaped) |
| `the_permalink()` | Output post URL | Yes (esc_url) |
| `the_author()` | Output author name | Yes (esc_html) |
| `the_date()` | Output post date | Yes (esc_html) |
| `the_category()` | Output categories | No (already escaped) |
| `the_tags()` | Output tags | No (already escaped) |
| `the_post_thumbnail()` | Output featured image | No (already escaped) |
| `get_header()` | Include header.php | N/A |
| `get_footer()` | Include footer.php | N/A |
| `get_sidebar()` | Include sidebar.php | N/A |
| `get_template_part()` | Include template part | N/A |
| `comments_template()` | Include comments.php | N/A |

## Template Hierarchy Fallback Chain

### Front Page Display

**Settings > Reading > "Your homepage displays"**

**Option 1: Latest posts**

```
front-page.php → home.php → index.php
```

**Option 2: Static page**

Front page:
```
front-page.php → page-{slug}.php → page-{id}.php → page.php → singular.php → index.php
```

Blog page:
```
home.php → index.php
```

### Single Post Hierarchy

```
single-{post-type}-{slug}.php
→ single-{post-type}.php
→ single.php
→ singular.php
→ index.php
```

**Example for post type "product" with slug "widget":**

Classic theme:
```
single-product-widget.php
→ single-product.php
→ single.php
→ singular.php
→ index.php
```

Block theme:
```
templates/single-product-widget.html
→ templates/single-product.html
→ templates/single.html
→ templates/singular.html
→ templates/index.html
```

### Page Hierarchy

```
page-{slug}.php
→ page-{id}.php
→ page.php
→ singular.php
→ index.php
```

**Custom page templates:**

Classic theme:
```php
<?php
/**
 * Template Name: Full Width Page
 * Template Post Type: page
 */

get_header();
// Template content
get_footer();
```

Block theme:

In theme.json:
```json
{
	"customTemplates": [
		{
			"name": "page-full-width",
			"title": "Full Width Page",
			"postTypes": ["page"]
		}
	]
}
```

Create: `templates/page-full-width.html`

### Archive Hierarchy

**Category archives:**
```
category-{slug}.php → category-{id}.php → category.php → archive.php → index.php
```

**Tag archives:**
```
tag-{slug}.php → tag-{id}.php → tag.php → archive.php → index.php
```

**Author archives:**
```
author-{nicename}.php → author-{id}.php → author.php → archive.php → index.php
```

**Date archives:**
```
date.php → archive.php → index.php
```

**Custom post type archives:**
```
archive-{post-type}.php → archive.php → index.php
```

**Taxonomy archives:**
```
taxonomy-{taxonomy}-{term}.php → taxonomy-{taxonomy}.php → taxonomy.php → archive.php → index.php
```

## Template Parts Deep Dive

### Block Theme Template Parts

**parts/header.html:**

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

**parts/footer.html:**

```html
<!-- wp:group {"align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|50","bottom":"var:preset|spacing|50"}}},"backgroundColor":"contrast","textColor":"base","layout":{"type":"constrained"}} -->
<div class="wp-block-group alignfull has-contrast-background-color has-base-color has-text-color has-background">
	<!-- wp:group {"layout":{"type":"flex","justifyContent":"space-between","flexWrap":"wrap"}} -->
	<div class="wp-block-group">
		<!-- wp:paragraph -->
		<p>&copy; <?php echo esc_html( date( 'Y' ) ); ?> <?php echo esc_html( get_bloginfo( 'name' ) ); ?></p>
		<!-- /wp:paragraph -->

		<!-- wp:social-links {"iconColor":"base","iconColorValue":"#ffffff","className":"is-style-logos-only"} -->
		<ul class="wp-block-social-links has-icon-color is-style-logos-only">
			<!-- wp:social-link {"url":"#","service":"twitter"} /-->
			<!-- wp:social-link {"url":"#","service":"facebook"} /-->
		</ul>
		<!-- /wp:social-links -->
	</div>
	<!-- /wp:group -->
</div>
<!-- /wp:group -->
```

**Template part registration in theme.json:**

```json
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

**Using template parts in templates:**

```html
<!-- wp:template-part {"slug":"header","area":"header"} /-->

<!-- Main content here -->

<!-- wp:template-part {"slug":"footer","area":"footer"} /-->
```

### Classic Theme Template Parts

**header.php:**

```php
<!DOCTYPE html>
<html <?php language_attributes(); ?>>
<head>
	<meta charset="<?php bloginfo( 'charset' ); ?>">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<link rel="profile" href="https://gmpg.org/xfn/11">
	<?php wp_head(); ?>
</head>

<body <?php body_class(); ?>>
<?php wp_body_open(); ?>
<div id="page" class="site">
	<a class="skip-link screen-reader-text" href="#primary"><?php esc_html_e( 'Skip to content', 'mytheme' ); ?></a>

	<header id="masthead" class="site-header">
		<div class="site-branding">
			<?php
			if ( has_custom_logo() ) {
				the_custom_logo();
			}
			?>
			<h1 class="site-title"><a href="<?php echo esc_url( home_url( '/' ) ); ?>" rel="home"><?php bloginfo( 'name' ); ?></a></h1>
			<?php
			$description = get_bloginfo( 'description', 'display' );
			if ( $description || is_customize_preview() ) :
				?>
				<p class="site-description"><?php echo esc_html( $description ); ?></p>
			<?php endif; ?>
		</div>

		<nav id="site-navigation" class="main-navigation">
			<button class="menu-toggle" aria-controls="primary-menu" aria-expanded="false"><?php esc_html_e( 'Menu', 'mytheme' ); ?></button>
			<?php
			wp_nav_menu( array(
				'theme_location' => 'primary',
				'menu_id'        => 'primary-menu',
			) );
			?>
		</nav>
	</header>
```

**footer.php:**

```php
	<footer id="colophon" class="site-footer">
		<div class="site-info">
			<a href="<?php echo esc_url( __( 'https://wordpress.org/', 'mytheme' ) ); ?>">
				<?php
				/* translators: %s: CMS name (WordPress) */
				printf( esc_html__( 'Proudly powered by %s', 'mytheme' ), 'WordPress' );
				?>
			</a>
			<span class="sep"> | </span>
			<?php
			/* translators: 1: Theme name, 2: Theme author */
			printf( esc_html__( 'Theme: %1$s by %2$s.', 'mytheme' ), 'mytheme', '<a href="https://example.com/">Author Name</a>' );
			?>
		</div>
	</footer>
</div><!-- #page -->

<?php wp_footer(); ?>

</body>
</html>
```

**sidebar.php:**

```php
<?php
/**
 * The sidebar containing the main widget area
 */

if ( ! is_active_sidebar( 'sidebar-1' ) ) {
	return;
}
?>

<aside id="secondary" class="widget-area">
	<?php dynamic_sidebar( 'sidebar-1' ); ?>
</aside>
```

**get_template_part() usage:**

```php
<?php
// Load template-parts/content.php
get_template_part( 'template-parts/content' );

// Load template-parts/content-page.php
get_template_part( 'template-parts/content', 'page' );

// Load template-parts/content-{post-format}.php
get_template_part( 'template-parts/content', get_post_format() );
?>
```

**template-parts/content.php:**

```php
<?php
/**
 * Template part for displaying posts
 */
?>

<article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
	<header class="entry-header">
		<?php
		if ( is_singular() ) :
			the_title( '<h1 class="entry-title">', '</h1>' );
		else :
			the_title( '<h2 class="entry-title"><a href="' . esc_url( get_permalink() ) . '" rel="bookmark">', '</a></h2>' );
		endif;
		?>
	</header>

	<?php if ( has_post_thumbnail() ) : ?>
		<div class="post-thumbnail">
			<a href="<?php echo esc_url( get_permalink() ); ?>">
				<?php the_post_thumbnail(); ?>
			</a>
		</div>
	<?php endif; ?>

	<div class="entry-content">
		<?php
		the_excerpt();
		?>
	</div>
</article>
```

## Conditional Tags (Classic Themes)

### Page Type Detection

| Function | Returns true when |
|----------|-------------------|
| `is_front_page()` | On the front page (static page or posts page) |
| `is_home()` | On the blog posts index page |
| `is_single()` | On a single post page |
| `is_single( 'post-slug' )` | On specific single post |
| `is_single( array( 'slug1', 'slug2' ) )` | On any of specified posts |
| `is_singular()` | On any single post or page |
| `is_singular( 'page' )` | On any single page |
| `is_singular( 'product' )` | On any single product post |
| `is_page()` | On any page |
| `is_page( 'about' )` | On specific page (slug, ID, or title) |
| `is_page_template( 'template-full-width.php' )` | Using specific page template |
| `is_archive()` | On any archive page |
| `is_category()` | On category archive |
| `is_category( 'news' )` | On specific category |
| `is_tag()` | On tag archive |
| `is_tag( 'featured' )` | On specific tag |
| `is_author()` | On author archive |
| `is_author( 'john' )` | On specific author archive |
| `is_date()` | On date archive |
| `is_year()` | On yearly archive |
| `is_month()` | On monthly archive |
| `is_day()` | On daily archive |
| `is_tax()` | On custom taxonomy archive |
| `is_tax( 'product_cat' )` | On specific taxonomy |
| `is_post_type_archive( 'product' )` | On custom post type archive |
| `is_search()` | On search results page |
| `is_404()` | On 404 error page |
| `is_attachment()` | On attachment page |
| `is_paged()` | On paginated page (page 2+) |

### Usage Examples

```php
<?php
// Conditional sidebar
if ( is_active_sidebar( 'sidebar-1' ) && ! is_page_template( 'template-full-width.php' ) ) {
	get_sidebar();
}

// Conditional content
if ( is_front_page() && is_home() ) {
	// Default homepage (blog posts)
} elseif ( is_front_page() ) {
	// Static front page
} elseif ( is_home() ) {
	// Blog posts page
}

// Custom query based on page type
if ( is_single() && in_category( 'featured' ) ) {
	// Show related posts for featured category
}
?>
```

## Custom Templates

### Block Theme Custom Templates

**In theme.json:**

```json
{
	"customTemplates": [
		{
			"name": "page-full-width",
			"title": "Full Width Page",
			"postTypes": ["page"]
		},
		{
			"name": "page-no-title",
			"title": "Page Without Title",
			"postTypes": ["page"]
		},
		{
			"name": "single-portfolio",
			"title": "Portfolio Single",
			"postTypes": ["portfolio"]
		}
	]
}
```

**Create template file:**

`templates/page-full-width.html`:

```html
<!-- wp:template-part {"slug":"header","area":"header"} /-->

<!-- wp:group {"tagName":"main","align":"full","layout":{"type":"constrained","contentSize":"100%"}} -->
<main class="wp-block-group alignfull">
	<!-- wp:post-title {"level":1} /-->
	<!-- wp:post-content {"layout":{"type":"constrained","contentSize":"100%"}} /-->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","area":"footer"} /-->
```

### Classic Theme Custom Templates

**Template file with header:**

`template-full-width.php`:

```php
<?php
/**
 * Template Name: Full Width Page
 * Template Post Type: page, post
 */

get_header();
?>

<main id="primary" class="site-main full-width">

	<?php
	while ( have_posts() ) :
		the_post();
		?>

		<article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
			<header class="entry-header">
				<?php the_title( '<h1 class="entry-title">', '</h1>' ); ?>
			</header>

			<div class="entry-content">
				<?php the_content(); ?>
			</div>
		</article>

	<?php endwhile; ?>

</main>

<?php
get_footer();
```

**Template selection:**

Users select custom template in the page/post editor sidebar under "Template".

## Template Part Areas

**Available areas in theme.json:**

- `header` — Header area (typically site-wide header)
- `footer` — Footer area (typically site-wide footer)
- `uncategorized` — General area (sidebar, content sections)

**Custom areas (WordPress 6.0+):**

Can define custom areas, but WordPress doesn't provide UI labels for them.

**Best practice:** Stick to `header`, `footer`, `uncategorized` for theme directory submission.

## Anti-Patterns

### BAD: Hardcoded Navigation in Block Templates

```html
<!-- BAD: Hardcoded menu items -->
<!-- wp:group -->
<div class="wp-block-group">
	<a href="/about">About</a>
	<a href="/contact">Contact</a>
</div>
<!-- /wp:group -->
```

**GOOD: Navigation block**

```html
<!-- GOOD: User-editable navigation -->
<!-- wp:navigation {"layout":{"type":"flex","orientation":"horizontal"}} /-->
```

### BAD: Missing Escaping in Classic Templates

```php
// BAD: Unescaped output
<h1><?php echo get_the_title(); ?></h1>
<a href="<?php echo get_permalink(); ?>">Read more</a>
<div class="author"><?php echo get_the_author(); ?></div>
```

**GOOD: Properly escaped**

```php
// GOOD: All output escaped
<h1><?php echo esc_html( get_the_title() ); ?></h1>
<a href="<?php echo esc_url( get_permalink() ); ?>"><?php esc_html_e( 'Read more', 'mytheme' ); ?></a>
<div class="author"><?php echo esc_html( get_the_author() ); ?></div>
```

### BAD: Direct Database Queries in Templates

```php
// BAD: Direct $wpdb usage in template
global $wpdb;
$results = $wpdb->get_results( "SELECT * FROM {$wpdb->posts} WHERE post_type = 'post'" );
```

**GOOD: Use WP_Query**

```php
// GOOD: Use WordPress APIs
$query = new WP_Query( array(
	'post_type'      => 'post',
	'posts_per_page' => 10,
) );
```

### BAD: Loading All Template Parts Unconditionally

```php
// BAD: Always loads sidebar even when not needed
get_header();
get_sidebar();
get_footer();
```

**GOOD: Conditional template parts**

```php
// GOOD: Conditional sidebar loading
get_header();

if ( is_active_sidebar( 'sidebar-1' ) && ! is_page_template( 'template-full-width.php' ) ) {
	get_sidebar();
}

get_footer();
```

## Cross-References

- **theme.json configuration:** See theme-json-guide.md
- **Full Site Editing features:** See fse-guide.md
- **Classic to block theme migration:** See classic-to-block-guide.md
- **Template security and escaping:** See wp-security-review skill
- **Block markup in templates:** See wp-block-development skill
- **WP_Query optimization:** See wp-performance-review skill
