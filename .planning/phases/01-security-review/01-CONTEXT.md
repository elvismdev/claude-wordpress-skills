# Phase 1: Security Review - Context

**Gathered:** 2026-02-06
**Status:** Ready for planning

<domain>
## Phase Boundary

Deliver the `wp-security-review` skill for code-level WordPress security vulnerability detection. Covers XSS, SQL injection, CSRF/nonce verification, privilege escalation, file upload validation, and authentication patterns. Code-level only — no server hardening, WAF configuration, or infrastructure security. Targets WordPress 6.x+ with PHP and JavaScript file scanning.

</domain>

<decisions>
## Implementation Decisions

### Finding output style
- Mirror the performance skill's output format: grouped by severity (Critical/Warning/Info), with line numbers, explanation, and fix per finding
- Security-specific addition: each finding includes a **CWE reference** where applicable (e.g., CWE-79 for XSS) to give findings credibility and searchability
- Severity mapping for security: CRITICAL = exploitable without authentication or leads to data breach, WARNING = exploitable with authentication or requires specific conditions, INFO = defense-in-depth improvement
- Group findings by file (like performance skill), not by vulnerability category — developers fix file by file, not vulnerability-class by vulnerability-class
- Each finding includes a concrete ❌ BAD / ✅ GOOD code pair showing the vulnerable pattern and the fix, following WordPress PHP Coding Standards

### Detection scope & depth
- **Deep coverage (dedicated sections in SKILL.md):** XSS (output escaping), SQL injection (wpdb::prepare), CSRF/nonce verification, capability/authorization checks, file upload validation
- **Surface-level coverage (scan patterns + brief guidance):** Object injection (unserialize), path traversal, open redirect, information disclosure, XML/XXE (rare in modern WP)
- **Context-aware detection:** The skill must distinguish execution contexts to avoid false positives:
  - Admin-only code (lower severity for some patterns since authenticated access required)
  - WP-CLI / cron context (nonce verification not applicable)
  - REST API endpoints (different auth model — permission_callback vs nonce)
  - Public-facing code (highest severity for unescaped output)
- **False positive guidance:** Include a "Common Mistakes" table (like performance skill) documenting known false-positive scenarios:
  - Nonces not required in WP-CLI context
  - `wp_kses_post()` is valid output for trusted content (not a missing-escape issue)
  - Admin-only `$wpdb->query()` with hardcoded SQL (no user input = no injection risk)
  - `current_user_can()` checks inside REST permission_callback are correct (not redundant)

### Reference doc tone & structure
- Follow the performance skill pattern: reference docs are **deep-dive companions** to the SKILL.md, not standalone guides
- Each reference doc is a **cookbook** — organized by pattern with ❌ BAD / ✅ GOOD examples and brief explanations
- Structure per reference doc:
  1. Quick reference table (pattern → risk → fix) for scanning
  2. Detailed patterns with code examples
  3. Edge cases and exceptions
  4. WordPress-specific nuances (e.g., late escaping philosophy, `wp_kses` family)
- Five reference docs planned:
  - `vulnerability-patterns.md` — comprehensive vulnerability catalog across all categories
  - `escaping-guide.md` — output escaping functions, context-specific escaping, late escaping principle
  - `sanitization-guide.md` — input sanitization functions, validation vs sanitization, type-specific patterns
  - `auth-patterns.md` — capability checks, role hierarchies, meta capabilities, REST permission callbacks
  - `nonce-csrf-guide.md` — nonce lifecycle, form nonces, AJAX nonces, REST nonce header, when nonces aren't needed

### Quick scan vs full review boundary
- `/wp-sec` (quick scan): Grep-based pattern detection only — runs bash grep commands against the codebase looking for high-risk signatures (like `$wpdb->query(` without `prepare`, `echo $` without escaping functions, missing `wp_verify_nonce`, `move_uploaded_file`, `unserialize`). Reports matches with file:line and severity. No deep analysis. Takes seconds.
- `/wp-sec-review` (full review): Complete skill workflow — file-type identification, context-aware analysis, severity assessment with CWE references, fix suggestions with code examples, loads reference docs for deeper patterns. Produces structured report.
- The boundary matches the performance skill exactly: quick scan = grep patterns from "Search Patterns for Quick Detection" section, full review = complete Code Review Workflow

### Claude's Discretion
- Exact grep patterns for quick detection (will be tuned during implementation based on false-positive rates)
- Ordering of vulnerability categories within SKILL.md sections
- Depth of edge-case coverage in reference docs (enough to be useful, not encyclopedic)
- Whether to include a "Platform Context" section (like performance skill has for hosting environments) — likely yes, with notes on managed hosts that have additional security layers

</decisions>

<specifics>
## Specific Ideas

- Use the existing `wp-performance-review` SKILL.md as the structural template — same 12-section layout (frontmatter, overview, when to use, workflow, file-type checks, search patterns, platform context, quick reference, severity definitions, output format, common mistakes, deep-dive references)
- The description field in YAML frontmatter should include trigger phrases: "security review", "vulnerability", "XSS", "SQL injection", "CSRF", "nonce", "sanitization", "escaping", "capability check", "privilege escalation", "file upload security", "insecure code"
- Slash commands follow the exact same pattern: full review command delegates to skill workflow, quick scan command delegates to grep patterns section
- Security findings should feel actionable, not alarmist — every CRITICAL finding must include a concrete fix, not just "this is dangerous"

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-security-review*
*Context gathered: 2026-02-06*
