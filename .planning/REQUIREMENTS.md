# Requirements: Claude WordPress Skills Build-Out

**Defined:** 2026-02-06
**Core Value:** Every WordPress developer using Claude Code gets comprehensive, expert-level code review and development guidance across all major WordPress domains.

## v1 Requirements

Requirements for building out all 5 missing skills. Each maps to roadmap phases.

### Security Review (wp-security-review)

- [x] **SEC-01**: SKILL.md with YAML frontmatter (name, description with trigger phrases, max 1024 chars)
- [x] **SEC-02**: SQL injection detection — flag missing $wpdb->prepare(), raw query interpolation
- [x] **SEC-03**: XSS prevention — detect missing esc_html(), esc_attr(), esc_url() on output
- [x] **SEC-04**: CSRF/nonce verification — check wp_verify_nonce() in forms and AJAX handlers
- [x] **SEC-05**: Input sanitization — detect missing sanitize_text_field(), sanitize_email(), etc.
- [x] **SEC-06**: Output escaping — validate esc_*() function usage at display
- [x] **SEC-07**: Capability checks — check current_user_can() before privileged operations
- [x] **SEC-08**: Authentication validation — detect missing is_user_logged_in(), role checks
- [x] **SEC-09**: REST API permission callbacks — ensure permission_callback on register_rest_route()
- [x] **SEC-10**: File upload validation — check wp_handle_upload(), MIME types, extensions
- [x] **SEC-11**: Direct file access prevention — ABSPATH checks in plugin/theme files
- [x] **SEC-12**: Privilege escalation detection — missing caps in AJAX/REST, role manipulation
- [x] **SEC-13**: Dangerous function detection — flag eval(), base64_decode(), unserialize(), exec(), shell_exec()
- [x] **SEC-14**: Nonce context validation — verify nonce action/name matches operation
- [x] **SEC-15**: WordPress-specific anti-patterns — serialize() without unserialize() protection, custom auth
- [x] **SEC-16**: AJAX security patterns — wp_ajax_ vs wp_ajax_nopriv_ hooks, nonce in POST and GET
- [x] **SEC-17**: Object injection prevention — unsafe unserialize() with user input
- [x] **SEC-18**: wp_kses() vs wp_kses_post() — correct usage for context
- [x] **SEC-19**: Security constants check — DISALLOW_FILE_EDIT, FORCE_SSL_ADMIN recommendations
- [x] **SEC-20**: Search patterns for quick detection (grep commands for fast triage)
- [x] **SEC-21**: Code examples following WordPress PHP Coding Standards (❌ BAD / ✅ GOOD pattern)
- [x] **SEC-22**: Severity levels (CRITICAL/WARNING/INFO) applied consistently
- [x] **SEC-23**: Output format matching wp-performance-review structure
- [x] **SEC-24**: Common mistakes section (false positive scenarios to avoid)
- [x] **SEC-25**: Reference doc — vulnerability patterns catalog
- [x] **SEC-26**: Reference doc — escaping and sanitization guide
- [x] **SEC-27**: Reference doc — authentication and authorization patterns
- [x] **SEC-28**: Slash command /wp-sec-review (full review)
- [x] **SEC-29**: Slash command /wp-sec (quick scan)

### Gutenberg Blocks (wp-gutenberg-blocks)

- [x] **BLK-01**: SKILL.md with YAML frontmatter (name, description with trigger phrases)
- [x] **BLK-02**: block.json validation — name, title, apiVersion, structure checks
- [x] **BLK-03**: Render patterns — dynamic vs static block guidance
- [x] **BLK-04**: InnerBlocks usage — InnerBlocks.Content in save, allowedBlocks, template
- [x] **BLK-05**: useBlockProps() hook — required in edit and save functions
- [x] **BLK-06**: Attribute schema — types, defaults, sources validation
- [x] **BLK-07**: Edit vs save function consistency
- [x] **BLK-08**: Block supports — color, spacing, typography in block.json
- [x] **BLK-09**: Block deprecation — migration patterns when save changes
- [x] **BLK-10**: Script/style enqueuing — editorScript, script, style paths in block.json
- [x] **BLK-11**: Server-side rendering — render callback or render property
- [x] **BLK-12**: Interactivity API patterns — data-wp-* directives, store, state management
- [x] **BLK-13**: Block variations vs patterns — guidance on when to use which
- [x] **BLK-14**: InnerBlocks.Content gotcha — dynamic blocks must still save InnerBlocks to DB
- [x] **BLK-15**: useInnerBlocksProps hook order — useBlockProps before useInnerBlocksProps
- [x] **BLK-16**: Block context — providesContext/usesContext for parent-child data flow
- [x] **BLK-17**: Block Bindings API — dynamic attribute connections
- [x] **BLK-18**: Block patterns registration — structure, categories, keywords
- [x] **BLK-19**: Block hooks — blockHooks property for auto-insertion
- [x] **BLK-20**: Custom block controls — InspectorControls, BlockControls integration
- [x] **BLK-21**: wp_kses_post() with InnerBlocks warning — performance anti-pattern
- [x] **BLK-22**: Search patterns for quick detection (grep commands)
- [x] **BLK-23**: Code examples (PHP + React/JSX) following WordPress coding standards
- [x] **BLK-24**: Severity levels and output format matching existing skill
- [x] **BLK-25**: Common mistakes section
- [x] **BLK-26**: Reference doc — block.json and block API guide
- [x] **BLK-27**: Reference doc — React/JSX patterns for Gutenberg
- [x] **BLK-28**: Reference doc — Interactivity API guide
- [x] **BLK-29**: Slash command /wp-block-review (full review)
- [x] **BLK-30**: Slash command /wp-block (quick scan)

### Theme Development (wp-theme-development)

- [x] **THM-01**: SKILL.md with YAML frontmatter (name, description with trigger phrases)
- [x] **THM-02**: theme.json structure validation — version, settings, styles, patterns
- [x] **THM-03**: Template hierarchy — proper file naming for block and classic themes
- [x] **THM-04**: Template parts — header.html, footer.html in block themes
- [x] **THM-05**: FSE templates — templates directory structure, block markup
- [x] **THM-06**: Conditional tags — is_front_page(), is_singular(), is_page() usage
- [x] **THM-07**: get_template_part() — classic theme organization
- [x] **THM-08**: Theme supports — add_theme_support() calls validation
- [x] **THM-09**: Navigation menus — register_nav_menus() for classic, Navigation block for FSE
- [x] **THM-10**: Global styles — color palettes, typography, spacing scales in theme.json
- [x] **THM-11**: Block patterns — patterns directory, registration
- [x] **THM-12**: Block theme vs classic theme detection — auto-detect and guide accordingly
- [x] **THM-13**: theme.json version differences — v2 vs v3 features
- [x] **THM-14**: Hardcoded styles anti-pattern — inline styles, style tags detection
- [x] **THM-15**: Block stylesheets pattern — separate CSS for conditional loading
- [x] **THM-16**: Child theme safety — parent theme function override checks
- [x] **THM-17**: Custom templates registration — block theme templates/ naming
- [x] **THM-18**: Template part areas — header, footer, sidebar areas
- [x] **THM-19**: Style variations — styles/ directory structure validation
- [x] **THM-20**: Root padding aware alignments — useRootPaddingAwareAlignments
- [x] **THM-21**: Search patterns for quick detection (grep commands)
- [x] **THM-22**: Code examples (PHP + HTML templates) following WordPress coding standards
- [x] **THM-23**: Severity levels and output format matching existing skill
- [x] **THM-24**: Common mistakes section
- [x] **THM-25**: Reference doc — theme.json guide
- [x] **THM-26**: Reference doc — template patterns (classic + FSE)
- [x] **THM-27**: Reference doc — classic-to-block migration guide
- [x] **THM-28**: Slash command /wp-theme-review (full review)
- [x] **THM-29**: Slash command /wp-theme (quick scan)

### Plugin Development (wp-plugin-development)

- [x] **PLG-01**: SKILL.md with YAML frontmatter (name, description with trigger phrases)
- [x] **PLG-02**: Plugin header validation — required fields (name, version, author, etc.)
- [x] **PLG-03**: Prefix convention — 4-5 char unique prefix on functions/classes
- [x] **PLG-04**: Activation/deactivation hooks — register_activation_hook(), register_deactivation_hook()
- [x] **PLG-05**: Uninstall cleanup — uninstall.php or register_uninstall_hook()
- [x] **PLG-06**: Custom post types — register_post_type() parameters review
- [x] **PLG-07**: Custom taxonomies — register_taxonomy() implementation
- [x] **PLG-08**: Settings API usage — register_setting(), sections, fields
- [x] **PLG-09**: Options API — get_option(), update_option(), delete_option() usage
- [x] **PLG-10**: Hooks (actions/filters) — proper usage, priority, accepted args
- [x] **PLG-11**: Internationalization — text domain, __(), _e(), esc_html__(), load_plugin_textdomain()
- [x] **PLG-12**: Plugin Check standards — WordPress.org submission compliance patterns
- [x] **PLG-13**: REST API endpoint registration — register_rest_route(), permission_callback
- [x] **PLG-14**: AJAX handler patterns — wp_ajax_ and wp_ajax_nopriv_ hooks
- [x] **PLG-15**: Admin menu registration — add_menu_page(), add_submenu_page() with capabilities
- [x] **PLG-16**: Custom database tables — $wpdb->prefix, table creation in activation
- [x] **PLG-17**: Cron job registration — wp_schedule_event(), schedule checks
- [x] **PLG-18**: Transients API — set_transient() with proper expiration
- [x] **PLG-19**: Function/class existence checks — function_exists(), class_exists()
- [x] **PLG-20**: Conditional loading — is_admin() to separate admin/public code
- [x] **PLG-21**: Search patterns for quick detection (grep commands)
- [x] **PLG-22**: Code examples following WordPress PHP Coding Standards
- [x] **PLG-23**: Severity levels and output format matching existing skill
- [x] **PLG-24**: Common mistakes section
- [x] **PLG-25**: Reference doc — plugin architecture patterns
- [x] **PLG-26**: Reference doc — hooks and API guide
- [x] **PLG-27**: Reference doc — WordPress.org submission checklist
- [x] **PLG-28**: Slash command /wp-plugin-review (full review)
- [x] **PLG-29**: Slash command /wp-plugin (quick scan)

### WooCommerce Development (wp-woocommerce-dev)

- [x] **WOO-01**: SKILL.md with YAML frontmatter (name, description with trigger phrases)
- [x] **WOO-02**: Custom product types — register_product_type(), product data stores
- [x] **WOO-03**: Payment gateway integration — WC_Payment_Gateway class, process_payment()
- [x] **WOO-04**: Shipping method development — WC_Shipping_Method class, calculate_shipping()
- [x] **WOO-05**: WooCommerce hooks — woocommerce_before_cart, woocommerce_checkout_process, etc.
- [x] **WOO-06**: Product data — custom fields via woocommerce_product_options_*, save hooks
- [x] **WOO-07**: Order handling — WC_Order methods, status transitions, CRUD operations
- [x] **WOO-08**: Cart operations — WC()->cart methods, cart item data, fees
- [x] **WOO-09**: REST API extensions — WooCommerce REST API endpoints, custom endpoints
- [x] **WOO-10**: Template overrides — woocommerce/ directory in themes, proper override patterns
- [x] **WOO-11**: Shop page customization — archive-product.php, content-product.php patterns
- [x] **WOO-12**: Cart/checkout theming — template parts, form field customization
- [x] **WOO-13**: Product display patterns — single-product.php, gallery, variations
- [x] **WOO-14**: WooCommerce blocks — product grid, cart block, checkout block integration
- [x] **WOO-15**: Cart fragments performance — wc-cart-fragments.js, disable/optimize patterns
- [x] **WOO-16**: High-order-volume patterns — HPOS (High-Performance Order Storage) compatibility
- [x] **WOO-17**: WooCommerce query optimization — WC_Product_Query, wc_get_products() patterns
- [x] **WOO-18**: Session handling — WC_Session_Handler, avoid custom sessions
- [x] **WOO-19**: Checkout security — payment data handling, PCI considerations in custom code
- [x] **WOO-20**: AJAX add-to-cart security — nonce verification, capability checks
- [x] **WOO-21**: Coupon/discount validation — custom coupon types, validation hooks
- [x] **WOO-22**: Search patterns for quick detection (grep commands)
- [x] **WOO-23**: Code examples following WordPress PHP Coding Standards
- [x] **WOO-24**: Severity levels and output format matching existing skill
- [x] **WOO-25**: Common mistakes section (WooCommerce-specific false positives)
- [x] **WOO-26**: Reference doc — WooCommerce extension development guide
- [x] **WOO-27**: Reference doc — WooCommerce template and theming guide
- [x] **WOO-28**: Reference doc — WooCommerce performance patterns
- [x] **WOO-29**: Slash command /wp-woo-review (full review)
- [x] **WOO-30**: Slash command /wp-woo (quick scan)

### Infrastructure Updates

- [x] **INF-01**: Update README.md — all 5 new skills status ✅, trigger phrases, command docs
- [x] **INF-02**: Update marketplace.json — register all new skills
- [x] **INF-03**: Update plugin.json — version bump
- [x] **INF-04**: Update CHANGELOG.md — document all new skills

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Advanced Security

- **SEC-V2-01**: Magic method exploit detection (__toString, __destruct chains)
- **SEC-V2-02**: Security header generation (CSP, X-Frame-Options for plugin admin)

### Advanced Blocks

- **BLK-V2-01**: Block Slots API patterns (when standardized)
- **BLK-V2-02**: Block library performance profiling guidance

### Multisite

- **MS-01**: Multisite-specific patterns and network admin
- **MS-02**: Network-wide plugin activation patterns

## Out of Scope

| Feature | Reason |
|---------|--------|
| Server/infrastructure hardening | Claude reviews code, not server configs |
| Multisite skill | Niche use case, defer to v2 |
| Automated testing/CI | Plugin is documentation-only, no runtime |
| WordPress 5.x backward compat | Targeting WP 6.x+ modern APIs |
| CSS preprocessor patterns | Build tooling choice, not WP-specific |
| Page builder plugin patterns | Third-party tools, too many to cover |
| TypeScript block examples | Language choice, JS examples with TS note |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| SEC-01 | Phase 1 | Complete |
| SEC-02 | Phase 1 | Complete |
| SEC-03 | Phase 1 | Complete |
| SEC-04 | Phase 1 | Complete |
| SEC-05 | Phase 1 | Complete |
| SEC-06 | Phase 1 | Complete |
| SEC-07 | Phase 1 | Complete |
| SEC-08 | Phase 1 | Complete |
| SEC-09 | Phase 1 | Complete |
| SEC-10 | Phase 1 | Complete |
| SEC-11 | Phase 1 | Complete |
| SEC-12 | Phase 1 | Complete |
| SEC-13 | Phase 1 | Complete |
| SEC-14 | Phase 1 | Complete |
| SEC-15 | Phase 1 | Complete |
| SEC-16 | Phase 1 | Complete |
| SEC-17 | Phase 1 | Complete |
| SEC-18 | Phase 1 | Complete |
| SEC-19 | Phase 1 | Complete |
| SEC-20 | Phase 1 | Complete |
| SEC-21 | Phase 1 | Complete |
| SEC-22 | Phase 1 | Complete |
| SEC-23 | Phase 1 | Complete |
| SEC-24 | Phase 1 | Complete |
| SEC-25 | Phase 1 | Complete |
| SEC-26 | Phase 1 | Complete |
| SEC-27 | Phase 1 | Complete |
| SEC-28 | Phase 1 | Complete |
| SEC-29 | Phase 1 | Complete |
| PLG-01 | Phase 2 | Complete |
| PLG-02 | Phase 2 | Complete |
| PLG-03 | Phase 2 | Complete |
| PLG-04 | Phase 2 | Complete |
| PLG-05 | Phase 2 | Complete |
| PLG-06 | Phase 2 | Complete |
| PLG-07 | Phase 2 | Complete |
| PLG-08 | Phase 2 | Complete |
| PLG-09 | Phase 2 | Complete |
| PLG-10 | Phase 2 | Complete |
| PLG-11 | Phase 2 | Complete |
| PLG-12 | Phase 2 | Complete |
| PLG-13 | Phase 2 | Complete |
| PLG-14 | Phase 2 | Complete |
| PLG-15 | Phase 2 | Complete |
| PLG-16 | Phase 2 | Complete |
| PLG-17 | Phase 2 | Complete |
| PLG-18 | Phase 2 | Complete |
| PLG-19 | Phase 2 | Complete |
| PLG-20 | Phase 2 | Complete |
| PLG-21 | Phase 2 | Complete |
| PLG-22 | Phase 2 | Complete |
| PLG-23 | Phase 2 | Complete |
| PLG-24 | Phase 2 | Complete |
| PLG-25 | Phase 2 | Complete |
| PLG-26 | Phase 2 | Complete |
| PLG-27 | Phase 2 | Complete |
| PLG-28 | Phase 2 | Complete |
| PLG-29 | Phase 2 | Complete |
| BLK-01 | Phase 3 | Complete |
| BLK-02 | Phase 3 | Complete |
| BLK-03 | Phase 3 | Complete |
| BLK-04 | Phase 3 | Complete |
| BLK-05 | Phase 3 | Complete |
| BLK-06 | Phase 3 | Complete |
| BLK-07 | Phase 3 | Complete |
| BLK-08 | Phase 3 | Complete |
| BLK-09 | Phase 3 | Complete |
| BLK-10 | Phase 3 | Complete |
| BLK-11 | Phase 3 | Complete |
| BLK-12 | Phase 3 | Complete |
| BLK-13 | Phase 3 | Complete |
| BLK-14 | Phase 3 | Complete |
| BLK-15 | Phase 3 | Complete |
| BLK-16 | Phase 3 | Complete |
| BLK-17 | Phase 3 | Complete |
| BLK-18 | Phase 3 | Complete |
| BLK-19 | Phase 3 | Complete |
| BLK-20 | Phase 3 | Complete |
| BLK-21 | Phase 3 | Complete |
| BLK-22 | Phase 3 | Complete |
| BLK-23 | Phase 3 | Complete |
| BLK-24 | Phase 3 | Complete |
| BLK-25 | Phase 3 | Complete |
| BLK-26 | Phase 3 | Complete |
| BLK-27 | Phase 3 | Complete |
| BLK-28 | Phase 3 | Complete |
| BLK-29 | Phase 3 | Complete |
| BLK-30 | Phase 3 | Complete |
| THM-01 | Phase 4 | Complete |
| THM-02 | Phase 4 | Complete |
| THM-03 | Phase 4 | Complete |
| THM-04 | Phase 4 | Complete |
| THM-05 | Phase 4 | Complete |
| THM-06 | Phase 4 | Complete |
| THM-07 | Phase 4 | Complete |
| THM-08 | Phase 4 | Complete |
| THM-09 | Phase 4 | Complete |
| THM-10 | Phase 4 | Complete |
| THM-11 | Phase 4 | Complete |
| THM-12 | Phase 4 | Complete |
| THM-13 | Phase 4 | Complete |
| THM-14 | Phase 4 | Complete |
| THM-15 | Phase 4 | Complete |
| THM-16 | Phase 4 | Complete |
| THM-17 | Phase 4 | Complete |
| THM-18 | Phase 4 | Complete |
| THM-19 | Phase 4 | Complete |
| THM-20 | Phase 4 | Complete |
| THM-21 | Phase 4 | Complete |
| THM-22 | Phase 4 | Complete |
| THM-23 | Phase 4 | Complete |
| THM-24 | Phase 4 | Complete |
| THM-25 | Phase 4 | Complete |
| THM-26 | Phase 4 | Complete |
| THM-27 | Phase 4 | Complete |
| THM-28 | Phase 4 | Complete |
| THM-29 | Phase 4 | Complete |
| WOO-01 | Phase 5 | Complete |
| WOO-02 | Phase 5 | Complete |
| WOO-03 | Phase 5 | Complete |
| WOO-04 | Phase 5 | Complete |
| WOO-05 | Phase 5 | Complete |
| WOO-06 | Phase 5 | Complete |
| WOO-07 | Phase 5 | Complete |
| WOO-08 | Phase 5 | Complete |
| WOO-09 | Phase 5 | Complete |
| WOO-10 | Phase 5 | Complete |
| WOO-11 | Phase 5 | Complete |
| WOO-12 | Phase 5 | Complete |
| WOO-13 | Phase 5 | Complete |
| WOO-14 | Phase 5 | Complete |
| WOO-15 | Phase 5 | Complete |
| WOO-16 | Phase 5 | Complete |
| WOO-17 | Phase 5 | Complete |
| WOO-18 | Phase 5 | Complete |
| WOO-19 | Phase 5 | Complete |
| WOO-20 | Phase 5 | Complete |
| WOO-21 | Phase 5 | Complete |
| WOO-22 | Phase 5 | Complete |
| WOO-23 | Phase 5 | Complete |
| WOO-24 | Phase 5 | Complete |
| WOO-25 | Phase 5 | Complete |
| WOO-26 | Phase 5 | Complete |
| WOO-27 | Phase 5 | Complete |
| WOO-28 | Phase 5 | Complete |
| WOO-29 | Phase 5 | Complete |
| WOO-30 | Phase 5 | Complete |
| INF-01 | Phase 5 | Complete |
| INF-02 | Phase 5 | Complete |
| INF-03 | Phase 5 | Complete |
| INF-04 | Phase 5 | Complete |

**Coverage:**
- v1 requirements: 150 total
- Mapped to phases: 150
- Unmapped: 0 ✓

---
*Requirements defined: 2026-02-06*
*Last updated: 2026-02-06 after Phase 5 completion — all v1 requirements complete*
