# P0 — Adversarial Review of P1-P4

**Date:** 2026-05-15
**Reviewer:** adversarial sub-agent

## Verdict at a glance

| File | Grade | Most dangerous claim | Confidence the file is sound |
|---|---|---|---|
| P1 | B+ | "Anthropic Fusion connector default port 27182" — sourced only to one third-party blog, not Anthropic/Autodesk primary | Medium-high |
| P2 | B- | "29 hook events" — actual count is 29 in lifecycle table but P2's list is inferred, not enumerated | Medium |
| P3 | B+ | "Smart Composer with Max risks an account ban" — true in spirit but slightly misframed (the ban is OAuth-token, not Smart Composer-specific) | High |
| P4 | A- | "3-5 parallel workers per wave / 15× tokens / 90% time savings" — all verified to Anthropic's post verbatim | High |

## Per-file critique

### P1 — Fusion connector ecosystem

**Verified claims:**
- Anthropic launched 9 creative-work connectors on 2026-04-28 including Autodesk Fusion (Anthropic news page confirmed).
- "Built on MCP, accessible to other LLMs" — verified, though Anthropic's page says this specifically about Blender, not generically about all 9 connectors. P1 generalizes silently.
- `faust-machines/fusion360-mcp-server`: 84 tools confirmed in README; MIT, port 9876 confirmed. Stars are **32**, not 11 as the table suggests (P1 says "11 commits" which is correct — but the stars column is absent and the number 11 may be misread as stars by a casual reader).
- `sockcymbal`: MIT, 18 stars, 5 commits, YC MCP Hackathon, `generate_cube` tool — all confirmed. **However**, P1's architecture description ("Python server on :8000") is incomplete — the actual stack is LiveCube add-in on :18080 + Fusion Server on :8000 + MCP stdio server (3-tier, not 2). P1 line 32 misses port 18080.
- `Joe-Spencer/fusion-mcp-server`: 42 stars, 7 commits, GPL-3.0, 3 tools — all confirmed.
- `ndoo/fusion360-mcp-bridge`: MIT, 7 stars, 1 commit, 2 tools — all confirmed. Mac is the **primary** platform with a quickstart script; Windows is manual. P1 says "explicit Mac + Windows install" which overstates Windows parity.

**Unverified or wrong claims:**
- **"Default port 27182"** (line 24, 89, 124) — The Anthropic news page does NOT mention port 27182. The APS blog (`bringing-fusion-claude-creative-work`) does NOT mention a port. **The only source for "27182" is `knightli.com`, a single third-party hands-on blog.** It says "the default example is `27182`" — the word "example" matters. This may be one author's local configuration, not an Anthropic/Autodesk-published default. **Treat as unverified.**
- **"Anthropic authored the connector jointly with Autodesk"** (line 8) — not in any cited primary source. Autodesk shipped the Fusion MCP; Anthropic shipped the connector that uses it. "Jointly authored" is an inference, not a quoted claim.
- **"10+ independent Fusion MCP servers"** — P1 lists 10 in the table but rows 5-10 are flagged "Listed in search results" with no detail. Existence not actually verified beyond search hits — some may be forks or shells.
- **"Project Salvador is not currently being developed and is not planned to be superseded by Autodesk Assistant"** (line 72, 149) — the quoted phrasing is internally contradictory ("not planned to be superseded" reads as the opposite of the surrounding context, which says it WAS absorbed into Assistant). Likely a transcription error or a misquote; the actual blog phrasing should be re-fetched.

**Missed topics:**
- **Licensing / commercial terms** for the Anthropic Fusion connector. Is it gated to Claude Pro/Max? Free? Enterprise-only? Not addressed.
- **Data-flow / security model** of the connector. What design data egresses to Anthropic? Cited Autodesk language about "user stays in control" is mentioned in adjacent posts but not analyzed.
- **Versioning strategy** — no discussion of MCP protocol version compat (the spec has evolved 2024 → 2025 → 2026).
- **Empirical performance** — latency, reliability of the connector. No source addresses.
- **Anthropic Connectors API extensibility** — can adze-fusion ship as a "connector" via the same surface, or only via raw MCP? Not addressed.
- **Maker / hobby license compatibility** — given the existing project's open question about SOLIDWORKS Maker licenses, the parallel Fusion-personal-use restriction on commercial connectors is conspicuously absent.

### P2 — Claude Code foundation

**Verified claims:**
- Hooks system, settings.json schema, MCP scopes, skills layout — broadly correct per the live docs.
- 29 hook events claim: **the actual hooks page lists ~29 events** (verified). The body of P2 only names ~11 of them in the "Most-used" table; the "29" is a docs-citation, not enumerated in the file. Defensible but the reader cannot audit the count from P2 alone.
- `defaultMode` values — all six listed appear in the permissions docs.
- Permission specifier syntax (`Bash(...)`, `Read(/...)`, `mcp__server__tool`) — confirmed in docs.

**Unverified or wrong claims:**
- **"Imports still load fully at launch, they don't reduce context"** (line 275) — this is a strong claim about `@path/to/file` behavior. The memory docs describe imports; whether they're lazy or eager wasn't verified directly. Plausible but uncited beyond a general doc link.
- **"settings.local.json — extends, doesn't override the project allowlist"** (line 63) — merge precedence is `deny > ask > allow` per-rule, not "extend vs override". The framing in P2 is imprecise.
- **"Block-level HTML comments are stripped before injection"** (line 273) — uncited specific behavioral claim. Plausible but unverified.
- **"User-level skills override project skills if names collide"** (line 210) — uncited; doc precedence is usually the other way (project overrides user). Should be re-checked.
- **"The existing adze-cad project has 150+ allow entries"** (line 346) — empirical claim about a real file. Reviewer didn't verify; if it's wrong, the anti-pattern weakens.

**Missed topics:**
- **Cost of full `.claude/` discipline** — token overhead of skills + agents + hooks combined. P4 addresses orchestrator budget; P2 doesn't.
- **Reproducibility** of hook scripts across teammates' machines (path assumptions, shebangs).
- **Migration path** from `commands/` to `skills/` for legacy projects.
- **How `defaultMode: auto` (research preview) actually classifies** — described as "background classifier" with no further detail.

### P3 — Obsidian / Karpathy

**Verified claims:**
- Karpathy gist published 2026-04-04, three-layer architecture, "Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase," index.md / log.md — all verified verbatim.
- `cyanheads/obsidian-mcp-server`: Apache-2.0, v3.2.2, 561 stars confirmed. Tool list matches.
- Local REST API plugin v4.0.0+ requirement confirmed.
- Anthropic OAuth third-party ban is real: enforcement began **2026-01-09**, formal terms update 2026-02-19 (search verified — see The Register, Winbuzzer, Natural 20 coverage).

**Unverified or wrong claims:**
- **"As of Jan 2026 Anthropic restricted third-party OAuth — using Smart Composer with a Max subscription risks an account ban"** (line 37) — the restriction is real BUT it applies specifically to **using subscription OAuth tokens in third-party tools**. Smart Composer can be used with an API key without triggering it. P3 conflates "Smart Composer over Max OAuth" (risky) with "Smart Composer in general" (fine with API key). Misframing, not falsehood.
- **"Karpathy gist explicitly endorses splitting on demand"** (line 71) — the gist does describe periodic lint; "explicitly endorses splitting" is a paraphrase, not a quote.
- **"Software 3.0 talk, 2025-06-17"** (line 128) — date appears plausible but unverified from primary source (the ikyle.me summary is a community summary, not the talk transcript).
- **Hub threshold "25 inlinks"** and **"3-7 outlinks healthy"** — heuristics presented without sourcing. Stated as "from PKM practitioners" — vibe-based, not citable.

**Missed topics:**
- **Community adoption signal** for cyanheads vs alternatives — 561 stars is good but no comparison to MarkusPfundstein (which P3 calls "older" without numbers).
- **Quartz publishing pipeline gotchas** beyond "Dataview doesn't render" — no analysis of frontmatter strictness, theme compat, breaking changes.
- **Performance** of Dataview at 1000+ notes (known to slow down) — not mentioned.
- **Mobile Obsidian** compatibility of the recommended plugin stack — not addressed.

### P4 — Sub-agent orchestration

**Verified claims:**
- "3-5 subagents in parallel" — verbatim quote from Anthropic's multi-agent research post.
- "90% time savings for complex queries" — verified.
- "~15× more tokens than chats" — verified.
- "Spawning 50 subagents for simple queries" anti-pattern — verified.
- "Each subagent needs an objective, output format, guidance on tools and sources, and clear task boundaries" — quoted accurately from the post.

**Unverified or wrong claims:**
- **"~300 tokens of repeated brief boilerplate per dispatch"** (line 43) — empirical estimate without measurement. Plausible but uncited.
- **"Sub-agent failures aren't execution failures — they're invocation failures"** (line 156) — presented as a quote but no source URL. Could not locate exact phrasing in Anthropic posts. May be a paraphrase.
- **"1000-1800 word body is the right band"** (line 74) — heuristic without source.

**Missed topics:**
- **Cost in dollars** of a 15-worker dispatch — the 15× token multiplier is mentioned but not translated to actionable budget. For Sonnet at $3/$15 per MTok, this is a real number.
- **Failure recovery** when a worker crashes mid-dispatch — no mention.
- **Evaluation rubrics** beyond "did it answer the question" — graders, scorecards, etc.

## Cross-file issues

- **P1 and P3 both touch MCP** and are compatible — both treat it as a stable lingua franca. No contradiction.
- **P2 and P4 overlap on sub-agents**: P2 §"Sub-agents" is the lite version of P4. The overlap is well-managed — P2 explicitly defers to P4 for depth. Minor: P2 line 215 says "memory: project → .claude/agent-memory/<name>/" while P4 does not address per-agent memory at all. Not contradictory, just an asymmetry.
- **OAuth ban**: P3 references the Jan 2026 ban but doesn't note that **this is the ban that birthed `openclaw`** — which Karpathy himself has commented on (cited in P3's own source #4 about "Karpathy on OpenClaw"). P3 missed the chance to thread this through.
- **Common gap**: none of the four files addresses **what happens when Fusion 360 isn't running** for the MCP-based workflow (P1's local pattern) — P3 mentions vault-side, P1 mentions cloud Fusion Data MCP, but the "Fusion-not-running but I want to talk to Claude about my vault" case is unaddressed.

## The single most dangerous claim

**Port 27182 is the Anthropic Fusion connector's documented default.** This is load-bearing because:
1. It anchors a specific UX claim ("single-click connection, default port 27182").
2. P1 leans on it to argue Anthropic set a high UX bar (line 124).
3. It's sourced to exactly one third-party blog that says "default example is 27182" — the word "example" is doing a lot of work.

If the actual default is different or there is no published default (Autodesk's own MCP add-in may use a different port; the App Store listing wasn't fetched for port detail), every downstream UX-comparison and architecture decision built on "27182" weakens.

This is also the most easily wrong-by-typo claim in the file: 27182 ≈ 10000×e, which feels like a cute hacker port choice that one blogger picked and others copied.

## Recommended fixes before vault promotion

1. **Re-verify port 27182** by fetching the actual Autodesk MCP add-in install page or App Store listing (currently 403'd in P1's sources). If unverifiable, soften the claim to "one blog reports 27182 as the default."
2. **Re-fetch the Project Salvador quote** — the current quoted phrase reads contradictorily and is likely transcription-corrupted.
3. **P2: enumerate the 29 hook events explicitly** or cite the docs section that does, so the count is auditable.
4. **P3: rephrase the Smart Composer/Max warning** to "Smart Composer over Max OAuth risks the OAuth ban — use API key instead."
5. **P1: clarify that "MCP-based, accessible to other LLMs"** was said about Blender specifically, not generically.
6. **All files: add licensing/commercial-terms section** where applicable (P1 for Fusion connector, P3 for plugins).
7. **P1: actually visit rows 5-10 of the community MCP table** or remove them — listing repos as "Listed in search results" without confirmation is filler.

## Sources

URLs fetched and result:

- https://www.anthropic.com/news/claude-for-creative-work — 200 — confirms 9 connectors launch but "MCP-accessible to other LLMs" was said about Blender specifically.
- https://aps.autodesk.com/blog/bringing-fusion-claude-creative-work — 200 — does NOT mention port 27182.
- https://knightli.com/en/2026/05/14/claude-fusion-360-mcp-step-model-edit/ — 200 — only source confirming "27182" and explicitly calls it an "example."
- https://github.com/faust-machines/fusion360-mcp-server — 200 — confirms 84 tools, MIT, port 9876, 32 stars, 11 commits.
- https://github.com/sockcymbal/autodesk-fusion-mcp-python — 200 — confirms 18 stars, 5 commits, MIT, generate_cube tool, YC MCP Hackathon attribution; actual port stack is 18080 + 8000, not just 8000.
- https://github.com/Joe-Spencer/fusion-mcp-server — 200 — 42 stars, 7 commits, GPL-3.0, 3 tools, Windows-only paths confirmed.
- https://github.com/ndoo/fusion360-mcp-bridge — 200 — 7 stars, 1 commit, MIT; Mac is primary, Windows is manual.
- https://github.com/cyanheads/obsidian-mcp-server — 200 — v3.2.2, Apache-2.0, 561 stars, tool list matches P3.
- https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f — 200 — 2026-04-04 publish date, three-layer architecture, all quoted phrases verified.
- https://code.claude.com/docs/en/hooks — 200 — ~29 hook events confirmed (P2's claim survives despite under-enumeration).
- https://www.anthropic.com/engineering/multi-agent-research-system — 200 — "3-5 parallel," "90% time," "15× tokens," "50 subagents" anti-pattern all verified verbatim.
- WebSearch "Anthropic Claude Max OAuth third-party plugin ban January 2026" — confirms 2026-01-09 enforcement, 2026-02-19 ToS clarification. P3's underlying claim is correct; framing needs tightening.
