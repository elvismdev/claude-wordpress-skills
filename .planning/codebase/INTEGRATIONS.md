# External Integrations

**Analysis Date:** 2026-02-06

## APIs & External Services

**Code Review Analysis:**
- WordPress Core APIs - Analyzed through skill patterns (no SDK integration)
  - Patterns analyzed: WP_Query, get_posts(), hooks, options, transients
  - Authentication: None (static analysis only)

## Data Storage

**Databases:**
- Not applicable - This is a skill/plugin repository, not a backend service
- Skills analyze target WordPress databases but do not connect to them directly
- Database analysis is pattern-based code review of target PHP code

**File Storage:**
- Local filesystem only - Skills read local WordPress code files for analysis
- No remote file storage integration

**Caching:**
- Not applicable - No runtime caching layer

## Authentication & Identity

**Auth Provider:**
- None - Plugin requires no authentication
- Claude Code handles user authentication; skills receive no credentials

## Monitoring & Observability

**Error Tracking:**
- None - Skills are stateless analysis tools

**Logs:**
- Console output only - Results printed to Claude Code CLI interface
- No persistent logging or analytics

## CI/CD & Deployment

**Hosting:**
- GitHub Repository - Distribution channel
- Claude Code Marketplace - Optional plugin distribution (registered in `marketplace.json`)

**CI Pipeline:**
- None - Repository has no CI/CD configuration
- Manual version updates and releases via git tags

**Plugin Installation Methods:**
- GitHub clone: `git clone https://github.com/elvismdev/claude-wordpress-skills.git`
- Marketplace add: `/plugin marketplace add elvismdev/claude-wordpress-skills`
- Git submodule: `git submodule add https://github.com/elvismdev/claude-wordpress-skills.git`
- Direct copy: Copy skill directory to `~/.claude/skills/`

## Environment Configuration

**Required env vars:**
- None - Plugin requires no environment variables or secrets

**Configuration approach:**
- Plugin metadata in `.claude-plugin/plugin.json`
- Skill trigger phrases in `SKILL.md` frontmatter
- Command arguments passed via CLI: `$ARGUMENTS` placeholder in command definitions

## Webhooks & Callbacks

**Incoming:**
- None - Plugin is CLI-driven, not server-based

**Outgoing:**
- None - Plugin produces no callbacks or network requests

## Integration Points with Target Systems

**WordPress Analysis Targets:**
- Skills analyze local WordPress theme/plugin files (no API calls)
- Reference guides document WordPress APIs: WP_Query, database hooks, caching APIs
- Support for platform-specific guidance: WordPress VIP, WP Engine, Pantheon patterns

**Supported File Types Analyzed:**
- `functions.php` - Plugin/theme initialization files
- `*.php` - PHP source files
- `block.json` - Gutenberg block definitions
- `*.js` - JavaScript asset files
- Template files in theme directories

---

*Integration audit: 2026-02-06*
