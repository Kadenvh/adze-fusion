# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in adze-fusion, please report it responsibly:

**Email:** kaden@vhtech.me
**Subject prefix:** `[SECURITY]`

Please include:
- Description of the vulnerability
- Steps to reproduce
- Affected scope (vault contents / configuration / dependencies / planned implementation)
- Potential impact

Do not open a public GitHub issue for security vulnerabilities. Acknowledgment target: within 72 hours. Initial triage target: within 7 days.

## Current scope

adze-fusion is currently a **research and architecture project**, not a deployed application. There is no production surface yet. Security-relevant areas at this stage:

- **Vault contents.** The Obsidian-based vault under `vault/` contains research findings, source citations, and architecture decisions. Sensitive material (proprietary research, competitive analysis) should NOT be committed; use a separate gitignored side-vault for those.
- **Sub-agent prompts.** The agent dispatches research streams that may include URLs and search queries. Prompts should not include API keys, tokens, or private credentials.
- **Source citations.** External URLs in `vault/09-sources/` and `raw/` are not vetted for malware — they are documentation references. Treat them as such.

Once adze-fusion reaches Stage 4 (narrow prototype), this policy will be updated to cover the running code surface.

## Out of scope (currently)

- Authentication, since there is no deployed service yet
- API key handling, since adze-fusion does not yet consume external paid APIs
- Data exfiltration via the agent — agent operates within Claude Code's standard sandbox and permission model

These will be in scope at Stage 4+.

## Secure development practices

- All commits are made on signed branches (when GPG / SSH signing is configured)
- Secret scanning is enabled on the public repository
- Branch protection on `main` will be configured before any non-research code lands
- No dependencies are installed during the research stages (Stages 0-3) — there is no Python / Node / build pipeline yet
- The `.gitignore` includes `.env`, `*.local.json`, `.claude/settings.local.json`, and other secret-prone surfaces

## Supported versions

The project is pre-1.0 and pre-implementation. Every published commit is "current." Once Stage 4 prototype lands, supported versions will be tracked here.
