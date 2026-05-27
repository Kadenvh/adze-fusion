# Hot — Rolling Session Context

Working memory. Maximum ~500 words. Overwritten as the agent works. Read at session start, updated at session end.

**If you're picking up cold:** this file tells you what the agent was just doing. If it's stale (`Last updated` older than ~24 hours), prefer `log.md` for the audit trail.

---

**Last updated:** 2026-05-15 (Stage 0.5 — adjustments landed)

## Current focus

Pre-Stage-1 verification and adjustments. Four findings streams (P1-P4) plus adversarial review (P0) landed. ScrapingArt Karpathy-LLM-Wiki-Stack pattern discovered and integrated into the vault structure (raw/ + wiki/ + CLAUDE.md three-layer architecture; hot.md, overview.md, _health.md spine; ingest/query/lint operations).

## Open questions

- Anthropic Fusion connector "default port 27182" — single-source claim from one blog. Anthropic's announcement says nothing about ports. To be marked `[unverified]` in any vault entry derived from P1.
- Fusion connector requires a paid Fusion subscription — what does this mean for maker / community positioning?
- Can adze-fusion ship its own MCP-based connector that composes with Anthropic's? Per the announcement, MCP is treated as open. Likely yes; needs Stage 1 deep-dive.
- Multi-CAD core architecture: shared Python core + per-host adapter, or separate products with shared brand?

## Recent decisions

- Project scaffolded at `C:\adze-fusion` (commit `e83dcc0`)
- Adopt Karpathy 3-layer architecture (`raw/` + `vault/` + `CLAUDE.md`)
- Adopt 3 core operations (ingest / query / lint)
- Stage 1 dispatch reshaped: 3 waves × 5 streams (was 4 waves × ~4 — over-parallelized per P4)
- Stage 1 → Stage 2 gated by 10-Source Test

## Last operations

- 2026-05-15 — Stage 0 scaffold landed (commit `e83dcc0`)
- 2026-05-15 — Dispatched P1-P4 (parallel research streams)
- 2026-05-15 — Dispatched P0 (adversarial review)
- 2026-05-15 — Verification pass via WebFetch (Anthropic post, Karpathy gist, ScrapingArt repo)
- 2026-05-15 — Stage 0.5 adjustments landed (this update)

## Active pages

Findings (not yet promoted to vault):

- `research/findings/P0-adversarial-review.md` — graded P1 B+, P2 B-, P3 B+, P4 A-
- `research/findings/P1-fusion-connector-ecosystem.md` — Anthropic Fusion connector confirmed, port claim refuted
- `research/findings/P2-claude-code-project-foundation.md` — Claude Code best practices
- `research/findings/P3-obsidian-llm-vault-curation.md` — Karpathy + cyanheads MCP recommended
- `research/findings/P4-subagent-orchestration.md` — 3-5 parallel per wave

Sources pinned but not yet promoted:

- [[09-sources/karpathy-llm-wiki-gist]] (created Stage 0.5)
- [[09-sources/scrapingart-llm-wiki-stack]] (created Stage 0.5)

## Next concrete action

Either:
- (a) **Promote P1-P4 findings into vault entries.** Each finding gets atomized into 3-8 entries spanning concepts / tools / mcp-servers / patterns / sources. After promotion, the **10-Source Test** is checked.
- (b) **Dispatch Stage 1.** 3 waves × 5 streams per the revised `plans/stage-1-research-orchestration.md`. New findings drain into new vault entries.

User picks. Likely (a) first since promotion populates the vault from already-paid-for research; (b) extends coverage.

## Related

- [[overview]]
- [[index]]
- [[log]]
- [[../CLAUDE]]
