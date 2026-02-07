# Phase 3: Gutenberg Blocks - Research

**Researched:** 2026-02-06
**Domain:** WordPress Block Editor (Gutenberg) - Block Development
**Confidence:** HIGH

## Summary

WordPress block development (Gutenberg) requires both PHP and JavaScript/React expertise. Modern blocks (WordPress 6.3+) use `block.json` as the single source of truth with apiVersion 3, which enables iframe isolation in the editor. The WordPress Block Editor follows a dual-rendering architecture: React components for the editor experience (edit function) and either static HTML (save function) or server-side PHP (render_callback) for the frontend.

WordPress 6.5+ introduced the Interactivity API, which provides a declarative system for adding frontend interactions using `data-wp-*` directives in HTML markup and server-side state management via `wp_interactivity_state()`. This is fundamentally different from the React-based editor code and is **only for frontend views**, not the editor.

Key review concerns include block validation errors (when save function markup changes without deprecation handlers), InnerBlocks misuse in dynamic blocks (must save `<InnerBlocks.Content />` even if using render_callback), and the wp_kses_post() anti-pattern on InnerBlocks content (breaks embed blocks).

**Primary recommendation:** Structure the skill around the block.json schema as the foundation, then cover the three rendering approaches (static, dynamic, Interactivity API) with dedicated reference docs for each paradigm.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Detection scope & depth:**
- Deep coverage (dedicated sections in SKILL.md): block.json schema validation (every field), register_block_type() patterns, dynamic blocks with render_callback/render file, static blocks with save() JSX, block attributes (type/source/selector/default), useBlockProps hook, InnerBlocks with allowedBlocks/template, InspectorControls/BlockControls, block deprecations, Interactivity API directives
- Surface-level coverage (scan patterns + brief guidance): block transforms, variations, styles, SlotFill, format types, block patterns (PHP templates), block templates and template lock, @wordpress/scripts build toolchain, block hooks
- Context-aware detection: Distinguish single block plugin, multi-block plugin, block library, theme blocks

**JavaScript/React coverage:**
- First skill covering JavaScript/React (prior skills were PHP-only)
- Block editor patterns only, NOT general React patterns
- Cover WordPress-specific APIs: useBlockProps, RichText, InspectorControls, BlockControls, InnerBlocks, useSelect, useDispatch, useEntityProp
- BAD/GOOD code pairs work for JSX (ES6+ modern JS conventions)
- WordPress JS coding standards: tab indentation, JSDoc comments, camelCase for variables/functions, PascalCase for components
- Grep patterns must cover both PHP and JS/JSX files
- No TypeScript depth (mention supported, but don't make primary)
- Import patterns: detect @wordpress/* package imports (correct) vs window.wp.* global access (legacy/incorrect)

**Interactivity API treatment:**
- Dedicated reference doc (interactivity-api-guide.md)
- WP 6.5+ version marker throughout
- Key distinction: Interactivity API is for FRONTEND (view), not editor. Editor still uses React.
- Cover directives: data-wp-bind, data-wp-on, data-wp-class, data-wp-style, data-wp-text, data-wp-context, data-wp-watch, data-wp-init, data-wp-each, data-wp-interactive
- Server-side state: wp_interactivity_state() and wp_interactivity_config()
- Cross-reference: When Interactivity API involves security (user input in state), reference wp-security-review

**Reference doc structure:**
- Follow established cookbook format: quick reference table + detailed patterns + edge cases + WP nuances
- Four reference docs organized by concern (not by language):
  - block-json-guide.md: Complete block.json schema reference
  - editor-patterns.md: React/JSX patterns for block editor
  - dynamic-blocks-guide.md: Server-side rendering patterns
  - interactivity-api-guide.md: WP 6.5+ Interactivity API
- BAD/GOOD pairs throughout (PHP follows WordPress PHP Coding Standards, JS/JSX follows WordPress JS coding standards)

**Security and plugin skill crossover:**
- Do NOT duplicate security or plugin patterns
- Block-specific security: Escape output in render_callback, sanitize block attributes in PHP, validate attribute types. Brief reminder + cross-reference.
- Block-specific plugin patterns: register_block_type() is plugin registration function. Brief mention + cross-reference wp-plugin-development.
- Slash command integration: If /wp-block-review finds security concerns, suggest /wp-sec-review. If plugin architecture issues found, suggest /wp-plugin-review.

**Finding output style:**
- Mirror security/performance/plugin skill output format exactly
- Group findings by FILE (PHP and JS files intermixed by actual file path)
- Each finding: line number, severity, issue description, explanation, BAD/GOOD code pair
- Summary at end with total counts by severity
- Severity mapping:
  - CRITICAL: Block won't render OR crashes editor (missing apiVersion, invalid attribute type, save function mismatch causing block validation error, missing useBlockProps in save)
  - WARNING: Block works but has quality/compatibility issues (deprecated API usage, missing supports flags, no deprecation handler for changed save, hardcoded strings without i18n)
  - INFO: Best practice improvements (could use render PHP file instead of render_callback, missing viewScript for frontend interaction, not using block.json for asset registration)

**Quick scan vs full review boundary:**
- /wp-block (quick scan): Grep-based pattern detection across PHP and JS/JSX files
- /wp-block-review (full review): Complete skill workflow, produces structured report
- Same boundary pattern as security/performance/plugin skills

### Claude's Discretion

- Exact grep patterns for quick detection (tuned during implementation - JS grep patterns may need different regex than PHP)
- Depth of @wordpress/scripts toolchain coverage (enough to identify misconfiguration, not a build system tutorial)
- Whether to include block theme integration patterns (blocks in theme context) or defer to Phase 4
- How to handle the JSX compilation question (source vs build files - which to review)
- Block pattern (template) coverage depth vs block code patterns

### Deferred Ideas (OUT OF SCOPE)

None - all Gutenberg block development concerns fit within phase scope
</user_constraints>

## Standard Stack

The established libraries/tools for WordPress block development:

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| @wordpress/block-editor | Latest | Block editor components (useBlockProps, InspectorControls, RichText) | Official WordPress package for editor UI components |
| @wordpress/blocks | Latest | Block registration and validation | Core package for registerBlockType() and block lifecycle |
| @wordpress/data | Latest | State management (useSelect, useDispatch) | WordPress's React-Redux layer for editor data |
| @wordpress/components | Latest | UI primitives (PanelBody, ToggleControl, TextControl) | Official UI component library for block settings |
| @wordpress/scripts | Latest | Build toolchain (Webpack, Babel, ESLint config) | Zero-config build setup from WordPress core team |
| @wordpress/i18n | Latest | Internationalization (__(), _e()) | Official i18n solution for block strings |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| @wordpress/element | Latest | React abstraction layer | Implicit dependency (uses createElement) |
| @wordpress/api-fetch | Latest | REST API communication | When blocks fetch data from WordPress REST API |
| @wordpress/hooks | Latest | Filters and actions in JS | When extending core block behavior |
| @wordpress/compose | Latest | Higher-order components and hooks | For advanced patterns (useInstanceId, compose) |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| @wordpress/scripts | Custom Webpack config | Lose automatic updates and WordPress compatibility guarantees |
| @wordpress/components | Third-party UI library | Inconsistent with WordPress design system, larger bundle size |
| useSelect/useDispatch | Direct Redux | Breaks compatibility with WordPress data stores |

**Installation:**
```bash
npm install @wordpress/scripts --save-dev
npm install @wordpress/block-editor @wordpress/blocks @wordpress/data @wordpress/components @wordpress/i18n --save
```

**Note:** Using `@wordpress/create-block` scaffolds a complete block with all dependencies pre-configured.

## Architecture Patterns

### Recommended Project Structure
```
my-block-plugin/
├── plugin.php                    # Main plugin file, register_block_type() on init hook
├── block.json                    # Block metadata (single block plugins)
├── package.json                  # npm scripts, @wordpress dependencies
├── src/
│   ├── index.js                  # registerBlockType() for client-side
│   ├── edit.js                   # Edit component (React)
│   ├── save.js                   # Save function (static blocks) or null (dynamic)
│   ├── style.scss                # Frontend + editor styles
│   └── editor.scss               # Editor-only styles
├── build/                        # Compiled assets (wp-scripts output)
│   ├── index.js
│   ├── index.asset.php           # Auto-generated dependency list
│   └── style-index.css
└── render.php                    # Optional: Dynamic block render template
```

**Multi-block structure:**
```
multi-block-plugin/
├── plugin.php
├── package.json
├── src/
│   ├── block-one/
│   │   ├── block.json            # Metadata per block
│   │   ├── index.js
│   │   ├── edit.js
│   │   └── save.js
│   └── block-two/
│       ├── block.json
│       ├── index.js
│       └── render.php             # Dynamic block
└── build/
    ├── block-one/
    └── block-two/
```

### Pattern 1: Static Block (Fixed HTML Output)
**What:** Block saves HTML to database via save() function. Frontend displays exact saved markup.
**When to use:** Fixed content (buttons, spacers, testimonials), no need for updates after saving.
**Example:**
```javascript
// Source: https://developer.wordpress.org/block-editor/getting-started/fundamentals/block-wrapper/
import { useBlockProps, RichText } from '@wordpress/block-editor';

// Edit function
export default function Edit( { attributes, setAttributes } ) {
	const blockProps = useBlockProps();
	return (
		<div { ...blockProps }>
			<RichText
				tagName="p"
				value={ attributes.content }
				onChange={ ( content ) => setAttributes( { content } ) }
			/>
		</div>
	);
}

// Save function - MUST use useBlockProps.save()
function save( { attributes } ) {
	const blockProps = useBlockProps.save();
	return (
		<div { ...blockProps }>
			<RichText.Content tagName="p" value={ attributes.content } />
		</div>
	);
}
```

### Pattern 2: Dynamic Block (Server-Side Rendering)
**What:** Block saves minimal/no HTML, server-side PHP generates output on each render.
**When to use:** Content that changes (Latest Posts), markup updates should apply site-wide without re-saving posts.
**Example:**
```php
// Source: https://developer.wordpress.org/block-editor/getting-started/fundamentals/static-dynamic-rendering/
// plugin.php
function my_plugin_register_block() {
	register_block_type( __DIR__ . '/build/my-block', array(
		'render_callback' => 'my_plugin_render_callback',
	) );
}
add_action( 'init', 'my_plugin_register_block' );

function my_plugin_render_callback( $attributes, $content, $block ) {
	$wrapper_attributes = get_block_wrapper_attributes();
	return sprintf(
		'<div %1$s><p>%2$s</p></div>',
		$wrapper_attributes,
		esc_html( $attributes['content'] )
	);
}
```

```javascript
// save.js - Dynamic blocks return null
export default function save() {
	return null;
}
```

### Pattern 3: Interactivity API (Frontend JavaScript)
**What:** Server-side HTML with `data-wp-*` directives for frontend interactivity. State managed in PHP.
**When to use:** Frontend interactions (toggles, tabs, accordions) without custom JavaScript bundles (WP 6.5+).
**Example:**
```php
// Source: https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/
// render.php
<?php
wp_interactivity_state( 'myPlugin', array(
	'isOpen' => false,
) );

$wrapper_attributes = get_block_wrapper_attributes();
?>
<div
	<?php echo $wrapper_attributes; ?>
	data-wp-interactive="myPlugin"
	data-wp-context='{ "id": "<?php echo esc_attr( uniqid() ); ?>" }'
>
	<button data-wp-on--click="actions.toggle">
		Toggle
	</button>
	<div data-wp-bind--hidden="!state.isOpen">
		<p>Content here</p>
	</div>
</div>
```

```javascript
// view.js - Interactivity API store
import { store } from '@wordpress/interactivity';

store( 'myPlugin', {
	actions: {
		toggle: ( { state } ) => {
			state.isOpen = !state.isOpen;
		},
	},
} );
```

### Pattern 4: InnerBlocks (Nested Blocks)
**What:** Allows blocks to contain other blocks. Essential for container/layout blocks.
**When to use:** Columns, groups, cards with flexible content.
**Example:**
```javascript
// Source: https://developer.wordpress.org/block-editor/how-to-guides/block-tutorial/nested-blocks-inner-blocks/
import { useBlockProps, InnerBlocks } from '@wordpress/block-editor';

// Edit function
export default function Edit() {
	const blockProps = useBlockProps();
	const ALLOWED_BLOCKS = [ 'core/paragraph', 'core/image' ];
	const TEMPLATE = [
		[ 'core/heading', { placeholder: 'Card Title' } ],
		[ 'core/paragraph', { placeholder: 'Card content...' } ],
	];

	return (
		<div { ...blockProps }>
			<InnerBlocks
				allowedBlocks={ ALLOWED_BLOCKS }
				template={ TEMPLATE }
			/>
		</div>
	);
}

// Save function - CRITICAL: Must use InnerBlocks.Content
function save() {
	const blockProps = useBlockProps.save();
	return (
		<div { ...blockProps }>
			<InnerBlocks.Content />
		</div>
	);
}
```

**CRITICAL for dynamic blocks with InnerBlocks:**
```javascript
// Even if using render_callback, save MUST include InnerBlocks.Content
function save() {
	return <InnerBlocks.Content />; // Save nested blocks to database
}
```

### Pattern 5: Block Deprecation (Save Function Migration)
**What:** Handles save function changes while maintaining backward compatibility.
**When to use:** Any time save() markup or attributes schema changes.
**Example:**
```javascript
// Source: https://developer.wordpress.org/block-editor/reference-guides/block-api/block-deprecation/
const deprecated = [
	{
		// Version 1: Old save function
		attributes: {
			content: { type: 'string' },
		},
		save( { attributes } ) {
			// Old markup without useBlockProps
			return <p>{ attributes.content }</p>;
		},
		migrate( attributes ) {
			// Optional: Transform old attributes to new schema
			return {
				...attributes,
				newField: 'default',
			};
		},
	},
];

export default {
	// Current block definition
	edit: Edit,
	save,
	deprecated, // Array checked when validation fails
};
```

### Pattern 6: Block Controls (Settings Sidebar & Toolbar)
**What:** InspectorControls for settings sidebar, BlockControls for toolbar.
**When to use:** Block-level configuration options.
**Example:**
```javascript
// Source: https://developer.wordpress.org/block-editor/getting-started/fundamentals/block-in-the-editor/
import { InspectorControls, BlockControls, AlignmentToolbar } from '@wordpress/block-editor';
import { PanelBody, ToggleControl } from '@wordpress/components';

export default function Edit( { attributes, setAttributes } ) {
	return (
		<>
			<BlockControls>
				<AlignmentToolbar
					value={ attributes.align }
					onChange={ ( align ) => setAttributes( { align } ) }
				/>
			</BlockControls>

			<InspectorControls>
				<PanelBody title="Settings">
					<ToggleControl
						label="Show featured image"
						checked={ attributes.showImage }
						onChange={ ( showImage ) => setAttributes( { showImage } ) }
					/>
				</PanelBody>
			</InspectorControls>

			<div { ...useBlockProps() }>
				{/* Block content */}
			</div>
		</>
	);
}
```

### Anti-Patterns to Avoid

- **Using wp_kses_post() on InnerBlocks content:** Breaks embed blocks and oEmbed processing. Inner blocks are already sanitized by WordPress.
  ```php
  // BAD
  echo wp_kses_post( $content ); // Breaks core/embed and filters

  // GOOD
  echo $content; // Inner blocks already sanitized
  ```

- **Missing useBlockProps in save():** Causes apiVersion 3 blocks to fail validation.
  ```javascript
  // BAD
  function save( { attributes } ) {
  	return <div className="my-block">{ attributes.content }</div>; // Missing block wrapper
  }

  // GOOD
  function save( { attributes } ) {
  	return <div { ...useBlockProps.save() }>{ attributes.content }</div>;
  }
  ```

- **window.wp.* global access:** Legacy pattern, breaks modern build toolchain.
  ```javascript
  // BAD
  const { registerBlockType } = window.wp.blocks;

  // GOOD
  import { registerBlockType } from '@wordpress/blocks';
  ```

- **Dynamic blocks returning null without InnerBlocks.Content:** When block has nested blocks, they won't save.
  ```javascript
  // BAD (if block uses InnerBlocks)
  function save() {
  	return null; // Nested blocks lost
  }

  // GOOD
  function save() {
  	return <InnerBlocks.Content />; // Saves nested blocks
  }
  ```

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Build configuration | Custom Webpack setup | @wordpress/scripts | Auto-updates with WordPress core, handles JSX, SCSS, asset dependencies |
| Color picker UI | Custom color input | ColorPalette from @wordpress/components | Integrates with theme.json, provides palette fallbacks |
| Rich text editing | contentEditable wrapper | RichText from @wordpress/block-editor | Handles formatting, undo/redo, paste handling, block splitting |
| Block validation | Manual HTML parsing | Block deprecation API | Handles multiple migration paths, non-sequential deprecations |
| Frontend state management | Custom event listeners | Interactivity API (WP 6.5+) | Server-side state hydration, directives, standardized patterns |
| Block wrapper classes | Manual class concatenation | useBlockProps() / get_block_wrapper_attributes() | Handles block supports, custom classes, unique IDs, accessibility |
| Media upload | Custom file upload handlers | MediaUpload / MediaUploadCheck from @wordpress/block-editor | Permission checks, media library integration, size limits |
| Block-level settings UI | Custom modal/panel | InspectorControls with @wordpress/components | Consistent UX, responsive sidebar, tabs integration (WP 6.2+) |

**Key insight:** WordPress provides comprehensive UI primitives and state management. Custom solutions lose theme.json integration, accessibility features, and future WordPress updates.

## Common Pitfalls

### Pitfall 1: Block Validation Errors on Save Function Changes
**What goes wrong:** Changing save() markup without deprecation causes "Block validation error" in editor.
**Why it happens:** WordPress compares saved database HTML with current save() output. Mismatch = invalid block.
**How to avoid:** Add deprecation entry BEFORE changing save function. List newest deprecations first.
**Warning signs:** Users see "This block contains unexpected or invalid content" with "Attempt Block Recovery" button.
```javascript
// When changing save markup
const deprecated = [
	{
		save( attributes ) {
			return <div>{ attributes.oldContent }</div>; // Old version
		},
	},
];

// Current save with new markup
function save( attributes ) {
	const blockProps = useBlockProps.save();
	return <div { ...blockProps }>{ attributes.oldContent }</div>; // Added useBlockProps
}
```

### Pitfall 2: Missing apiVersion in Deprecation Entries
**What goes wrong:** Deprecation logic not triggered even though block is invalid.
**Why it happens:** `getSaveElement` applies `blocks.getSaveContent.extraProps` filter only when apiVersion < 2. Without explicit apiVersion in deprecation, class names mismatch.
**How to avoid:** Define apiVersion explicitly in each deprecation entry.
**Warning signs:** Deprecation's save() should match but validation still fails.
```javascript
const deprecated = [
	{
		apiVersion: 2, // CRITICAL: Must match original block's apiVersion
		attributes: { /* ... */ },
		save( { attributes } ) {
			// Old save logic
		},
	},
];
```

### Pitfall 3: useInnerBlocksProps Hook Order
**What goes wrong:** Block wrapper attributes not properly merged when using InnerBlocks.
**Why it happens:** useInnerBlocksProps expects useBlockProps result as first parameter.
**How to avoid:** Call useBlockProps first, pass result to useInnerBlocksProps.
**Warning signs:** InnerBlocks render but block classes or styles missing.
```javascript
// BAD
const innerBlocksProps = useInnerBlocksProps();
const blockProps = useBlockProps( innerBlocksProps ); // Wrong order

// GOOD
const blockProps = useBlockProps();
const innerBlocksProps = useInnerBlocksProps( blockProps ); // Merges properly
```

### Pitfall 4: Meta Attribute Sources (Deprecated)
**What goes wrong:** Using `source: 'meta'` in attributes definition.
**Why it happens:** Legacy pattern from early Gutenberg. Now deprecated.
**How to avoid:** Use EntityProvider with useEntityProp hook instead.
**Warning signs:** Block attributes stored in post meta but not displaying correctly.
```javascript
// BAD
attributes: {
	metaValue: {
		type: 'string',
		source: 'meta',
		meta: 'my_meta_key', // Deprecated
	},
}

// GOOD
import { useEntityProp } from '@wordpress/core-data';

function Edit() {
	const [ meta, setMeta ] = useEntityProp( 'postType', 'post', 'meta' );
	const value = meta.my_meta_key;
	// ...
}
```

### Pitfall 5: Attribute Type Mismatches Causing Silent Failures
**What goes wrong:** Block attributes don't save or display default values unexpectedly.
**Why it happens:** Attribute type doesn't match source data type. For example, `type: 'number'` but source extracts string.
**How to avoid:** Ensure attribute type matches extracted data type. Use attribute validation.
**Warning signs:** Attributes are `undefined` or default value in edit function despite being set.
```javascript
// BAD
attributes: {
	count: {
		type: 'number',
		source: 'text', // Extracts string "5", type mismatch
		selector: '.count',
	},
}

// GOOD
attributes: {
	count: {
		type: 'number',
		source: 'attribute',
		selector: '.count',
		attribute: 'data-count', // Extract from data attribute as number
		default: 0,
	},
}
```

### Pitfall 6: Block Hooks Only Work with Dynamic Blocks
**What goes wrong:** Trying to auto-insert static blocks using blockHooks causes validation errors.
**Why it happens:** WordPress 6.4-6.5 limitation - blockHooks only support dynamic blocks (null save function).
**How to avoid:** Convert to dynamic block if using blockHooks. Use render.php or render_callback.
**Warning signs:** Block doesn't auto-insert, or inserts but shows validation error.
```javascript
// BAD - Static block with blockHooks
// block.json
{
	"blockHooks": {
		"core/post-content": "after"
	}
}
// save.js
function save() {
	return <div>Content</div>; // Static save = blockHooks fail
}

// GOOD - Dynamic block with blockHooks
// block.json
{
	"blockHooks": {
		"core/post-content": "after"
	},
	"render": "file:./render.php"
}
// save.js
function save() {
	return null; // Dynamic block = blockHooks work
}
```

### Pitfall 7: Block Patterns (Templates) vs Block Variations Confusion
**What goes wrong:** Creating block variation when block pattern is needed, or vice versa.
**Why it happens:** Similar names, different purposes. Variations = functionality presets. Patterns = layout presets.
**How to avoid:** Variation for single block with preset attributes. Pattern for multi-block layouts.
**Warning signs:** Users can't customize layout (variation too rigid) or preset doesn't appear in variation picker.
```javascript
// Use VARIATION when: Preset configuration of single block
{
	name: 'three-columns',
	title: 'Three Columns',
	attributes: {
		columns: 3,
	},
	innerBlocks: [
		[ 'core/column' ],
		[ 'core/column' ],
		[ 'core/column' ],
	],
}

// Use PATTERN when: Multi-block layout template
register_block_pattern( 'my-plugin/hero-section', array(
	'title'       => 'Hero Section',
	'content'     => '<!-- wp:cover -->...<!-- /wp:cover --><!-- wp:group -->...<!-- /wp:group -->',
	'categories'  => array( 'featured' ),
) );
```

### Pitfall 8: Hardcoded Strings Without i18n
**What goes wrong:** Block UI text not translatable, fails WordPress.org plugin review.
**Why it happens:** Forgetting to wrap user-facing strings in __() or _e().
**How to avoid:** Wrap all user-facing strings. Use text domain matching plugin slug.
**Warning signs:** Plugin Check (PCP) flags missing i18n, translators can't translate block.
```javascript
// BAD
title: 'My Block',
description: 'A custom block',

// GOOD
import { __ } from '@wordpress/i18n';

title: __( 'My Block', 'my-plugin' ),
description: __( 'A custom block', 'my-plugin' ),
```

### Pitfall 9: useSelect Performance Anti-Pattern
**What goes wrong:** Editor becomes slow when many blocks mount.
**Why it happens:** Each useSelect creates separate store subscription. Thousands of subscriptions = performance degradation.
**How to avoid:** Minimize useSelect calls. Read multiple values in single useSelect when possible.
**Warning signs:** Editor lag with many instances of same block type.
```javascript
// BAD - Multiple subscriptions
const postTitle = useSelect( ( select ) => select( 'core/editor' ).getEditedPostAttribute( 'title' ) );
const postMeta = useSelect( ( select ) => select( 'core/editor' ).getEditedPostAttribute( 'meta' ) );

// GOOD - Single subscription
const { postTitle, postMeta } = useSelect( ( select ) => {
	const { getEditedPostAttribute } = select( 'core/editor' );
	return {
		postTitle: getEditedPostAttribute( 'title' ),
		postMeta: getEditedPostAttribute( 'meta' ),
	};
}, [] );
```

### Pitfall 10: Missing ABSPATH Check in PHP Files
**What goes wrong:** Direct file access possible, security vulnerability.
**Why it happens:** Porting block registration patterns without security header.
**How to avoid:** Add `defined( 'ABSPATH' ) || exit;` to top of all PHP files.
**Warning signs:** WordPress.org Plugin Check flags file, wp-security-review detects issue.
```php
<?php
// BAD - No ABSPATH check
function my_plugin_register_block() {
	// ...
}

// GOOD
<?php
defined( 'ABSPATH' ) || exit;

function my_plugin_register_block() {
	// ...
}
```

## Code Examples

Verified patterns from official sources:

### Block Registration (PHP + JavaScript)
```php
// Source: https://developer.wordpress.org/block-editor/getting-started/fundamentals/registration-of-a-block/
// plugin.php
<?php
defined( 'ABSPATH' ) || exit;

function my_plugin_register_blocks() {
	register_block_type( __DIR__ . '/build/my-block' ); // Points to build/my-block/block.json
}
add_action( 'init', 'my_plugin_register_blocks' );
```

```javascript
// src/my-block/index.js
import { registerBlockType } from '@wordpress/blocks';
import Edit from './edit';
import save from './save';
import metadata from './block.json';

registerBlockType( metadata.name, {
	edit: Edit,
	save,
} );
```

### Complete block.json Schema
```json
{
	"$schema": "https://schemas.wp.org/trunk/block.json",
	"apiVersion": 3,
	"name": "my-plugin/my-block",
	"title": "My Block",
	"category": "widgets",
	"icon": "smiley",
	"description": "A custom block example",
	"keywords": [ "custom", "example" ],
	"version": "1.0.0",
	"textdomain": "my-plugin",
	"attributes": {
		"content": {
			"type": "string",
			"source": "html",
			"selector": "p",
			"default": ""
		}
	},
	"supports": {
		"html": false,
		"color": {
			"background": true,
			"text": true
		},
		"spacing": {
			"margin": true,
			"padding": true
		}
	},
	"editorScript": "file:./index.js",
	"editorStyle": "file:./index.css",
	"style": "file:./style-index.css"
}
```

### RichText Component Pattern
```javascript
// Source: https://developer.wordpress.org/block-editor/reference-guides/richtext/
import { useBlockProps, RichText } from '@wordpress/block-editor';

export default function Edit( { attributes, setAttributes } ) {
	const blockProps = useBlockProps();

	return (
		<RichText
			{ ...blockProps }
			tagName="h2"
			value={ attributes.heading }
			onChange={ ( heading ) => setAttributes( { heading } ) }
			placeholder="Enter heading..."
			allowedFormats={ [ 'core/bold', 'core/italic' ] }
		/>
	);
}

function save( { attributes } ) {
	return (
		<RichText.Content
			{ ...useBlockProps.save() }
			tagName="h2"
			value={ attributes.heading }
		/>
	);
}
```

### Block Context (Parent/Child Data Flow)
```javascript
// Source: https://developer.wordpress.org/block-editor/reference-guides/block-api/block-context/
// Parent block provides context
// block.json
{
	"providesContext": {
		"myPlugin/userId": "userId"
	},
	"attributes": {
		"userId": {
			"type": "number"
		}
	}
}

// Child block uses context
// block.json
{
	"usesContext": [ "myPlugin/userId" ]
}

// child edit.js
export default function Edit( { context } ) {
	const userId = context['myPlugin/userId'];
	// Use parent's userId value
}
```

### Interactivity API with State Management
```php
// Source: https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/
// render.php
<?php
wp_interactivity_state( 'myPlugin', array(
	'isOpen' => false,
	'count'  => 0,
) );

$wrapper_attributes = get_block_wrapper_attributes( array(
	'class' => 'my-interactive-block',
) );
?>
<div
	<?php echo $wrapper_attributes; ?>
	data-wp-interactive="myPlugin"
>
	<button data-wp-on--click="actions.increment">
		Count: <span data-wp-text="state.count"></span>
	</button>

	<div data-wp-bind--hidden="!state.isOpen">
		Hidden content
	</div>
</div>
```

```javascript
// view.js
import { store, getContext } from '@wordpress/interactivity';

store( 'myPlugin', {
	state: {
		isOpen: false,
		count: 0,
	},
	actions: {
		increment: ( { state } ) => {
			state.count += 1;
		},
	},
} );
```

### Block Supports (Color, Spacing, Typography)
```json
// Source: https://developer.wordpress.org/block-editor/reference-guides/block-api/block-supports/
{
	"supports": {
		"color": {
			"text": true,
			"background": true,
			"gradients": true,
			"link": true
		},
		"spacing": {
			"margin": true,
			"padding": true,
			"blockGap": true
		},
		"typography": {
			"fontSize": true,
			"lineHeight": true,
			"fontFamily": true,
			"fontWeight": true
		},
		"anchor": true,
		"align": [ "wide", "full" ],
		"html": false
	}
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| PHP array for register_block_type() | block.json metadata file | WordPress 5.8 | Single source of truth, auto-generates asset dependencies |
| apiVersion 1 or 2 | apiVersion 3 | WordPress 6.3 | Enables iframe editor isolation, removes automatic class injection in save |
| Custom JavaScript for frontend interactions | Interactivity API with data-wp-* directives | WordPress 6.5 | Declarative frontend state, server-side hydration, no custom JS bundles |
| window.wp.* global access | @wordpress/* package imports | WordPress 5.0+ | Modern build toolchain, tree-shaking, ESM support |
| source: 'meta' for post meta | useEntityProp hook with EntityProvider | WordPress 5.9+ | Proper React data flow, avoids deprecated meta source |
| Manual InspectorControls tabs | InspectorControls group prop (settings/styles) | WordPress 6.2 | Auto-sorts into Settings/Appearance tabs in sidebar |
| viewScript field for frontend JS | viewScriptModule for ES modules | WordPress 6.5 | Native ES module support for Interactivity API scripts |

**Deprecated/outdated:**
- **getEditedPostAttribute() without useSelect dependency array**: Causes unnecessary re-renders. Always include empty array `[]` as second parameter to useSelect.
- **blocks.getSaveElement.extraProps filter**: Legacy filter for adding wrapper props. Use block supports or useBlockProps instead.
- **wp.blockEditor vs @wordpress/block-editor**: Global namespace deprecated. Use package imports.
- **Template without templateLock for fixed layouts**: Users can remove required blocks. Use `templateLock: "all"` for fixed structures.

## Open Questions

Things that couldn't be fully resolved:

1. **JSX Source vs Build Review Scope**
   - What we know: @wordpress/scripts compiles src/ to build/. Block registration points to build/ assets.
   - What's unclear: Should skill review src/ files (developer code) or build/ files (what WordPress loads)?
   - Recommendation: Review src/ files for code patterns. Flag if build/ missing or stale (check index.asset.php timestamp vs src/ files). Grep can search both.

2. **Block Theme Integration Depth**
   - What we know: Blocks can be registered in themes. Different loading context than plugins (functions.php vs plugin.php).
   - What's unclear: How much theme-specific block guidance belongs in this skill vs Phase 4 (Theme Development)?
   - Recommendation: Cover block registration in theme context (detect functions.php registration), but defer theme.json integration and template patterns to Phase 4.

3. **TypeScript Support Depth**
   - What we know: WordPress supports TypeScript via @wordpress/scripts. Core uses JSDoc types, not TS.
   - What's unclear: Should skill detect TypeScript-specific patterns or treat .ts/.tsx as equivalent to .js/.jsx?
   - Recommendation: Detect .ts/.tsx files, flag if type definitions missing, but don't enforce TypeScript-specific patterns. Mention TS support in reference docs but keep examples in JavaScript with JSDoc.

4. **Block Pattern (Template) Coverage**
   - What we know: Block patterns are PHP-registered multi-block layouts. Different from block variations (single block presets).
   - What's unclear: How much register_block_pattern() coverage belongs in block skill vs separate pattern skill?
   - Recommendation: Surface-level coverage - detect pattern registration, distinguish from variations, brief mention in SKILL.md, but no dedicated reference doc. Focus on block code patterns, not layout template patterns.

5. **Multi-Block Build Configuration Grep Patterns**
   - What we know: Multi-block plugins have src/block-one/, src/block-two/ structure. Each has block.json.
   - What's unclear: How to grep effectively across multi-block projects where block.json locations vary?
   - Recommendation: Grep for block.json files recursively, detect structure from file locations. Flag if editorScript/script paths in block.json don't resolve relative to block.json location.

## Sources

### Primary (HIGH confidence)
- [WordPress Block Editor Handbook - block.json Metadata Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-metadata/) - Complete schema with all fields
- [WordPress Interactivity API Reference](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/) - Directive catalog and state management
- [WordPress Block Wrapper Documentation](https://developer.wordpress.org/block-editor/getting-started/fundamentals/block-wrapper/) - useBlockProps and get_block_wrapper_attributes usage
- [WordPress Block Deprecation Guide](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-deprecation/) - Migration patterns and save function handling
- [WordPress InnerBlocks Documentation](https://developer.wordpress.org/block-editor/how-to-guides/block-tutorial/nested-blocks-inner-blocks/) - Nested block patterns
- [WordPress Static vs Dynamic Rendering](https://developer.wordpress.org/block-editor/getting-started/fundamentals/static-dynamic-rendering/) - Render approach decision guide
- [WordPress Block Context API](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-context/) - providesContext and usesContext patterns
- [WordPress Block Supports Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-supports/) - Color, spacing, typography configuration
- [WordPress Block Attributes Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-attributes/) - Source, selector, type definitions
- [WordPress @wordpress/data Package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/) - useSelect and useDispatch documentation
- [GitHub Gutenberg Schema - block.json](https://github.com/WordPress/gutenberg/blob/trunk/schemas/json/block.json) - JSON schema definition
- [WordPress Block API Versions Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-api-versions/) - apiVersion 1, 2, 3 differences

### Secondary (MEDIUM confidence)
- [10up Block Editor Best Practices - Data API Guide](https://gutenberg.10up.com/guides/data-api/) - useSelect/useDispatch patterns
- [WordPress Developer News - Block Deprecation Tutorial](https://developer.wordpress.org/news/2023/03/block-deprecation-a-tutorial/) - Migration examples
- [WordPress Developer News - useSelect Hook Best Practices](https://developer.wordpress.org/news/2024/03/how-to-work-effectively-with-the-useselect-hook/) - Performance optimization
- [WordPress Core - Interactivity API Dev Note](https://make.wordpress.org/core/2024/03/04/interactivity-api-dev-note/) - WP 6.5 introduction
- [WordPress Core - Block Hooks Introduction](https://make.wordpress.org/core/2023/10/15/introducing-block-hooks-for-dynamic-blocks/) - Auto-insertion patterns
- [WordPress Developer News - Block Hooks API Exploration](https://developer.wordpress.org/news/2024/03/exploring-the-block-hooks-api-in-wordpress-6-5/) - WP 6.5 updates
- [GitHub Discussion - wp_kses_post with Inner Blocks](https://github.com/WordPress/gutenberg/discussions/37823) - Escaping anti-pattern
- [WordPress @wordpress/scripts Package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/) - Build toolchain documentation
- [WordPress JavaScript Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/javascript/) - Naming conventions, formatting
- [DLX Plugins - block.json Version 3 Guide](https://dlxplugins.com/tutorials/block-json-version-3-a-guide-for-block-developers) - apiVersion 3 migration

### Tertiary (LOW confidence - flagged for validation)
- [Varun Dubey - Gutenberg Blocks in 2026](https://vapvarun.com/gutenberg-blocks-2026-wordpress-block-editor-ai-era/) - WebSearch only, future predictions
- Various Stack Overflow discussions on block validation errors - Community solutions, not official guidance

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Official @wordpress packages, verified via npm and WordPress.org
- Architecture: HIGH - Patterns directly from WordPress Block Editor Handbook and official tutorials
- Pitfalls: HIGH - Sourced from GitHub issues, WordPress Core dev notes, and official documentation discussions
- Interactivity API: HIGH - Official WordPress 6.5 documentation and Make WordPress Core announcement
- Block hooks: MEDIUM - Feature introduced in 6.4, refined in 6.5/6.6, still evolving
- TypeScript integration: LOW - Mentioned as supported but core uses JSDoc; unclear how deeply to cover

**Research date:** 2026-02-06
**Valid until:** Approximately 30 days (stable APIs, but Interactivity API and Block Hooks still evolving in WP 6.5+)

**Key version markers:**
- WordPress 6.3+: apiVersion 3, iframe editor
- WordPress 6.4+: Block Hooks API
- WordPress 6.5+: Interactivity API, viewScriptModule
- WordPress 6.2+: InspectorControls group prop (Settings/Appearance tabs)
- WordPress 5.8+: block.json metadata support
