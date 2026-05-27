# Stage 1 — Research Orchestration

**Stage:** 1 (Research dispatch)
**Status:** Planned, not yet dispatched
**Prerequisite:** Stage 0 + Stage 0.5 complete
**Owner:** adze-fusion agent
**Revised:** 2026-05-15 (Stage 0.5) — restructured from 4×~4 to 3×5 per P4 verified guidance

This document is the dispatch plan. Stage 1 begins when this plan is invoked — the agent fires the streams below in parallel waves, drains findings into the vault, and reports when all are complete.

---

## Dispatch principles (verified)

1. **3-5 parallel workers per wave** — Anthropic's documented sweet spot per [`research/findings/P4-subagent-orchestration.md`](../research/findings/P4-subagent-orchestration.md). Not 15 at once. Context isolation is the benefit, not just speed.
2. **`subagent_type = general-purpose`** for all research streams. Declare custom `research-worker` agent in `.claude/agents/` once Stage 1 stabilizes the pattern (per P4 advice: declare custom only once patterns repeat).
3. **Each brief includes:** objective · specific questions · output format · source rules · task boundaries · success criteria · file-write target.
4. **File-write outputs:** agents write findings to `research/findings/<stream-id>.md` and return ~100-word summary in their response.
5. **Source-carve parallel workers** — agents in the same wave don't search the same surfaces. Each gets a distinct slice.
6. **Verify returns** against numbered questions before accepting / promoting.

---

## Waves

### Wave 1 — Fusion Platform Deep-Dive (5 streams, parallel)

The foundation. Until we understand Fusion's runtime + Anthropic's connector + cross-platform reality, downstream architecture is speculation.

| ID | Subject | Source-carve | Output file |
|---|---|---|---|
| W1-1 | Fusion Python add-in API: runtime, lifecycle, packaging, install path | help.autodesk.com Fusion docs, autodesk.com developer docs, official samples | `findings/W1-1-fusion-python-runtime.md` |
| W1-2 | Fusion document model + workspaces + threading | help.autodesk.com Fusion API reference, community blogs on threading | `findings/W1-2-fusion-doc-model.md` |
| W1-3 | **Anthropic Fusion connector deep-dive** — capabilities, security model, licensing (Fusion subscription requirement), extensibility | anthropic.com news, aps.autodesk.com blog, Anthropic connector docs | `findings/W1-3-anthropic-fusion-connector.md` |
| W1-4 | Autodesk App Store mechanics + monetization + listing requirements | apps.autodesk.com, Autodesk dev portal | `findings/W1-4-app-store.md` |
| W1-5 | Cross-platform reality (Mac vs Windows for connectors + add-ins) | Autodesk system requirements, Mac-specific docs, community Mac forums | `findings/W1-5-cross-platform.md` |

**Wave 1 success criteria:**
- We can describe Fusion's add-in architecture in 3 sentences without hedging.
- We have a verified answer on whether adze-fusion can ship as a connector / MCP server alongside Anthropic's official one.
- We know the licensing implications for users (Fusion subscription tier required, etc.).
- We know Mac vs Windows parity status.

---

### Wave 2 — Ecosystem + Community (5 streams, parallel, AFTER Wave 1 fully returns)

Once we know what's possible technically, we map who's doing what and how users live.

| ID | Subject | Source-carve | Output file |
|---|---|---|---|
| W2-1 | Top 3 community Fusion MCP servers — **empirical analysis** (clone + run if practical) | GitHub repos: faust-machines, sockcymbal, Joe-Spencer | `findings/W2-1-community-mcp-empirical.md` |
| W2-2 | Autodesk Assistant + Project Salvador — current capabilities, extensibility, roadmap | Autodesk official blog, GoEngineer summaries | `findings/W2-2-autodesk-assistant.md` |
| W2-3 | Backflip + competitive AI plugins on Fusion App Store (with pricing) | Backflip site, Autodesk App Store filtered for AI | `findings/W2-3-competitors.md` |
| W2-4 | Fusion community surfaces — forums, Reddit r/Fusion360, YouTube creators, Discord | Autodesk Forums, Reddit, YouTube trends | `findings/W2-4-community.md` |
| W2-5 | Training/reseller landscape via **ava-docs MCP** (Hawkridge / Markforged / SolidProfessor equivalents — finally unblocked) | ava-docs `mcp__ava-docs__*` | `findings/W2-5-training-resellers.md` |

**Wave 2 success criteria:**
- Empirical baseline (not just doc-claims) on what community MCP servers actually do.
- Clear map of who competes with adze-fusion on the App Store and at what price.
- We've drained the long-deferred `nqgeaw7ivx` open handoff from adze-cad.

---

### Wave 3 — Architecture + Patterns (5 streams, parallel, AFTER Wave 2 fully returns)

Synthesis-ready. Combines what we now know about Fusion (Wave 1) and ecosystem (Wave 2) with portable patterns from sibling projects.

| ID | Subject | Source-carve | Output file |
|---|---|---|---|
| W3-1 | Crawl `C:\adze-cad` for portable architectural patterns (agentic loop, write safety lifecycle, recipe capture, trust tiers, error tiers) — **Explore subagent** | Local `C:\adze-cad\src\`, `plans\`, brain.db | `findings/W3-1-adze-cad-patterns.md` |
| W3-2 | Multi-CAD architecture patterns from outside our domain (Cursor, Copilot, Replit, JetBrains AI Assistant — how do they handle shared-core + per-host adapter) | Engineering blogs from those tools | `findings/W3-2-multi-platform-ai-tools.md` |
| W3-3 | ScrapingArt Karpathy-LLM-Wiki-Stack — extract what we adopt vs hold back | github.com/ScrapingArt/Karpathy-LLM-Wiki-Stack repo contents | `findings/W3-3-scrapingart-extraction.md` |
| W3-4 | Anthropic Connectors API — can adze-fusion ship its own connector as a first-class Claude integration? | Anthropic docs, Anthropic connector samples | `findings/W3-4-anthropic-connectors-api.md` |
| W3-5 | Synthesis sketch — multi-CAD shape proposal (informed by W1+W2+W3-1..4) — **draft of ADR-001 foundation** | All prior findings | `findings/W3-5-multi-cad-shape.md` |

**Wave 3 success criteria:**
- Every load-bearing adze-cad pattern has a Fusion-port verdict (port / rewrite / drop).
- Concrete answer on whether adze-fusion should ship as an MCP / Anthropic connector / both / neither.
- W3-5 is a coherent multi-CAD shape that ADR-001 (Stage 3) can be written against.

---

## 10-Source Test gate

Stage 1 → Stage 2 transition requires:

- ≥ **10 promoted vault entries** (not findings — actual atomized, sourced, replicable entries in category folders)
- Coverage across **≥ 6 of 9 categories** (01-concepts, 02-platforms, 03-apis, 04-tools, 05-mcp-servers, 06-communities, 07-patterns, 08-decisions, 09-sources)
- All 10+ entries clear the 4-point quality bar (first-principles · sourced · replicable · load-bearing)
- `confidence: low` entries don't count toward the 10
- `vault/inbox/` is empty
- `vault/_health.md` health queries return clean (no orphans, no open contradictions, no broken links, no source-rigor violations)

Until the test passes, **no synthesis is written**. Stage 2 doesn't begin. The agent EITHER promotes more findings (preferred) OR dispatches additional research.

---

## Triage + promotion flow

Per Karpathy LLM-Wiki ingest operation:

1. Read every `findings/<id>.md` file after each wave returns.
2. For each finding, decide: **promote** (atomize into vault entries) / **defer** (real but not vault-ready — keep in `inbox/` with follow-up question) / **reject** (didn't meet success criteria — record reason, re-dispatch with refined brief).
3. On promote: write atomized entries using the right templates from `vault/00-meta/_templates/`. Set `confidence` field honestly. Add `## Related` cross-links. Source-rigor non-negotiable.
4. Update `vault/index.md` with new entries (one line each).
5. Append to `vault/log.md` (`## [YYYY-MM-DD] ingest | Wave X finding Y → [[entry-name]]` format).
6. Update `vault/hot.md` to reflect new state.
7. After all promotions: drain `inbox/`. Update `findings/<id>.md` header with `Promoted to vault: [[entry-1]] [[entry-2]] …`.

---

## Anti-patterns

- ❌ Dispatching all 15 streams in one go — waves are serialized for triage capacity.
- ❌ Dumping raw findings into category folders — findings → triage → atomized entries.
- ❌ Promoting findings with unresolved `[unverified]` claims.
- ❌ Skipping cross-links on new entries.
- ❌ Letting `vault/inbox/` grow across waves.
- ❌ Starting Stage 2 synthesis before 10-Source Test passes.
- ❌ Spawning two agents on the same source — source-carve.
- ❌ Treating sub-agent return summaries as canonical — read the actual files.

---

## Closing Stage 1

Stage 1 closes when:
- All 15 findings/ files exist with `Promoted to vault: …` headers
- All atomized vault entries pass the 4-point quality bar
- 10-Source Test passes (verified via `vault/_health.md`)
- `vault/inbox/` is empty
- A short `STAGE-1-COMPLETE.md` is written: what we learned, what surprised us, what we now know is wrong, where the gaps remain

Then the agent surfaces "Stage 1 complete, ready to begin Stage 2 synthesis" and pauses for user confirmation.
