# WordPress Interactivity API Reference (WP 6.5+)

This reference catalogs the WordPress Interactivity API — a declarative system for adding frontend interactions to blocks using `data-wp-*` directives. Covers all directives, state management, server-side setup, context vs state, and integration with dynamic blocks. WordPress 6.5+ only.

## Quick Reference Table

### All data-wp-* Directives

| Directive | Syntax | Purpose | Example Use Case |
|-----------|--------|---------|------------------|
| `data-wp-interactive` | `data-wp-interactive="namespace"` | Marks interactive region root | Enable Interactivity API for block |
| `data-wp-bind` | `data-wp-bind--{attribute}` | Bind attribute to state | Toggle `hidden`, `disabled`, `src` |
| `data-wp-on` | `data-wp-on--{event}` | Attach event listener | Click handlers, input changes |
| `data-wp-class` | `data-wp-class--{className}` | Toggle CSS class | Show/hide elements, active states |
| `data-wp-style` | `data-wp-style--{property}` | Set inline style | Dynamic colors, widths, transforms |
| `data-wp-text` | `data-wp-text` | Set text content | Counters, dynamic labels |
| `data-wp-context` | `data-wp-context` | Define local context | Scoped state for DOM subtree |
| `data-wp-watch` | `data-wp-watch` | Run side effect on state change | Track analytics, update external services |
| `data-wp-init` | `data-wp-init` | Run callback on mount | Initialize third-party libraries |
| `data-wp-each` | `data-wp-each` | Render list from array | Dynamic lists, repeating elements |

### Version Requirements

| Feature | WordPress Version | Notes |
|---------|-------------------|-------|
| **Basic Interactivity API** | 6.5+ | Initial release |
| **viewScriptModule field** | 6.5+ | Required for Interactivity API scripts |
| **Enhanced directives** | 6.6+ | Additional directive features |
| **data-wp-each** | 6.6+ | List rendering support |

## What is the Interactivity API

The Interactivity API is a **declarative frontend interaction system** introduced in WordPress 6.5.

**KEY DISTINCTION:** Interactivity API is for the **FRONTEND ONLY** — the block editor still uses React.

### Architecture Overview

**Editor (React):**
- `edit.js` — React component with useBlockProps, InspectorControls
- Full React ecosystem (hooks, state, components)

**Frontend (Interactivity API):**
- `render.php` — Outputs HTML with `data-wp-*` directives
- `view.js` — Defines store with state and actions
- Declarative state binding via directives

**Not a replacement for React in editor** — Interactivity API is specifically for frontend views.

### Why Use Interactivity API

**Problems it solves:**
- Custom JavaScript bundles for simple interactions
- Managing state across multiple block instances
- Server-side state hydration
- Consistent patterns for frontend interaction

**Use when:**
- Need frontend interactions (tabs, accordions, toggles, counters)
- Targeting WordPress 6.5+
- Want standardized approach instead of custom JavaScript

**Don't use when:**
- Targeting WordPress < 6.5
- Need extremely complex state management (use custom React app)
- Simple CSS-only interactions suffice

## Setup Requirements

### block.json Configuration

**CRITICAL:** Use `viewScriptModule`, NOT `viewScript`:

```json
{
	"apiVersion": 3,
	"name": "my-plugin/interactive-block",
	"title": "Interactive Block",
	"viewScriptModule": "file:./view.js",
	"render": "file:./render.php"
}
```

**Why viewScriptModule:**
- Loads as ES module (required for `import` statements)
- Interactivity API uses `@wordpress/interactivity` package
- viewScript loads as classic script (doesn't support imports)

**BAD: Using viewScript**

```json
{
	"viewScript": "file:./view.js"
}
```

**Error:** `import` statements fail in browser — viewScript not loaded as module.

**GOOD: Using viewScriptModule**

```json
{
	"viewScriptModule": "file:./view.js"
}
```

Loads as ES module, supports imports.

### PHP Setup (wp_interactivity_state)

**render.php:**

```php
<?php
wp_interactivity_state( 'myPlugin', array(
	'isOpen' => false,
	'count'  => 0,
) );

$wrapper_attributes = get_block_wrapper_attributes();
?>
<div
	<?php echo $wrapper_attributes; ?>
	data-wp-interactive="myPlugin"
	data-wp-context='<?php echo wp_json_encode( array( 'id' => uniqid() ) ); ?>'
>
	<button data-wp-on--click="actions.toggle">
		Toggle
	</button>
	<div data-wp-bind--hidden="!state.isOpen">
		<p>Content revealed!</p>
	</div>
</div>
```

**wp_interactivity_state() parameters:**
1. Namespace (string) — Plugin identifier, matches `data-wp-interactive`
2. State (array) — Initial state object

### JavaScript Setup (store)

**view.js:**

```javascript
import { store } from '@wordpress/interactivity';

store( 'myPlugin', {
	state: {
		isOpen: false,
		count: 0,
	},
	actions: {
		toggle: ( { state } ) => {
			state.isOpen = ! state.isOpen;
		},
		increment: ( { state } ) => {
			state.count += 1;
		},
	},
	callbacks: {
		logCount: ( { state } ) => {
			console.log( 'Count:', state.count );
		},
	},
} );
```

**store() parameters:**
1. Namespace (string) — Must match `data-wp-interactive` and wp_interactivity_state namespace
2. Store configuration (object) — state, actions, callbacks

## Directive Catalog

### data-wp-interactive

**Purpose:** Marks the root element of an interactive region. Required for all other directives to work.

**Syntax:**

```html
<div data-wp-interactive="myPlugin">
	<!-- Directives work inside here -->
</div>
```

**Rules:**
- Must be present on parent element
- Value must match store namespace
- Can have multiple interactive regions with different namespaces

**BAD: Missing data-wp-interactive**

```html
<div>
	<button data-wp-on--click="actions.toggle">Toggle</button>
</div>
```

Button click handler doesn't work — no interactive region defined.

**GOOD: With data-wp-interactive**

```html
<div data-wp-interactive="myPlugin">
	<button data-wp-on--click="actions.toggle">Toggle</button>
</div>
```

### data-wp-bind

**Purpose:** Bind element attribute to state value.

**Syntax:**

```html
<div data-wp-bind--{attribute}="state.value"></div>
```

**Examples:**

```html
<!-- Bind hidden attribute -->
<div data-wp-bind--hidden="!state.isOpen">
	Content
</div>

<!-- Bind disabled attribute -->
<button data-wp-bind--disabled="state.isProcessing">
	Submit
</button>

<!-- Bind src attribute -->
<img data-wp-bind--src="state.imageUrl" alt="Dynamic image" />

<!-- Bind aria-expanded -->
<button data-wp-bind--aria-expanded="state.isExpanded">
	Menu
</button>
```

**Boolean attributes:** Set to true/false to add/remove attribute.

**BAD: Binding to undefined state**

```html
<div data-wp-bind--hidden="state.nonExistentProperty">
	Content
</div>
```

**state.nonExistentProperty** undefined — binding fails silently.

**GOOD: Binding to defined state**

```javascript
store( 'myPlugin', {
	state: {
		isOpen: false,
	},
} );
```

```html
<div data-wp-bind--hidden="!state.isOpen">
	Content
</div>
```

### data-wp-on

**Purpose:** Attach event listener to element.

**Syntax:**

```html
<element data-wp-on--{event}="actions.handlerName"></element>
```

**Common events:**

```html
<!-- Click -->
<button data-wp-on--click="actions.handleClick">Click Me</button>

<!-- Input change -->
<input data-wp-on--input="actions.handleInput" />

<!-- Form submit -->
<form data-wp-on--submit="actions.handleSubmit">

<!-- Mouse enter/leave -->
<div data-wp-on--mouseenter="actions.showTooltip"
     data-wp-on--mouseleave="actions.hideTooltip">
</div>

<!-- Focus/blur -->
<input data-wp-on--focus="actions.handleFocus"
       data-wp-on--blur="actions.handleBlur" />
```

**Action implementation:**

```javascript
store( 'myPlugin', {
	actions: {
		handleClick: ( { state, event } ) => {
			event.preventDefault(); // Access native event
			state.clickCount += 1;
		},
		handleInput: ( { state, event } ) => {
			state.inputValue = event.target.value;
		},
	},
} );
```

**Action parameters:**
- `state` — Store state (reactive)
- `context` — Local context
- `event` — Native DOM event

### data-wp-class

**Purpose:** Toggle CSS class based on state.

**Syntax:**

```html
<element data-wp-class--{className}="state.condition"></element>
```

**Examples:**

```html
<!-- Toggle active class -->
<button data-wp-class--active="state.isActive">
	Tab
</button>

<!-- Toggle multiple classes -->
<div data-wp-class--open="state.isOpen"
     data-wp-class--loading="state.isLoading"
     data-wp-class--error="state.hasError">
	Content
</div>
```

**How it works:**
- If condition is truthy → add class
- If condition is falsy → remove class

**Result:**

```html
<!-- When state.isActive = true -->
<button class="active">Tab</button>

<!-- When state.isActive = false -->
<button class="">Tab</button>
```

**BAD: Manipulating className directly**

```javascript
actions: {
	toggle: ( { event } ) => {
		event.target.classList.toggle( 'active' ); // Anti-pattern
	},
}
```

Doesn't sync with state, not reactive.

**GOOD: Using data-wp-class**

```javascript
actions: {
	toggle: ( { state } ) => {
		state.isActive = ! state.isActive;
	},
}
```

```html
<button data-wp-class--active="state.isActive" data-wp-on--click="actions.toggle">
	Toggle
</button>
```

### data-wp-style

**Purpose:** Set inline CSS style property based on state.

**Syntax:**

```html
<element data-wp-style--{property}="state.value"></element>
```

**Examples:**

```html
<!-- Background color -->
<div data-wp-style--background-color="state.bgColor">
	Content
</div>

<!-- Width -->
<div data-wp-style--width="state.width + 'px'">
	Resizable
</div>

<!-- Transform -->
<div data-wp-style--transform="'rotate(' + state.rotation + 'deg)'">
	Rotated
</div>

<!-- Opacity -->
<div data-wp-style--opacity="state.opacity">
	Fade
</div>
```

**CSS property naming:** Use kebab-case (background-color, not backgroundColor).

**Dynamic values:**

```javascript
store( 'myPlugin', {
	state: {
		percentage: 50,
	},
} );
```

```html
<div data-wp-style--width="state.percentage + '%'">
	Progress Bar
</div>
```

Result: `<div style="width: 50%">Progress Bar</div>`

### data-wp-text

**Purpose:** Set text content of element from state.

**Syntax:**

```html
<element data-wp-text="state.value"></element>
```

**Examples:**

```html
<!-- Counter -->
<span data-wp-text="state.count"></span>

<!-- Dynamic label -->
<p data-wp-text="state.message"></p>

<!-- Formatted value -->
<span data-wp-text="'$' + state.price.toFixed(2)"></span>
```

**Replaces entire text content:**

```html
<button>
	<span data-wp-text="state.buttonText"></span>
</button>
```

If `state.buttonText = "Click Me"`, result:

```html
<button>
	<span>Click Me</span>
</button>
```

**BAD: Using innerHTML**

```javascript
actions: {
	updateText: ( { event } ) => {
		event.target.innerHTML = 'New text'; // XSS risk, not reactive
	},
}
```

**GOOD: Using data-wp-text**

```javascript
actions: {
	updateText: ( { state } ) => {
		state.buttonText = 'New text';
	},
}
```

```html
<button data-wp-text="state.buttonText"></button>
```

### data-wp-context

**Purpose:** Define local context for a DOM subtree (scoped state).

**Syntax:**

```html
<div data-wp-context='<?php echo wp_json_encode( array( 'key' => 'value' ) ); ?>'>
	<!-- Context available to children -->
</div>
```

**Example:**

```html
<div data-wp-interactive="myPlugin">
	<?php foreach ( $items as $item ) : ?>
		<div data-wp-context='<?php echo wp_json_encode( array( 'itemId' => $item->ID ) ); ?>'>
			<h3><?php echo esc_html( $item->title ); ?></h3>
			<button data-wp-on--click="actions.selectItem">Select</button>
		</div>
	<?php endforeach; ?>
</div>
```

**Access context in actions:**

```javascript
store( 'myPlugin', {
	actions: {
		selectItem: ( { context } ) => {
			const itemId = context.itemId;
			console.log( 'Selected item:', itemId );
		},
	},
} );
```

**Context vs State:**
- **State:** Global, shared across all instances
- **Context:** Local, scoped to DOM subtree

### data-wp-watch

**Purpose:** Run side effect when state changes (like useEffect in React).

**Syntax:**

```html
<element data-wp-watch="callbacks.watcherName"></element>
```

**Example:**

```html
<div data-wp-interactive="myPlugin">
	<input data-wp-on--input="actions.updateSearch" />
	<div data-wp-watch="callbacks.logSearch"></div>
</div>
```

**Callback implementation:**

```javascript
store( 'myPlugin', {
	state: {
		searchTerm: '',
	},
	actions: {
		updateSearch: ( { state, event } ) => {
			state.searchTerm = event.target.value;
		},
	},
	callbacks: {
		logSearch: ( { state } ) => {
			// Runs whenever searchTerm changes
			console.log( 'Search term:', state.searchTerm );

			// Side effects: analytics, external API calls
			if ( state.searchTerm.length > 3 ) {
				// Trigger search
			}
		},
	},
} );
```

**Use cases:**
- Analytics tracking
- Updating external services
- Syncing with localStorage
- Triggering API requests

### data-wp-init

**Purpose:** Run callback once when element mounts.

**Syntax:**

```html
<element data-wp-init="callbacks.initCallback"></element>
```

**Example:**

```html
<div data-wp-interactive="myPlugin">
	<div data-wp-init="callbacks.initialize">
		Content
	</div>
</div>
```

**Callback implementation:**

```javascript
store( 'myPlugin', {
	callbacks: {
		initialize: ( { state } ) => {
			// Runs once on mount
			console.log( 'Component initialized' );

			// Initialize third-party libraries
			// Set up event listeners
			// Load data
		},
	},
} );
```

**Use cases:**
- Initialize third-party libraries (chart.js, map libraries)
- Load initial data
- Set up global event listeners
- One-time setup logic

### data-wp-each

**Purpose:** Render list from array (WordPress 6.6+).

**Syntax:**

```html
<template data-wp-each="state.items">
	<!-- Template for each item -->
	<li data-wp-text="context.item.name"></li>
</template>
```

**Complete example:**

```javascript
store( 'myPlugin', {
	state: {
		items: [
			{ id: 1, name: 'Item 1' },
			{ id: 2, name: 'Item 2' },
			{ id: 3, name: 'Item 3' },
		],
	},
} );
```

```html
<div data-wp-interactive="myPlugin">
	<ul>
		<template data-wp-each="state.items">
			<li>
				<span data-wp-text="context.item.name"></span>
				<button data-wp-on--click="actions.removeItem">Remove</button>
			</li>
		</template>
	</ul>
</div>
```

**Each iteration:**
- Creates new context with `item` property
- Access current item via `context.item`

## State Management

### Server-Side State (wp_interactivity_state)

**Define initial state in PHP:**

```php
<?php
wp_interactivity_state( 'myPlugin', array(
	'isOpen'     => false,
	'count'      => 0,
	'items'      => array(),
	'user'       => array(
		'name'  => get_current_user()->display_name,
		'email' => get_current_user()->user_email,
	),
) );
?>
```

**Server-side state is hydrated to JavaScript** — becomes initial state in store.

**Security consideration:** Never put sensitive data in interactivity state. State is public, visible in page source.

**BAD: Sensitive data in state**

```php
wp_interactivity_state( 'myPlugin', array(
	'apiKey'       => 'secret-key-123', // Exposed in HTML!
	'userPassword' => get_user_meta( $user_id, 'password' ), // Never!
) );
```

**GOOD: Public data only**

```php
wp_interactivity_state( 'myPlugin', array(
	'userName'    => get_current_user()->display_name,
	'isLoggedIn'  => is_user_logged_in(),
	'postCount'   => (int) wp_count_posts()->publish,
) );
```

### Client-Side State (store)

**Define store in view.js:**

```javascript
import { store } from '@wordpress/interactivity';

store( 'myPlugin', {
	state: {
		// Merges with server-side state from wp_interactivity_state()
		localCounter: 0,
		uiState: 'idle',
	},
	actions: {
		// User-triggered functions
	},
	callbacks: {
		// Side effects and computed values
	},
} );
```

**State merging:** Server-side state merged with client-side state. Conflicts: server-side wins.

### Accessing State in Directives

**Direct state access:**

```html
<span data-wp-text="state.count"></span>
```

**Computed expressions:**

```html
<div data-wp-bind--hidden="state.count === 0"></div>
<span data-wp-text="'Total: ' + state.count"></span>
<div data-wp-class--active="state.count > 5"></div>
```

**Nested state:**

```javascript
state: {
	user: {
		name: 'John',
		email: 'john@example.com',
	},
}
```

```html
<span data-wp-text="state.user.name"></span>
<span data-wp-text="state.user.email"></span>
```

## Actions and Callbacks

### Actions (User-Triggered Functions)

**Purpose:** Handle user interactions (clicks, inputs, submits).

**Definition:**

```javascript
store( 'myPlugin', {
	actions: {
		actionName: ( { state, context, event } ) => {
			// Modify state
			state.count += 1;

			// Access context
			const id = context.itemId;

			// Access native event
			event.preventDefault();
		},
	},
} );
```

**Action parameters:**
- `state` — Store state (reactive, mutations trigger updates)
- `context` — Local context from data-wp-context
- `event` — Native DOM event

**Usage in HTML:**

```html
<button data-wp-on--click="actions.actionName">Click</button>
```

### Callbacks (Side Effects and Derived Values)

**Purpose:** React to state changes, compute derived values.

**Definition:**

```javascript
store( 'myPlugin', {
	callbacks: {
		callbackName: ( { state, context } ) => {
			// Side effect when dependencies change
			console.log( 'State changed:', state.value );

			// Can modify state (triggers reactivity)
			state.derivedValue = state.value * 2;
		},
	},
} );
```

**Usage in HTML:**

```html
<div data-wp-watch="callbacks.callbackName"></div>
```

**Difference from actions:**
- **Actions:** Triggered by user events
- **Callbacks:** Triggered by state changes (watchers, init)

### Accessing Store Parameter

All action and callback functions receive store parameter object:

```javascript
actions: {
	example: ( { state, context, event } ) => {
		// state — Reactive store state
		// context — Local context from nearest data-wp-context
		// event — Native DOM event (actions only)
	},
},
callbacks: {
	example: ( { state, context } ) => {
		// state — Reactive store state
		// context — Local context
		// No event in callbacks
	},
}
```

## Server-Side Setup Patterns

### wp_interactivity_state() Placement

**Call BEFORE template output:**

```php
<?php
// render.php

wp_interactivity_state( 'myPlugin', array(
	'isOpen' => false,
) );

$wrapper_attributes = get_block_wrapper_attributes();
?>
<div <?php echo $wrapper_attributes; ?> data-wp-interactive="myPlugin">
	<!-- Template -->
</div>
```

**Multiple blocks on same page:** State shared across all instances.

```php
// Each block instance
wp_interactivity_state( 'myPlugin', array(
	'globalCounter' => 0, // Shared across all instances
) );
```

State merged — all instances share same `globalCounter`.

**Instance-specific state:** Use data-wp-context for scoped state.

### wp_interactivity_config()

**For configuration data (not reactive state):**

```php
wp_interactivity_config( 'myPlugin', array(
	'apiEndpoint' => rest_url( 'myPlugin/v1/' ),
	'nonce'       => wp_create_nonce( 'myPlugin-nonce' ),
) );
```

**Access in JavaScript:**

```javascript
import { getConfig } from '@wordpress/interactivity';

const config = getConfig( 'myPlugin' );
console.log( config.apiEndpoint );
```

**Difference from state:**
- **State:** Reactive, changes trigger UI updates
- **Config:** Static configuration, doesn't change

### Dynamic State from Block Attributes

```php
<?php
// render.php
$initial_count = isset( $attributes['initialCount'] ) ? absint( $attributes['initialCount'] ) : 0;

wp_interactivity_state( 'myPlugin', array(
	'count' => $initial_count,
) );
?>
```

**Each block instance can have different initial state** based on attributes.

## Context vs State

### State: Global, Shared

**State** is global to the namespace, shared across all instances.

```javascript
store( 'myPlugin', {
	state: {
		globalCounter: 0, // Shared by all instances
	},
} );
```

**All blocks with `data-wp-interactive="myPlugin"`** share the same `globalCounter`.

**Use state when:**
- Data should be shared across instances
- One action affects all instances
- Global UI state (modal open/closed)

### Context: Local, Scoped

**Context** is local to a DOM subtree, scoped to specific instance.

```html
<div data-wp-context='{"instanceId": 1}'>
	<button data-wp-on--click="actions.log">Log ID</button>
</div>

<div data-wp-context='{"instanceId": 2}'>
	<button data-wp-on--click="actions.log">Log ID</button>
</div>
```

```javascript
actions: {
	log: ( { context } ) => {
		console.log( context.instanceId ); // Different per instance
	},
}
```

**Use context when:**
- Data is instance-specific
- Each block instance has unique ID
- List items with individual state

**Context inheritance:** Child elements inherit parent context.

```html
<div data-wp-context='{"color": "red"}'>
	<div data-wp-context='{"size": "large"}'>
		<!-- Has both color and size -->
		<button data-wp-on--click="actions.log">Log</button>
	</div>
</div>
```

```javascript
actions: {
	log: ( { context } ) => {
		console.log( context.color ); // "red"
		console.log( context.size );  // "large"
	},
}
```

## Integration with Dynamic Blocks

### Complete Dynamic Block with Interactivity API

**block.json:**

```json
{
	"apiVersion": 3,
	"name": "my-plugin/counter",
	"title": "Counter Block",
	"category": "widgets",
	"attributes": {
		"initialCount": {
			"type": "number",
			"default": 0
		}
	},
	"supports": {
		"interactivity": true
	},
	"viewScriptModule": "file:./view.js",
	"render": "file:./render.php"
}
```

**render.php:**

```php
<?php
$initial_count = isset( $attributes['initialCount'] ) ? absint( $attributes['initialCount'] ) : 0;

wp_interactivity_state( 'myPlugin/counter', array(
	'count' => $initial_count,
) );

$wrapper_attributes = get_block_wrapper_attributes();
?>
<div
	<?php echo $wrapper_attributes; ?>
	data-wp-interactive="myPlugin/counter"
	data-wp-context='<?php echo wp_json_encode( array( 'id' => uniqid() ) ); ?>'
>
	<p>
		Count: <span data-wp-text="state.count"></span>
	</p>
	<button data-wp-on--click="actions.increment">+</button>
	<button data-wp-on--click="actions.decrement">-</button>
	<button data-wp-on--click="actions.reset">Reset</button>
</div>
```

**view.js:**

```javascript
import { store } from '@wordpress/interactivity';

store( 'myPlugin/counter', {
	state: {
		count: 0,
	},
	actions: {
		increment: ( { state } ) => {
			state.count += 1;
		},
		decrement: ( { state } ) => {
			state.count -= 1;
		},
		reset: ( { state } ) => {
			state.count = 0;
		},
	},
} );
```

### Interactivity with InnerBlocks

**render.php:**

```php
<?php
wp_interactivity_state( 'myPlugin', array(
	'isOpen' => false,
) );

$wrapper_attributes = get_block_wrapper_attributes();
?>
<div
	<?php echo $wrapper_attributes; ?>
	data-wp-interactive="myPlugin"
>
	<button data-wp-on--click="actions.toggle">
		<span data-wp-text="state.isOpen ? 'Hide' : 'Show'"></span>
	</button>

	<div data-wp-bind--hidden="!state.isOpen">
		<?php echo $content; // InnerBlocks ?>
	</div>
</div>
```

**save.js:**

```javascript
import { InnerBlocks } from '@wordpress/block-editor';

function save() {
	return <InnerBlocks.Content />; // Save inner blocks
}
```

**view.js:**

```javascript
store( 'myPlugin', {
	state: {
		isOpen: false,
	},
	actions: {
		toggle: ( { state } ) => {
			state.isOpen = ! state.isOpen;
		},
	},
} );
```

## Common Interactivity API Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| **Using viewScript instead of viewScriptModule** | Import statements fail | Use `"viewScriptModule": "file:./view.js"` |
| **Missing data-wp-interactive** | Directives don't work | Add `data-wp-interactive="namespace"` to wrapper |
| **Namespace mismatch** | State not found | Match namespace in wp_interactivity_state(), store(), data-wp-interactive |
| **Mutating state outside actions** | State changes ignored | Only mutate state in actions/callbacks |
| **Sensitive data in state** | Security risk | Never put API keys, passwords, private data in state |
| **Using in block editor** | Interactivity API doesn't run in editor | Use only for frontend, editor still uses React |
| **Forgetting JSON encoding for context** | Invalid attribute value | Use `wp_json_encode()` for data-wp-context |
| **Accessing undefined state** | Binding fails silently | Ensure all state properties defined in store |

## When NOT to Use Interactivity API

### CSS-Only Interactions

**Don't use Interactivity API for:**
- Hover effects
- CSS transitions/animations
- Pure styling changes

**Use CSS:**

```css
.accordion-item:hover {
	background-color: #f0f0f0;
}

.button {
	transition: transform 0.2s;
}

.button:active {
	transform: scale(0.95);
}
```

### Blocks Targeting WP < 6.5

Interactivity API requires WordPress 6.5+.

**If targeting older versions:**
- Use custom JavaScript with vanilla JS or small library
- Use React (requires build step)
- Consider progressive enhancement (basic HTML, JavaScript for enhancement)

### Extremely Complex State Management

**Interactivity API is not React** — for complex apps:
- Consider custom React application
- Use established state management (Redux, Zustand)
- Build SPA with React Router

**Interactivity API sweet spot:** Simple to medium frontend interactions.

## Cross-References

- **block.json viewScriptModule configuration:** See block-json-guide.md
- **Dynamic blocks with render.php:** See dynamic-blocks-guide.md
- **React patterns for block editor:** See editor-patterns.md
- **Security considerations for state data:** See wp-security-review skill
