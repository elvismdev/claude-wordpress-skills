# Testing Patterns

**Analysis Date:** 2026-02-06

## Overview

This codebase is a Claude Code plugin containing documentation and skill definitions, not source code. **There is no automated test framework.** Testing is manual and involves:

1. **Skill testing** - Copy to Claude and validate behavior
2. **Documentation testing** - Verify code examples run correctly
3. **Content review** - Ensure accuracy of recommendations
4. **CI/CD checks** - Version synchronization validation

## Testing Approach

### Skill Testing

**Manual Testing Process** (documented in `CLAUDE.md`):

```bash
# Copy skill to Claude skills directory for testing
cp -r skills/your-skill ~/.claude/skills/

# Restart Claude Code to load changes
# Test manually by invoking the skill
```

**What to test:**
- Skill activation triggers (natural language phrases in `description` field)
- Slash command invocation (`/wp-perf-review`, `/wp-perf`)
- Workflow steps execute in documented order
- Output format matches specification in skill

**Trigger Testing:**

Skills activate via YAML `description` field containing trigger phrases. Example from `SKILL.md`:

```yaml
description: WordPress performance code review and optimization analysis. Use when reviewing WordPress PHP code for performance issues, auditing themes/plugins for scalability, optimizing WP_Query, analyzing caching strategies, checking code before launch, or detecting anti-patterns, or when user mentions "performance review", "optimization audit", "slow WordPress", "slow queries", "high-traffic", "scale WordPress", "code review", "timeout", "500 error", "out of memory", or "site won't load".
```

**Test scenarios:**
- "Review this plugin for performance issues" → Should trigger wp-performance-review
- "Audit this theme for scalability" → Should trigger wp-performance-review
- "Check my code for slow queries" → Should trigger wp-performance-review
- Verify trigger phrases in README match SKILL.md description

### Code Example Validation

**Documentation Testing:**

All PHP code examples must be valid WordPress code. Testing process:

1. **Extract example** from documentation file
2. **Verify syntax** - PHP must parse without errors
3. **Validate WordPress functions** - Functions must exist in WordPress API
4. **Check logic** - Example correctly demonstrates pattern
5. **Verify BAD/GOOD pairing** - Each anti-pattern has a solution

**Example structure in code** (from `SKILL.md` and `references/anti-patterns.md`):

```php
// ❌ BAD: Unbounded query (CRITICAL).
$query = new WP_Query(
    array(
        'post_type'      => 'post',
        'posts_per_page' => -1, // Returns ALL posts
    )
);

// ✅ GOOD: Set reasonable limit.
$query = new WP_Query(
    array(
        'post_type'      => 'post',
        'posts_per_page' => 100,
        'no_found_rows'  => true,
    )
);
```

**Validation checklist:**
- PHP syntax is valid (parentheses balanced, semicolons present)
- WordPress functions match actual API (no fictional functions)
- Parameters use correct argument names and types
- Example is complete and runnable (not pseudocode fragments)
- Comments explain the issue and solution

### Content Accuracy Testing

**Knowledge Verification:**

Before merging changes, verify:

1. **WordPress compatibility** - Recommendations work on current WordPress versions
2. **Performance impact** - Claims are substantiated (links to benchmarks when available)
3. **Best practices** - Patterns align with official WordPress coding standards
4. **Completeness** - All related anti-patterns documented together

**Reference files validation:**
- `anti-patterns.md` - 28+ patterns documented with severity
- `wp-query-guide.md` - WP_Query optimization patterns
- `caching-guide.md` - Caching implementation strategies
- `measurement-guide.md` - Performance measurement approaches

### Version Synchronization

**Configuration Files Testing:**

Before release, verify version consistency across:

```bash
# Check that versions match
grep '"version"' .claude-plugin/plugin.json
grep '"version"' .claude-plugin/marketplace.json

# Both should output same version (e.g., "1.3.1")
```

**Changelog validation:**
- Latest version in CHANGELOG.md matches plugin.json
- Changes documented in CHANGELOG are reflected in modified files
- Date format consistent (YYYY-MM-DD)

## Test Organization

### Manual Test Checklist

**Before releasing a skill update:**

1. **Syntax & Structure**
   - [ ] YAML frontmatter is valid (name, description fields)
   - [ ] Markdown headers are properly formatted
   - [ ] Code blocks have language specification (```php, ```javascript)
   - [ ] No broken links in references

2. **Content Accuracy**
   - [ ] PHP code examples follow WordPress Coding Standards
   - [ ] Each anti-pattern has a BAD example and GOOD solution
   - [ ] Function calls match WordPress API
   - [ ] Severity levels appropriate (CRITICAL > WARNING > INFO)

3. **Skill Functionality**
   - [ ] Copy skill to `~/.claude/skills/wp-performance-review/`
   - [ ] Restart Claude Code
   - [ ] Test `/wp-perf-review` command invocation
   - [ ] Test `/wp-perf` command invocation
   - [ ] Test natural language trigger phrases work
   - [ ] Output format matches specification

4. **References**
   - [ ] All referenced files exist and are accessible
   - [ ] Cross-references between files are correct
   - [ ] Skill mentions "Load `references/anti-patterns.md`" appropriately

5. **Configuration**
   - [ ] `plugin.json` version incremented
   - [ ] `marketplace.json` version matches plugin.json
   - [ ] CHANGELOG.md updated with changes
   - [ ] README.md trigger phrases match SKILL.md description

### Documentation Test Scenarios

**Performance Review Skill (`SKILL.md`):**

```
Test case: Review a theme for unbounded queries
File: fictional example with posts_per_page => -1
Expected: Skill identifies as CRITICAL and suggests fix
```

```
Test case: Check AJAX implementation
File: fictional example with setInterval + fetch
Expected: Skill identifies as CRITICAL (polling pattern) and suggests REST API
```

```
Test case: Validate cache patterns
File: fictional example with url_to_postid() call
Expected: Skill identifies as WARNING (uncached) and shows caching wrapper
```

**Quick Scan Command (`wp-perf.md`):**

```
Test case: Fast triage of large plugin
Action: /wp-perf wp-content/plugins/example
Expected: Grep patterns find critical issues quickly (no full analysis)
```

## Mocking & Fixtures

**Not applicable** - This codebase contains documentation, not executable code. There are no mocks, fixtures, or test doubles.

## Coverage

**No automated coverage metrics** - Skills are assessed qualitatively:

1. **Pattern coverage** - Are all major WordPress anti-patterns documented?
   - Documented in `SKILL.md` file-type sections
   - Full reference in `references/anti-patterns.md`

2. **Example coverage** - Does each pattern have a BAD/GOOD example?
   - Inline examples in main skill sections
   - Extended examples in reference guides

3. **Severity coverage** - Are issues properly categorized?
   - Quick lookup table shows 3 severity levels used
   - Consistency check across all documentation files

## Test Commands

**No automated test runners.** Manual verification process:

```bash
# Verify JSON syntax
cat .claude-plugin/plugin.json | python3 -m json.tool > /dev/null && echo "Valid JSON"

# Verify Markdown syntax
# (Manual review or markdown linter if installed)

# Verify version consistency
grep -h '"version"' .claude-plugin/*.json | sort | uniq -d

# Count code examples
grep -c "❌ BAD:" skills/wp-performance-review/SKILL.md
grep -c "✅ GOOD:" skills/wp-performance-review/SKILL.md

# Should have equal counts (each problem has a solution)
```

## Common Testing Patterns

**Verification of PHP code example correctness:**

1. **Extract code from documentation** (e.g., from line 160-166 of SKILL.md)
2. **Copy into local WordPress environment**
3. **Verify it produces documented behavior**
4. **Check WordPress version compatibility**
5. **Note any deprecated functions** and suggest alternatives

**Example from `SKILL.md` (lines 160-166):**

```php
// ❌ CRITICAL: Unbounded query.
'posts_per_page' => -1

// ✅ GOOD: Set reasonable limit, paginate if needed.
'posts_per_page' => 100,
'no_found_rows'  => true, // Skip count if not paginating.
```

**Tested:** Does `posts_per_page => -1` actually return all posts? YES ✅

## Test Coverage Gaps

**Known gaps (not tested by automation):**

1. **Real WordPress environment testing** - Code examples not run against live WordPress
2. **Plugin compatibility** - Patterns not tested against popular plugins (WooCommerce, etc.)
3. **Performance benchmarks** - Recommendations not benchmarked with metrics
4. **Edge cases** - Not all variations of patterns documented

**Mitigation:**
- Contributions reviewed by maintainers familiar with WordPress
- Community feedback collected through GitHub issues
- New patterns added based on reported WordPress issues

## Quality Assurance

**Pre-Release Checklist** (from `CONTRIBUTING.md`):

```markdown
## Pull Request Process

1. Ensure your code follows the standards above.
2. Update documentation if you've changed functionality.
3. Add a changelog entry in CHANGELOG.md under "Unreleased".
4. Submit the PR with a clear description of changes.
5. Respond to feedback — we may ask for adjustments.
```

**Standards verified before merge:**
- PHP code follows WordPress Coding Standards
- Severity levels consistently used (CRITICAL/WARNING/INFO)
- Comments use BAD/GOOD formatting
- All code examples are complete, not fragments
- Documentation clarity and completeness

---

*Testing analysis: 2026-02-06*
