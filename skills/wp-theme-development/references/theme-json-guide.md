# WordPress theme.json v3 Complete Reference

This reference provides exhaustive coverage of the theme.json v3 schema — the central configuration file for WordPress block themes (WordPress 6.6+). Every top-level key, property, validation rule, v2-to-v3 migration step, and common mistake is documented with examples following WordPress standards.

## Quick Reference Table

### Top-Level Keys

| Key | Type | Required | Description | Example Value |
|-----|------|----------|-------------|---------------|
| `$schema` | string | No | JSON schema URL for IDE validation | `"https://schemas.wp.org/trunk/theme.json"` |
| `version` | number | Yes | Schema version (1, 2, or 3) | `3` |
| `settings` | object | No | Global settings for blocks and editor | See Settings section |
| `styles` | object | No | Global styles and block-specific styles | See Styles section |
| `customTemplates` | array | No | Custom template definitions | `[{"name": "page-no-title", "title": "Page Without Title", "postTypes": ["page"]}]` |
| `templateParts` | array | No | Template part definitions | `[{"name": "header", "title": "Header", "area": "header"}]` |
| `patterns` | array | No | Pattern slugs from WordPress.org directory | `["core/text-two-columns"]` |

## Version History and Migration

### Version Overview

**theme.json v1** (WordPress 5.8) - DEPRECATED

```json
// BAD: Do not use v1
{
	"version": 1,
	"settings": {}
}
// CRITICAL: Version 1 is deprecated, update to version 3
```

**theme.json v2** (WordPress 5.9-6.5) - WARNING

```json
// WARNING: v2 still works but should upgrade
{
	"version": 2,
	"settings": {
		"typography": {
			"fontSizes": [
				{"slug": "small", "size": "14px", "name": "Small"}
			]
		}
	}
}
// WARNING: Version 2 is outdated, upgrade to v3 for latest features
```

**theme.json v3** (WordPress 6.6+) - CURRENT

```json
// GOOD: Current version
{
	"$schema": "https://schemas.wp.org/trunk/theme.json",
	"version": 3,
	"settings": {
		"typography": {
			"defaultFontSizes": false,
			"fontSizes": [
				{"slug": "small", "size": "0.875rem", "name": "Small"}
			]
		}
	}
}
```

### Critical v2 to v3 Breaking Changes

**Breaking Change 1: defaultFontSizes Behavior Reversed**

In v2, defining custom `fontSizes` automatically hid WordPress default sizes.
In v3, WordPress default sizes ALWAYS show unless explicitly disabled.

```json
// BAD: v2 behavior expected in v3
{
	"version": 3,
	"settings": {
		"typography": {
			"fontSizes": [
				{"slug": "small", "size": "0.875rem", "name": "Small"}
			]
		}
	}
}
// Result: WordPress default sizes + theme sizes both show (unwanted duplication)

// GOOD: v3 explicit control
{
	"version": 3,
	"settings": {
		"typography": {
			"defaultFontSizes": false,
			"fontSizes": [
				{"slug": "small", "size": "0.875rem", "name": "Small"}
			]
		}
	}
}
// Result: Only theme sizes show
```

**Breaking Change 2: defaultSpacingSizes Behavior Reversed**

Same pattern as fontSizes — v3 requires explicit `defaultSpacingSizes: false`.

```json
// BAD: v2 behavior expected in v3
{
	"version": 3,
	"settings": {
		"spacing": {
			"spacingSizes": [
				{"slug": "30", "size": "1rem", "name": "Small"}
			]
		}
	}
}
// Result: WordPress default spacing + theme spacing (unwanted duplication)

// GOOD: v3 explicit control
{
	"version": 3,
	"settings": {
		"spacing": {
			"defaultSpacingSizes": false,
			"spacingSizes": [
				{"slug": "30", "size": "1rem", "name": "Small"}
			]
		}
	}
}
// Result: Only theme spacing shows
```

### v2 to v3 Migration Steps

1. **Update version number**
   ```json
   {
   	"version": 3
   }
   ```

2. **Add defaultFontSizes if theme defines fontSizes**
   ```json
   {
   	"settings": {
   		"typography": {
   			"defaultFontSizes": false,
   			"fontSizes": []
   		}
   	}
   }
   ```

3. **Add defaultSpacingSizes if theme defines spacingSizes or spacingScale**
   ```json
   {
   	"settings": {
   		"spacing": {
   			"defaultSpacingSizes": false,
   			"spacingSizes": []
   		}
   	}
   }
   ```

4. **Test in WordPress 6.6+ Site Editor**
   - Open Site Editor > Styles panel
   - Verify only expected font sizes appear
   - Verify only expected spacing presets appear
   - Check that no duplicate sizes show

## Settings Deep Dive

The `settings` object controls available options in the block editor and Site Editor.

### settings.color

```json
{
	"settings": {
		"color": {
			"defaultPalette": false,
			"defaultGradients": false,
			"defaultDuotone": false,
			"custom": true,
			"customGradient": true,
			"customDuotone": true,
			"link": true,
			"heading": true,
			"button": true,
			"caption": true,
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
			"gradients": [
				{
					"slug": "primary-to-secondary",
					"gradient": "linear-gradient(135deg, #0073aa 0%, #005177 100%)",
					"name": "Primary to Secondary"
				}
			],
			"duotone": [
				{
					"slug": "blue-orange",
					"colors": ["#0073aa", "#ff6900"],
					"name": "Blue and Orange"
				}
			]
		}
	}
}
```

**Color property catalog:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `defaultPalette` | boolean | true | Show WordPress default colors |
| `defaultGradients` | boolean | true | Show WordPress default gradients |
| `defaultDuotone` | boolean | true | Show WordPress default duotone filters |
| `custom` | boolean | true | Allow custom color picker |
| `customGradient` | boolean | true | Allow custom gradient creation |
| `customDuotone` | boolean | true | Allow custom duotone creation |
| `link` | boolean | false | Enable link color controls |
| `heading` | boolean | false | Enable heading color controls |
| `button` | boolean | false | Enable button color controls |
| `caption` | boolean | false | Enable caption color controls |
| `palette` | array | [] | Theme color presets |
| `gradients` | array | [] | Theme gradient presets |
| `duotone` | array | [] | Theme duotone presets |

**Generated CSS variables from palette:**

```css
/* Generated automatically from palette */
--wp--preset--color--primary: #0073aa;
--wp--preset--color--secondary: #005177;
--wp--preset--color--foreground: #333333;
--wp--preset--color--background: #ffffff;
```

**Common mistakes:**

```json
// BAD: Hardcoded hex values in styles
{
	"styles": {
		"color": {
			"background": "#ffffff"
		}
	}
}
// Users can't change via Site Editor

// GOOD: Reference palette via CSS variable
{
	"settings": {
		"color": {
			"palette": [
				{"slug": "background", "color": "#ffffff", "name": "Background"}
			]
		}
	},
	"styles": {
		"color": {
			"background": "var(--wp--preset--color--background)"
		}
	}
}
// Users can customize background color in Site Editor
```

### settings.typography

```json
{
	"settings": {
		"typography": {
			"defaultFontSizes": false,
			"fluid": true,
			"customFontSize": true,
			"fontWeight": true,
			"fontStyle": true,
			"textTransform": true,
			"textDecoration": true,
			"letterSpacing": true,
			"lineHeight": true,
			"dropCap": true,
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
					"size": "clamp(1.25rem, 1.125rem + 0.625vw, 1.5rem)",
					"name": "Large",
					"fluid": {
						"min": "1.25rem",
						"max": "1.5rem"
					}
				}
			]
		}
	}
}
```

**Typography property catalog:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `defaultFontSizes` | boolean | true (v3) | Show WordPress default font sizes |
| `fluid` | boolean/object | false | Enable fluid typography (clamp values) |
| `customFontSize` | boolean | true | Allow custom font size input |
| `fontWeight` | boolean | false | Enable font weight controls |
| `fontStyle` | boolean | false | Enable italic toggle |
| `textTransform` | boolean | false | Enable uppercase/lowercase controls |
| `textDecoration` | boolean | false | Enable underline/strikethrough |
| `letterSpacing` | boolean | false | Enable letter spacing |
| `lineHeight` | boolean | false | Enable line height controls |
| `dropCap` | boolean | true | Enable drop cap option |
| `fontFamilies` | array | [] | Theme font family presets |
| `fontSizes` | array | [] | Theme font size presets |

**Font file paths:**

```json
// BAD: Absolute URL to CDN (GDPR concerns)
{
	"fontFace": [
		{
			"src": ["https://fonts.googleapis.com/css2?family=Inter"]
		}
	]
}
// Privacy issue: Loads from Google servers

// GOOD: Local font hosting
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
// Privacy-compliant, faster, offline-capable
```

**Fluid typography:**

```json
{
	"settings": {
		"typography": {
			"fluid": true,
			"fontSizes": [
				{
					"slug": "heading",
					"size": "2rem",
					"name": "Heading",
					"fluid": {
						"min": "1.5rem",
						"max": "2.5rem"
					}
				}
			]
		}
	}
}
// Generates: font-size: clamp(1.5rem, calc(1.5rem + ((1vw - 0.48rem) * 2.5)), 2.5rem);
```

### settings.spacing

```json
{
	"settings": {
		"spacing": {
			"defaultSpacingSizes": false,
			"customSpacingSize": true,
			"padding": true,
			"margin": true,
			"blockGap": true,
			"units": ["px", "em", "rem", "vh", "vw", "%"],
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
			"spacingScale": {
				"operator": "*",
				"increment": 1.5,
				"steps": 7,
				"mediumStep": 1.5,
				"unit": "rem"
			}
		}
	}
}
```

**Spacing property catalog:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `defaultSpacingSizes` | boolean | true (v3) | Show WordPress default spacing presets |
| `customSpacingSize` | boolean | true | Allow custom spacing input |
| `padding` | boolean | false | Enable padding controls |
| `margin` | boolean | false | Enable margin controls |
| `blockGap` | boolean | false | Enable block gap controls |
| `units` | array | ["px", "em", "rem", "vh", "vw", "%"] | Available spacing units |
| `spacingSizes` | array | [] | Theme spacing presets |
| `spacingScale` | object | null | Auto-generate spacing scale |

**spacingScale vs spacingSizes:**

```json
// Option 1: Manual presets
{
	"spacingSizes": [
		{"slug": "30", "size": "1rem", "name": "Small"},
		{"slug": "40", "size": "1.5rem", "name": "Medium"}
	]
}

// Option 2: Auto-generated scale
{
	"spacingScale": {
		"operator": "*",
		"increment": 1.5,
		"steps": 7,
		"mediumStep": 1.5,
		"unit": "rem"
	}
}
// Generates: 0.67rem, 1rem, 1.5rem, 2.25rem, 3.375rem, 5.063rem, 7.594rem
```

### settings.layout

```json
{
	"settings": {
		"layout": {
			"contentSize": "640px",
			"wideSize": "1200px"
		},
		"useRootPaddingAwareAlignments": true
	}
}
```

**Layout properties:**

| Property | Type | Description |
|----------|------|-------------|
| `contentSize` | string | Max width for default content (applies to constrained layout) |
| `wideSize` | string | Max width for wide alignment |
| `useRootPaddingAwareAlignments` | boolean | Enable full-width blocks to escape root padding |

**CRITICAL: useRootPaddingAwareAlignments**

```json
// BAD: Root padding without useRootPaddingAwareAlignments
{
	"settings": {
		"layout": {
			"contentSize": "640px",
			"wideSize": "1200px"
		}
	},
	"styles": {
		"spacing": {
			"padding": {
				"left": "2rem",
				"right": "2rem"
			}
		}
	}
}
// Result: Full-width blocks don't reach viewport edges

// GOOD: useRootPaddingAwareAlignments enabled
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
// Result: Full-width blocks properly reach edges with negative margins
```

**CRITICAL: Must use object notation for padding**

```json
// BAD: CSS shorthand doesn't work with useRootPaddingAwareAlignments
{
	"settings": {
		"useRootPaddingAwareAlignments": true
	},
	"styles": {
		"spacing": {
			"padding": "2rem"
		}
	}
}
// Result: useRootPaddingAwareAlignments won't work

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

### settings.border

```json
{
	"settings": {
		"border": {
			"color": true,
			"radius": true,
			"style": true,
			"width": true
		}
	}
}
```

### settings.shadow

```json
{
	"settings": {
		"shadow": {
			"defaultPresets": false,
			"presets": [
				{
					"slug": "natural",
					"shadow": "0 4px 6px rgba(0, 0, 0, 0.1)",
					"name": "Natural"
				}
			]
		}
	}
}
```

### settings.dimensions

```json
{
	"settings": {
		"dimensions": {
			"aspectRatio": true,
			"minHeight": true
		}
	}
}
```

### settings.position

```json
{
	"settings": {
		"position": {
			"sticky": true
		}
	}
}
```

### settings.appearanceTools

**Shortcut for enabling multiple controls:**

```json
{
	"settings": {
		"appearanceTools": true
	}
}
```

**Equivalent to:**

```json
{
	"settings": {
		"border": {
			"color": true,
			"radius": true,
			"style": true,
			"width": true
		},
		"color": {
			"link": true
		},
		"spacing": {
			"blockGap": true,
			"margin": true,
			"padding": true
		},
		"typography": {
			"lineHeight": true
		}
	}
}
```

## Styles Deep Dive

The `styles` object defines default styling for the site and individual blocks.

### Root-Level Styles

```json
{
	"styles": {
		"color": {
			"background": "var(--wp--preset--color--background)",
			"text": "var(--wp--preset--color--foreground)"
		},
		"typography": {
			"fontFamily": "var(--wp--preset--font-family--inter)",
			"fontSize": "var(--wp--preset--font-size--medium)",
			"lineHeight": "1.6",
			"fontWeight": "400"
		},
		"spacing": {
			"padding": {
				"top": "var(--wp--preset--spacing--50)",
				"right": "var(--wp--preset--spacing--50)",
				"bottom": "var(--wp--preset--spacing--50)",
				"left": "var(--wp--preset--spacing--50)"
			},
			"margin": {
				"top": "0",
				"bottom": "0"
			},
			"blockGap": "1.5rem"
		}
	}
}
```

**Applied to:** `<body>` element (root styles)

### Element Styles

```json
{
	"styles": {
		"elements": {
			"link": {
				"color": {
					"text": "var(--wp--preset--color--primary)"
				},
				":hover": {
					"color": {
						"text": "var(--wp--preset--color--secondary)"
					}
				},
				":focus": {
					"outline": {
						"width": "2px",
						"style": "solid",
						"color": "var(--wp--preset--color--primary)"
					}
				}
			},
			"heading": {
				"typography": {
					"fontWeight": "700",
					"lineHeight": "1.2"
				}
			},
			"h1": {
				"typography": {
					"fontSize": "var(--wp--preset--font-size--x-large)"
				}
			},
			"h2": {
				"typography": {
					"fontSize": "var(--wp--preset--font-size--large)"
				}
			},
			"button": {
				"color": {
					"background": "var(--wp--preset--color--primary)",
					"text": "var(--wp--preset--color--background)"
				},
				"border": {
					"radius": "4px"
				},
				":hover": {
					"color": {
						"background": "var(--wp--preset--color--secondary)"
					}
				}
			},
			"caption": {
				"typography": {
					"fontSize": "var(--wp--preset--font-size--small)"
				},
				"color": {
					"text": "#666666"
				}
			},
			"cite": {
				"typography": {
					"fontStyle": "italic"
				}
			}
		}
	}
}
```

**Available element selectors:**
- `link` — All links
- `heading` — All headings (h1-h6)
- `h1`, `h2`, `h3`, `h4`, `h5`, `h6` — Specific heading levels
- `button` — Button elements
- `caption` — Image captions
- `cite` — Citation elements

**Pseudo-classes:**
- `:hover`
- `:focus`
- `:active`
- `:visited` (links only)

### Block Styles

```json
{
	"styles": {
		"blocks": {
			"core/paragraph": {
				"spacing": {
					"margin": {
						"bottom": "1.5rem"
					}
				}
			},
			"core/heading": {
				"spacing": {
					"margin": {
						"top": "2rem",
						"bottom": "1rem"
					}
				}
			},
			"core/button": {
				"color": {
					"background": "var(--wp--preset--color--primary)",
					"text": "var(--wp--preset--color--background)"
				},
				"border": {
					"radius": "4px"
				},
				"spacing": {
					"padding": {
						"top": "0.75rem",
						"right": "1.5rem",
						"bottom": "0.75rem",
						"left": "1.5rem"
					}
				}
			},
			"core/image": {
				"border": {
					"radius": "8px"
				}
			},
			"core/post-content": {
				"spacing": {
					"blockGap": "2rem"
				}
			}
		}
	}
}
```

**Style precedence order (highest to lowest):**

1. User customizations (Global Styles UI)
2. Block-level styles in `styles.blocks`
3. Element styles in `styles.elements`
4. Root styles in `styles`
5. Block defaults from WordPress core

## templateParts Configuration

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
		},
		{
			"name": "post-meta",
			"title": "Post Meta",
			"area": "uncategorized"
		}
	]
}
```

**Template part properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Template part file name (without .html) |
| `title` | string | Yes | Human-readable title |
| `area` | string | No | Template part area designation |

**Valid area values:**
- `header` — Header area
- `footer` — Footer area
- `uncategorized` — Default area
- Custom areas (WordPress 6.0+)

**File mapping:**

```json
{
	"name": "header"
}
```

Maps to: `parts/header.html`

**Common mistake:**

```json
// BAD: File extension included
{
	"name": "header.html"
}
// Error: WordPress looks for parts/header.html.html

// GOOD: No file extension
{
	"name": "header"
}
// Correct: Maps to parts/header.html
```

## customTemplates Configuration

```json
{
	"customTemplates": [
		{
			"name": "page-no-title",
			"title": "Page Without Title",
			"postTypes": ["page"]
		},
		{
			"name": "single-product",
			"title": "Product Page",
			"postTypes": ["product"]
		},
		{
			"name": "archive-narrow",
			"title": "Narrow Archive",
			"postTypes": ["post"]
		}
	]
}
```

**Custom template properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Template file name (without .html) |
| `title` | string | Yes | Human-readable title shown in editor |
| `postTypes` | array | No | Post types that can use this template |

**File mapping:**

```json
{
	"name": "page-no-title"
}
```

Maps to: `templates/page-no-title.html`

**postTypes usage:**

If `postTypes` specified, template only appears as option for those post types.
If `postTypes` omitted, template available for all post types.

## patterns Configuration

```json
{
	"patterns": [
		"core/text-two-columns",
		"core/quote",
		"themeslug/custom-hero"
	]
}
```

**Pattern slug format:**
- Core patterns: `core/pattern-slug`
- Third-party patterns: `namespace/pattern-slug`

**What this does:**
Automatically registers patterns from WordPress.org pattern directory.

**Note:** This is different from theme-bundled patterns in `patterns/` directory, which are auto-registered by file presence.

## Validation Rules

### Required Fields

```json
{
	"version": 3
}
```

**CRITICAL:** `version` field is required. Theme.json without version will fail.

### Valid Value Types

```json
// BAD: version as string
{
	"version": "3"
}
// Error: version must be number

// GOOD: version as number
{
	"version": 3
}
```

### Deprecated Properties

**v1 deprecated properties:**

```json
// WARNING: These properties deprecated in v2+
{
	"settings": {
		"color": {
			"gradients": "add-default-gradients"
		}
	}
}
// Use "defaultGradients" instead
```

**v2 deprecated properties:**

None specific to v2, but v3 changes behavior of `defaultFontSizes` and `defaultSpacingSizes`.

### Unknown Properties

WordPress silently ignores unknown properties:

```json
{
	"settings": {
		"customFeature": true,
		"unknownProperty": "value"
	}
}
// No error, but properties have no effect
```

**Validation tip:** Use `$schema` property for IDE validation:

```json
{
	"$schema": "https://schemas.wp.org/trunk/theme.json",
	"version": 3
}
// IDE shows warnings for invalid properties
```

### Common JSON Syntax Errors

```json
// BAD: Trailing comma
{
	"version": 3,
	"settings": {},
}
// Error: JSON doesn't allow trailing commas

// BAD: Unquoted keys
{
	version: 3
}
// Error: Keys must be quoted strings

// BAD: Single quotes
{
	'version': 3
}
// Error: Must use double quotes
```

## useRootPaddingAwareAlignments Deep Dive

**What it does:**

Enables full-width blocks to properly extend to viewport edges when root-level padding is applied.

**When required:**

Any time `styles.spacing.padding` is set on root level.

**How it works:**

Without it:
```
┌─────────────────────────────────────┐
│ [padding]                           │ ← Root padding
│   ┌─────────────────────────────┐   │
│   │ Full-width block            │   │ ← Stops at padding
│   └─────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

With it:
```
┌─────────────────────────────────────┐
│ [padding]                           │ ← Root padding
┌─────────────────────────────────────┐
│ Full-width block                    │ ← Extends to edges via negative margins
└─────────────────────────────────────┘
│                                     │
└─────────────────────────────────────┘
```

**Implementation:**

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

**Generated CSS:**

```css
.alignfull {
	margin-left: calc(-1 * var(--wp--preset--spacing--50));
	margin-right: calc(-1 * var(--wp--preset--spacing--50));
}
```

**Common mistakes:**

See "CRITICAL: Must use object notation for padding" section above.

## Common theme.json Mistakes

| Mistake | Problem | Severity | Fix |
|---------|---------|----------|-----|
| **Missing version** | Theme.json won't load | CRITICAL | Add `"version": 3` |
| **v2 without defaultFontSizes** | Duplicate font sizes in v3 | WARNING | Add `"defaultFontSizes": false` when upgrading to v3 |
| **v2 without defaultSpacingSizes** | Duplicate spacing in v3 | WARNING | Add `"defaultSpacingSizes": false` when upgrading to v3 |
| **Hardcoded hex values in styles** | Users can't customize colors | WARNING | Define in palette, reference via CSS variable |
| **Root padding without useRootPaddingAwareAlignments** | Full-width blocks broken | WARNING | Set `"useRootPaddingAwareAlignments": true` |
| **CSS shorthand padding with useRootPaddingAwareAlignments** | Feature doesn't work | WARNING | Use object notation: `{"top": "2rem", ...}` |
| **Missing $schema** | No IDE validation | INFO | Add `"$schema": "https://schemas.wp.org/trunk/theme.json"` |
| **Trailing commas in JSON** | Parse error | CRITICAL | Remove trailing commas |
| **File paths in fontFace without file:** | Fonts won't load | CRITICAL | Use `"file:./assets/fonts/font.woff2"` |
| **Template part name with .html** | Template part not found | CRITICAL | Use `"header"` not `"header.html"` |

## WordPress.org Theme Submission Requirements

**Required for WordPress.org submission:**

1. **Valid theme.json structure**
   - Must have `version` field
   - Must be valid JSON (no syntax errors)
   - No deprecated v1 properties

2. **No hardcoded colors in templates**
   - Use theme.json palette
   - Reference via CSS variables
   - Allow user customization

3. **Accessibility**
   - Color contrast meets WCAG AA
   - Enable `color.enableContrastChecker: true`

4. **Font licensing**
   - Local fonts must be GPL-compatible
   - Include font license in LICENSE.txt

5. **No external resources**
   - No Google Fonts CDN links
   - Use local font hosting via fontFace

**Common rejection reasons:**

- Hardcoded inline styles in templates
- Missing `useRootPaddingAwareAlignments` with root padding
- Non-GPL-compatible fonts
- Color contrast failures
- Invalid JSON syntax

## Cross-References

- **Template structure and hierarchy:** See template-patterns.md
- **Full Site Editing features:** See fse-guide.md
- **Classic to block theme migration:** See classic-to-block-guide.md
- **Block pattern registration in themes:** See fse-guide.md
- **Template security (classic themes):** See wp-security-review skill
