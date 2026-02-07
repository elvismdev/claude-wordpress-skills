# WordPress block.json Schema Reference

This reference provides complete coverage of the block.json metadata file — the single source of truth for WordPress block registration (WordPress 5.8+). Every field, validation rule, apiVersion difference, and common mistake is documented with examples following WordPress coding standards.

## Quick Reference Table

### Metadata Fields

| Field | Type | Required | Description | Example Value |
|-------|------|----------|-------------|---------------|
| `$schema` | string | No | JSON schema URL for IDE validation | `"https://schemas.wp.org/trunk/block.json"` |
| `apiVersion` | number | Yes | Block API version (1, 2, or 3) | `3` |
| `name` | string | Yes | Block identifier (namespace/name) | `"my-plugin/custom-block"` |
| `title` | string | Yes | Human-readable title (translatable) | `"Custom Block"` |
| `category` | string | Yes | Block category slug | `"widgets"` |
| `icon` | string/object | No | Dashicon slug or custom SVG | `"smiley"` |
| `description` | string | No | Brief description (translatable) | `"A custom block example"` |
| `keywords` | array | No | Search keywords (max 3) | `["custom", "example"]` |
| `version` | string | No | Block version | `"1.0.0"` |
| `textdomain` | string | No | Translation domain | `"my-plugin"` |
| `attributes` | object | No | Attribute schema | See Attributes section |
| `providesContext` | object | No | Context provided to inner blocks | `{"myPlugin/userId": "userId"}` |
| `usesContext` | array | No | Context consumed from parent | `["postId", "postType"]` |

### Asset Fields

| Field | Type | Required | Description | Example Value |
|-------|------|----------|-------------|---------------|
| `editorScript` | string/array | No | Editor JavaScript asset | `"file:./index.js"` |
| `script` | string/array | No | Frontend JavaScript asset | `"file:./script.js"` |
| `viewScript` | string/array | No | Frontend JavaScript (legacy) | `"file:./view.js"` |
| `viewScriptModule` | string/array | No | Frontend ES module (WP 6.5+) | `"file:./view.js"` |
| `editorStyle` | string/array | No | Editor CSS asset | `"file:./editor.css"` |
| `style` | string/array | No | Frontend + editor CSS | `"file:./style.css"` |

### Behavior Fields

| Field | Type | Required | Description | Example Value |
|-------|------|----------|-------------|---------------|
| `supports` | object | No | Block supports configuration | See Supports section |
| `parent` | array | No | Restrict to parent block types | `["core/group", "core/column"]` |
| `ancestor` | array | No | Restrict to ancestor blocks | `["core/post-content"]` |
| `example` | object | No | Block preview for inserter | `{"attributes": {"content": "Preview"}}` |
| `variations` | array | No | Block variations | See Variations section |
| `blockHooks` | object | No | Auto-insert configuration (WP 6.4+) | `{"core/post-content": "after"}` |
| `render` | string | No | Render PHP file path | `"file:./render.php"` |

## Schema and Validation

### $schema Property for IDE Support

The `$schema` property enables IDE autocomplete and validation:

```json
{
	"$schema": "https://schemas.wp.org/trunk/block.json",
	"apiVersion": 3,
	"name": "my-plugin/my-block"
}
```

**Why use it:**
- VS Code shows autocomplete for all fields
- Real-time validation catches typos
- Hover tooltips show documentation
- Schema hosted by WordPress.org, always up-to-date

### WordPress Validation Rules

WordPress validates block.json when `register_block_type()` is called:

1. **Required fields:** `apiVersion`, `name`, `title`, `category` must be present
2. **Type validation:** apiVersion must be number, not string
3. **Name format:** Must match `namespace/block-name` pattern (lowercase, hyphens)
4. **Asset paths:** `file:` prefix required for relative paths
5. **Attribute types:** Must be valid JSON schema types

**Common validation errors:**

```json
// BAD: apiVersion as string
{
	"apiVersion": "3"
}
// Error: apiVersion must be a number

// BAD: Invalid name format
{
	"name": "MyBlock"
}
// Error: Block name must include namespace/block-name

// BAD: Missing required fields
{
	"apiVersion": 3,
	"name": "my-plugin/block"
}
// Error: title and category are required

// BAD: Relative path without file: prefix
{
	"editorScript": "./index.js"
}
// Warning: Asset path should use file: prefix
```

**GOOD: Complete validation-passing block.json**

```json
{
	"$schema": "https://schemas.wp.org/trunk/block.json",
	"apiVersion": 3,
	"name": "my-plugin/example-block",
	"title": "Example Block",
	"category": "widgets",
	"icon": "smiley",
	"description": "A properly configured block",
	"keywords": ["example", "demo"],
	"version": "1.0.0",
	"textdomain": "my-plugin",
	"editorScript": "file:./index.js",
	"style": "file:./style-index.css"
}
```

## apiVersion Deep Dive

The `apiVersion` field determines block behavior and wrapper handling.

### apiVersion 1 (Legacy)

**Characteristics:**
- Manual wrapper management required
- `useBlockProps()` not required
- Editor and frontend can have different markup
- Automatic class name injection via `blocks.getSaveContent.extraProps` filter

```javascript
// apiVersion 1 pattern (OUTDATED)
// block.json
{
	"apiVersion": 1,
	"name": "my-plugin/legacy-block"
}

// edit.js - Manual wrapper
function Edit( { attributes, setAttributes, className } ) {
	return (
		<div className={ className }>
			<p>Content</p>
		</div>
	);
}

// save.js - Manual wrapper
function save( { attributes } ) {
	return (
		<div>
			<p>Content</p>
		</div>
	);
}
```

**Do NOT use apiVersion 1** — deprecated pattern, no longer recommended.

### apiVersion 2 (Standard)

**Characteristics:**
- `useBlockProps()` required in edit and save
- Editor and frontend share same wrapper structure
- Automatic class name injection in save
- Default for most blocks pre-WordPress 6.3

```javascript
// apiVersion 2 pattern
// block.json
{
	"apiVersion": 2,
	"name": "my-plugin/standard-block"
}

// edit.js
import { useBlockProps } from '@wordpress/block-editor';

function Edit( { attributes, setAttributes } ) {
	const blockProps = useBlockProps();
	return (
		<div { ...blockProps }>
			<p>Content</p>
		</div>
	);
}

// save.js
import { useBlockProps } from '@wordpress/block-editor';

function save( { attributes } ) {
	const blockProps = useBlockProps.save();
	return (
		<div { ...blockProps }>
			<p>Content</p>
		</div>
	);
}
```

**BAD: Missing useBlockProps in apiVersion 2**

```javascript
// Block will fail validation
function save( { attributes } ) {
	return (
		<div className="my-custom-class">
			<p>{ attributes.content }</p>
		</div>
	);
}
// Error: Missing block wrapper attributes
```

**GOOD: Using useBlockProps properly**

```javascript
function save( { attributes } ) {
	const blockProps = useBlockProps.save( {
		className: 'my-custom-class',
	} );
	return (
		<div { ...blockProps }>
			<p>{ attributes.content }</p>
		</div>
	);
}
```

### apiVersion 3 (WordPress 6.3+)

**Characteristics:**
- Iframe editor isolation enabled
- `className` NOT automatically injected in save
- MUST explicitly call `useBlockProps.save()` to get wrapper attributes
- Better editor/frontend consistency

```javascript
// apiVersion 3 pattern
// block.json
{
	"apiVersion": 3,
	"name": "my-plugin/modern-block"
}

// edit.js (same as apiVersion 2)
import { useBlockProps } from '@wordpress/block-editor';

function Edit( { attributes } ) {
	const blockProps = useBlockProps();
	return (
		<div { ...blockProps }>
			<p>{ attributes.content }</p>
		</div>
	);
}

// save.js - CRITICAL: Must use useBlockProps.save()
import { useBlockProps } from '@wordpress/block-editor';

function save( { attributes } ) {
	const blockProps = useBlockProps.save();
	return (
		<div { ...blockProps }>
			<p>{ attributes.content }</p>
		</div>
	);
}
```

**BAD: apiVersion 3 without useBlockProps.save()**

```javascript
// apiVersion 3 block
function save( { attributes } ) {
	return (
		<div className="wp-block-my-plugin-modern-block">
			<p>{ attributes.content }</p>
		</div>
	);
}
// Block validation error: className not injected automatically in apiVersion 3
```

**GOOD: apiVersion 3 with useBlockProps.save()**

```javascript
function save( { attributes } ) {
	const blockProps = useBlockProps.save();
	return (
		<div { ...blockProps }>
			<p>{ attributes.content }</p>
		</div>
	);
}
// Generates correct wrapper with all block attributes
```

### Migration from apiVersion 2 to 3

**If your save function uses useBlockProps.save():** Migration is seamless.

**If your save function doesn't use useBlockProps.save():** Add deprecation entry.

```javascript
// Migration example
const deprecated = [
	{
		apiVersion: 2,
		attributes: {
			content: { type: 'string' },
		},
		save( { attributes } ) {
			// Old apiVersion 2 save without useBlockProps.save()
			return (
				<div className="wp-block-my-plugin-block">
					<p>{ attributes.content }</p>
				</div>
			);
		},
	},
];

// Current apiVersion 3 save
function save( { attributes } ) {
	const blockProps = useBlockProps.save();
	return (
		<div { ...blockProps }>
			<p>{ attributes.content }</p>
		</div>
	);
}

export default {
	edit: Edit,
	save,
	deprecated,
};
```

## Block Name and Registration

### Block Name Format

Block names must follow `namespace/block-name` pattern:

**BAD: Invalid block names**

```json
{
	"name": "MyBlock"
}
// Error: No namespace

{
	"name": "my_plugin/my_block"
}
// Error: Underscores not allowed

{
	"name": "MyPlugin/MyBlock"
}
// Error: Must be lowercase
```

**GOOD: Valid block names**

```json
{
	"name": "my-plugin/custom-block"
}
// Correct: namespace/block-name, lowercase, hyphens

{
	"name": "acme-corp/hero-section"
}
// Correct: Multi-word namespace
```

**Naming conventions:**
- **Namespace:** Plugin slug or company identifier (lowercase, hyphens)
- **Block name:** Descriptive, lowercase, hyphens
- **Uniqueness:** Name must be globally unique across all installed plugins

### Registration from block.json Path

WordPress resolves block.json from the directory path:

```php
// PHP registration
register_block_type( __DIR__ . '/build/my-block' );
// Looks for: /build/my-block/block.json
// Resolves assets relative to block.json location
```

**File structure:**

```
/wp-content/plugins/my-plugin/
├── my-plugin.php
└── build/
    └── my-block/
        ├── block.json         # Metadata
        ├── index.js           # Compiled editor JS
        ├── index.asset.php    # Auto-generated dependencies
        └── style-index.css    # Compiled CSS
```

**block.json:**

```json
{
	"name": "my-plugin/my-block",
	"editorScript": "file:./index.js",
	"style": "file:./style-index.css"
}
```

**How WordPress resolves paths:**
1. Reads block.json from `/build/my-block/block.json`
2. Resolves `"file:./index.js"` relative to block.json: `/build/my-block/index.js`
3. Looks for `/build/my-block/index.asset.php` (auto-generated by @wordpress/scripts)
4. Enqueues assets with correct dependencies and version

**BAD: Absolute paths**

```json
{
	"editorScript": "file:/var/www/html/wp-content/plugins/my-plugin/build/index.js"
}
// Error: Paths must be relative to block.json
```

**GOOD: Relative paths with file: prefix**

```json
{
	"editorScript": "file:./index.js",
	"style": "file:./style-index.css"
}
```

## Attributes Schema

Attributes define the data model for blocks. Each attribute has a type, optional source, and optional default.

### Complete Attribute Types

```json
{
	"attributes": {
		"stringAttribute": {
			"type": "string",
			"default": ""
		},
		"numberAttribute": {
			"type": "number",
			"default": 0
		},
		"integerAttribute": {
			"type": "integer",
			"default": 0
		},
		"booleanAttribute": {
			"type": "boolean",
			"default": false
		},
		"arrayAttribute": {
			"type": "array",
			"default": []
		},
		"objectAttribute": {
			"type": "object",
			"default": {}
		},
		"nullAttribute": {
			"type": "null"
		}
	}
}
```

### Attribute Sources

Sources determine where attribute values are extracted from saved HTML.

**Source: attribute (extract from HTML attribute)**

```json
{
	"url": {
		"type": "string",
		"source": "attribute",
		"selector": "img",
		"attribute": "src"
	}
}
```

Extracts from: `<img src="https://example.com/image.jpg" />`
Result: `{ url: "https://example.com/image.jpg" }`

**Source: text (extract text content)**

```json
{
	"heading": {
		"type": "string",
		"source": "text",
		"selector": "h2"
	}
}
```

Extracts from: `<h2>Hello World</h2>`
Result: `{ heading: "Hello World" }`

**Source: html (extract inner HTML)**

```json
{
	"content": {
		"type": "string",
		"source": "html",
		"selector": "p"
	}
}
```

Extracts from: `<p>Hello <strong>World</strong></p>`
Result: `{ content: "Hello <strong>World</strong>" }`

**Source: query (extract array of objects)**

```json
{
	"items": {
		"type": "array",
		"source": "query",
		"selector": "li",
		"query": {
			"text": {
				"type": "string",
				"source": "text"
			},
			"url": {
				"type": "string",
				"source": "attribute",
				"attribute": "data-url"
			}
		}
	}
}
```

Extracts from:
```html
<ul>
	<li data-url="/page1">Page One</li>
	<li data-url="/page2">Page Two</li>
</ul>
```

Result:
```javascript
{
	items: [
		{ text: "Page One", url: "/page1" },
		{ text: "Page Two", url: "/page2" }
	]
}
```

**Source: meta (DEPRECATED)**

```json
{
	"metaValue": {
		"type": "string",
		"source": "meta",
		"meta": "my_meta_key"
	}
}
```

**DO NOT USE** — Use `useEntityProp` hook instead (see editor-patterns.md).

### Selector Syntax

Selectors use CSS selector syntax:

```json
{
	"heading": {
		"source": "text",
		"selector": "h2"
	},
	"imageUrl": {
		"source": "attribute",
		"selector": ".feature-image img",
		"attribute": "src"
	},
	"linkText": {
		"source": "text",
		"selector": "a.primary-link"
	}
}
```

**No selector = root element:**

```json
{
	"content": {
		"type": "string",
		"source": "html"
	}
}
```

Extracts HTML from block wrapper itself.

### Type Mismatch Dangers

**BAD: Type doesn't match source data**

```json
{
	"count": {
		"type": "number",
		"source": "text",
		"selector": ".count"
	}
}
```

Extracts from: `<span class="count">5</span>`
Result: `{ count: undefined }` — text extraction returns string "5", type mismatch

**GOOD: Type matches source data**

```json
{
	"count": {
		"type": "number",
		"source": "attribute",
		"selector": ".count",
		"attribute": "data-count"
	}
}
```

Extracts from: `<span class="count" data-count="5">5</span>`
Result: `{ count: 5 }` — attribute value coerced to number

### Default Values

```json
{
	"alignment": {
		"type": "string",
		"default": "left"
	},
	"showImage": {
		"type": "boolean",
		"default": true
	},
	"columns": {
		"type": "number",
		"default": 3
	}
}
```

**When defaults apply:**
- Block first inserted (attribute not saved yet)
- Attribute not found in saved HTML
- Saved value doesn't match type

**BAD: Complex objects as defaults**

```json
{
	"settings": {
		"type": "object",
		"default": {
			"color": "#000000",
			"fontSize": 16,
			"nested": {
				"value": true
			}
		}
	}
}
```

**Caution:** Complex defaults can cause serialization issues. Keep defaults simple.

## Supports Catalog

The `supports` object enables block-level features and UI controls.

### Color Supports

```json
{
	"supports": {
		"color": {
			"text": true,
			"background": true,
			"gradients": true,
			"link": true,
			"button": false,
			"heading": false,
			"enableContrastChecker": true
		}
	}
}
```

**Available color options:**
- `text` — Text color picker
- `background` — Background color picker
- `gradients` — Gradient background picker
- `link` — Link color (for blocks with links)
- `button` — Button color (for blocks with buttons)
- `heading` — Heading color (for blocks with headings)
- `enableContrastChecker` — Accessibility contrast warnings

**Default:** `false` (no color controls)

**BAD: Missing color supports**

```json
{
	"supports": {}
}
```

Users cannot customize colors — hardcoded in block code.

**GOOD: Appropriate color supports**

```json
{
	"supports": {
		"color": {
			"text": true,
			"background": true,
			"gradients": true
		}
	}
}
```

Users can customize text, background, and gradients. Values applied via `useBlockProps()`.

### Spacing Supports

```json
{
	"supports": {
		"spacing": {
			"margin": true,
			"padding": true,
			"blockGap": true
		}
	}
}
```

**Available spacing options:**
- `margin` — Margin controls (top, right, bottom, left)
- `padding` — Padding controls
- `blockGap` — Gap between inner blocks

**Per-side control:**

```json
{
	"supports": {
		"spacing": {
			"margin": [ "top", "bottom" ],
			"padding": true
		}
	}
}
```

Only top/bottom margin controls shown, full padding controls.

### Typography Supports

```json
{
	"supports": {
		"typography": {
			"fontSize": true,
			"lineHeight": true,
			"fontFamily": true,
			"fontWeight": true,
			"fontStyle": true,
			"textTransform": true,
			"textDecoration": true,
			"letterSpacing": true
		}
	}
}
```

**Available typography options:**
- `fontSize` — Font size picker (preset + custom)
- `lineHeight` — Line height slider
- `fontFamily` — Font family dropdown
- `fontWeight` — Font weight (100-900)
- `fontStyle` — Italic toggle
- `textTransform` — Uppercase/lowercase/capitalize
- `textDecoration` — Underline/strikethrough
- `letterSpacing` — Letter spacing

### Alignment Supports

```json
{
	"supports": {
		"align": true
	}
}
```

Enables all alignments: left, center, right, wide, full.

**Restrict to specific alignments:**

```json
{
	"supports": {
		"align": [ "left", "center", "right" ]
	}
}
```

Only left/center/right alignment (no wide/full).

**Default:** `false` (no alignment controls)

### Other Common Supports

```json
{
	"supports": {
		"anchor": true,
		"html": false,
		"className": true,
		"customClassName": true,
		"reusable": true,
		"lock": true,
		"inserter": true,
		"multiple": true
	}
}
```

**Support flags:**
- `anchor` — HTML anchor field (for link targets)
- `html` — "Edit as HTML" option
- `className` — Additional CSS class field
- `customClassName` — Disable custom class name field
- `reusable` — Can be saved as reusable block
- `lock` — Can be locked (prevent removal/movement)
- `inserter` — Show in block inserter
- `multiple` — Allow multiple instances

### WordPress 6.x+ Supports

```json
{
	"supports": {
		"interactivity": true,
		"layout": {
			"type": "flex",
			"allowSwitching": true
		},
		"dimensions": {
			"minHeight": true
		},
		"position": {
			"sticky": true
		},
		"shadow": true
	}
}
```

**Advanced supports:**
- `interactivity` — Enable Interactivity API (WP 6.5+)
- `layout` — Layout controls (flex, grid)
- `dimensions` — Width, height, min-height
- `position` — Position controls (sticky, fixed)
- `shadow` — Box shadow controls

### Supports Applied via useBlockProps

Block supports automatically add classes and styles to block wrapper:

```javascript
import { useBlockProps } from '@wordpress/block-editor';

function Edit( { attributes } ) {
	const blockProps = useBlockProps();
	// blockProps contains:
	// - className: Block classes including alignment, color classes
	// - style: Inline styles for colors, spacing, typography
	return <div { ...blockProps }>Content</div>;
}
```

**BAD: Enabling supports without using useBlockProps**

```json
{
	"supports": {
		"color": {
			"text": true,
			"background": true
		}
	}
}
```

```javascript
function Edit() {
	return <div className="my-block">Content</div>;
}
// Supports enabled but not applied — users see color picker but colors don't apply
```

**GOOD: Supports with useBlockProps**

```javascript
function Edit() {
	const blockProps = useBlockProps();
	return <div { ...blockProps }>Content</div>;
}
// Colors, spacing, typography automatically applied
```

## Asset Registration

### Asset File Types

**editorScript** — JavaScript for editor only

```json
{
	"editorScript": "file:./index.js"
}
```

Loaded only in block editor. Contains `registerBlockType()`, edit component.

**script** — JavaScript for frontend AND editor

```json
{
	"script": "file:./script.js"
}
```

Loaded on both frontend and in editor. Use for shared utilities.

**viewScript** — JavaScript for frontend only (legacy)

```json
{
	"viewScript": "file:./view.js"
}
```

Loaded only on frontend. For frontend interactions.

**viewScriptModule** — JavaScript ES module for frontend (WP 6.5+)

```json
{
	"viewScriptModule": "file:./view.js"
}
```

Loaded as ES module. **Required for Interactivity API.** Use instead of viewScript for modern blocks.

**editorStyle** — CSS for editor only

```json
{
	"editorStyle": "file:./editor.css"
}
```

**style** — CSS for frontend AND editor

```json
{
	"style": "file:./style-index.css"
}
```

### file: Prefix and Relative Paths

**BAD: Missing file: prefix**

```json
{
	"editorScript": "./index.js",
	"style": "style.css"
}
```

WordPress won't recognize as relative paths.

**GOOD: file: prefix for relative paths**

```json
{
	"editorScript": "file:./index.js",
	"style": "file:./style-index.css"
}
```

Resolves relative to block.json location.

### Auto-Generated .asset.php Files

When using @wordpress/scripts, WordPress generates `.asset.php` files:

**Build output:**

```
build/
├── index.js
├── index.asset.php    # Auto-generated
└── style-index.css
```

**index.asset.php contents:**

```php
<?php return array(
	'dependencies' => array(
		'wp-block-editor',
		'wp-blocks',
		'wp-components',
		'wp-element',
		'wp-i18n'
	),
	'version' => 'abc123def456'
);
```

WordPress automatically reads this file when registering the block.

**Manual registration without @wordpress/scripts:**

```php
register_block_type( __DIR__ . '/build', array(
	'editor_script_handles' => array( 'my-custom-script' ),
) );
```

### Multiple Assets

```json
{
	"editorScript": [
		"file:./index.js",
		"file:./additional-editor.js"
	],
	"style": [
		"file:./base-style.css",
		"file:./theme-style.css"
	]
}
```

Multiple assets loaded in array order.

**BAD: Manual wp_enqueue_script instead of block.json**

```php
function mypl_enqueue_block_assets() {
	wp_enqueue_script(
		'mypl-block-script',
		plugins_url( 'build/index.js', __FILE__ ),
		array( 'wp-blocks', 'wp-element', 'wp-editor' ),
		'1.0.0'
	);
}
add_action( 'enqueue_block_editor_assets', 'mypl_enqueue_block_assets' );
```

Loses automatic dependency management, version hashing, .asset.php integration.

**GOOD: Asset registration via block.json**

```json
{
	"editorScript": "file:./index.js"
}
```

WordPress handles enqueuing, dependencies, versioning automatically.

## Block Categories and Icons

### Standard Block Categories

```json
{
	"category": "text"
}
```

**Built-in categories:**
- `text` — Text blocks (paragraph, heading, list)
- `media` — Media blocks (image, gallery, video)
- `design` — Design blocks (buttons, spacer, separator)
- `widgets` — Widget blocks (archives, calendar, search)
- `theme` — Theme blocks (site logo, navigation, query loop)
- `embed` — Embed blocks (YouTube, Twitter, etc.)

### Custom Categories

**PHP registration:**

```php
add_filter( 'block_categories_all', 'mypl_register_block_category', 10, 2 );

function mypl_register_block_category( $categories, $context ) {
	return array_merge(
		$categories,
		array(
			array(
				'slug'  => 'my-plugin',
				'title' => __( 'My Plugin Blocks', 'my-plugin' ),
				'icon'  => 'smiley',
			),
		)
	);
}
```

**Use in block.json:**

```json
{
	"category": "my-plugin"
}
```

### Block Icons

**Dashicon slug:**

```json
{
	"icon": "smiley"
}
```

See: https://developer.wordpress.org/resource/dashicons/

**Custom SVG:**

```json
{
	"icon": "<svg viewBox='0 0 24 24' xmlns='http://www.w3.org/2000/svg'><path d='M12 2L2 22h20L12 2z' /></svg>"
}
```

**Icon object with colors:**

```json
{
	"icon": {
		"src": "smiley",
		"background": "#7e70af",
		"foreground": "#fff"
	}
}
```

### Search Keywords

```json
{
	"keywords": [ "custom", "example", "demo" ]
}
```

**Limit:** Maximum 3 keywords
**Purpose:** Improve block discoverability in inserter search

## Render Property

### Dynamic Block with Render File

```json
{
	"render": "file:./render.php"
}
```

Specifies PHP file for server-side rendering (dynamic blocks).

**render.php structure:**

```php
<?php
/**
 * Render callback for my-plugin/my-block
 *
 * @param array    $attributes Block attributes.
 * @param string   $content    Block content (inner blocks).
 * @param WP_Block $block      Block instance.
 */

$wrapper_attributes = get_block_wrapper_attributes();
?>
<div <?php echo $wrapper_attributes; ?>>
	<p><?php echo esc_html( $attributes['content'] ); ?></p>
</div>
```

**Available variables in render.php:**
- `$attributes` — Associative array of block attributes
- `$content` — Saved HTML from InnerBlocks
- `$block` — WP_Block instance (context, parsed_block)

### Render File vs render_callback

**Method 1: render file in block.json (preferred)**

```json
{
	"render": "file:./render.php"
}
```

**Method 2: render_callback in PHP**

```php
register_block_type( __DIR__ . '/build/my-block', array(
	'render_callback' => 'mypl_render_callback',
) );
```

**Precedence:** `render` property takes precedence over `render_callback`.

**BAD: Both render and render_callback**

```json
{
	"render": "file:./render.php"
}
```

```php
register_block_type( __DIR__ . '/build', array(
	'render_callback' => 'mypl_render', // Ignored!
) );
```

render.php executes, render_callback ignored.

**GOOD: Use render file only**

```json
{
	"render": "file:./render.php"
}
```

```php
register_block_type( __DIR__ . '/build' );
```

Clean, declarative registration.

## Block Variations in block.json

### Variation Structure

```json
{
	"variations": [
		{
			"name": "blue",
			"title": "Blue Style",
			"description": "A blue variant",
			"icon": "admin-appearance",
			"attributes": {
				"backgroundColor": "blue",
				"textColor": "white"
			},
			"isDefault": false,
			"scope": [ "inserter", "transform" ]
		}
	]
}
```

**Variation properties:**
- `name` — Unique variation identifier
- `title` — Human-readable title
- `description` — Description shown in UI
- `icon` — Dashicon slug or SVG
- `attributes` — Preset attribute values
- `isDefault` — Set as default variation (first insert)
- `scope` — Where variation appears: `inserter`, `transform`, `block`

### Scope Options

**inserter** — Variation appears in block inserter

**transform** — Variation appears in block transforms

**block** — Variation appears in block inspector (style variations)

```json
{
	"scope": [ "inserter" ]
}
```

Only in inserter, not transforms.

### PHP-Registered Variations

Variations can also be registered in PHP:

```php
register_block_variation(
	'my-plugin/my-block',
	array(
		'name'       => 'red',
		'title'      => __( 'Red Style', 'my-plugin' ),
		'attributes' => array(
			'backgroundColor' => 'red',
		),
	)
);
```

**block.json variations vs PHP variations:**
- block.json: Static, bundled with block
- PHP: Dynamic, can be conditional or filtered

## Block Hooks in block.json

WordPress 6.4+ feature for auto-inserting blocks.

```json
{
	"blockHooks": {
		"core/post-content": "after",
		"core/group": "lastChild"
	}
}
```

**Position values:**
- `before` — Insert before target block
- `after` — Insert after target block
- `firstChild` — Insert as first inner block
- `lastChild` — Insert as last inner block

**CRITICAL: Dynamic blocks only**

```json
{
	"blockHooks": {
		"core/post-content": "after"
	},
	"render": "file:./render.php"
}
```

**BAD: Static block with blockHooks**

```json
{
	"blockHooks": {
		"core/post-content": "after"
	}
}
```

```javascript
function save() {
	return <div>Static HTML</div>;
}
```

Block won't auto-insert — blockHooks require dynamic rendering.

**GOOD: Dynamic block with blockHooks**

```json
{
	"blockHooks": {
		"core/post-content": "after"
	},
	"render": "file:./render.php"
}
```

```javascript
function save() {
	return null;
}
```

Block auto-inserts after post content.

## Common block.json Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| **apiVersion as string** | `"apiVersion": "3"` | `"apiVersion": 3` (number) |
| **Missing textdomain** | Strings not translatable | Add `"textdomain": "plugin-slug"` |
| **Relative path without file:** | `"editorScript": "./index.js"` | `"editorScript": "file:./index.js"` |
| **Attribute source/selector mismatch** | Type doesn't match extracted data | Match type to source data type |
| **Missing category** | Block registration fails | Add valid category slug |
| **Supports nested incorrectly** | `"color": "true"` | `"color": { "text": true }` (object) |
| **Using deprecated meta source** | `"source": "meta"` | Use `useEntityProp` hook instead |
| **blockHooks on static blocks** | Auto-insert doesn't work | Convert to dynamic block |
| **Multiple $schema URLs** | Validation confusion | Use single schema URL |
| **Invalid block name** | No namespace or uppercase | `"namespace/block-name"` (lowercase) |

## Cross-References

- **React/JSX editor patterns:** See editor-patterns.md
- **Dynamic rendering with render.php:** See dynamic-blocks-guide.md
- **Interactivity API with viewScriptModule:** See interactivity-api-guide.md
- **Block deprecation for apiVersion migrations:** See editor-patterns.md
