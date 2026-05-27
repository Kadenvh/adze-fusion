# Stage 1 — Research Orchestration

**Stage:** 1 (Research dispatch)
**Status:** Planned, not yet dispatched
**Prerequisite:** Stage 0 complete (CLAUDE.md, vault rules, charter all landed)
**Owner:** adze-fusion agent

This document is the dispatch plan. Stage 1 begins when this plan is invoked — the agent fires the streams below in parallel waves, drains findings into the vault, and reports when all are complete.

---

## Dispatch principle

**Parallel by default.** Within each wave, all streams fire in a single message via multiple `Agent` tool calls. Waves are serial only when a later wave depends on earlier wave findings.

Each stream is a sub-agent with:

- **Explicit brief** — specific questions, context, what's already known to avoid
- **Success criteria** — what makes the findings vault-worthy
- **Output format** — single markdown doc under 600 words with: First principles · Findings · Sources · Decision implications · Open questions
- **Source rigor instruction** — every claim has a URL or `[empirical]` / `[unverified]` marker

---

## Waves

### Wave A — Platform Reality (5 streams, parallel)

Foundation. Until we understand Fusion's actual runtime, everything downstream is speculation.

| ID | Subject | subagent_type | Output target |
|---|---|---|---|
| A1 | Fusion 360 Python add-in API: runtime, lifecycle, install path, packaging | `frontier-research` | `findings/A1-fusion-python-runtime.md` → vault: `01-platforms/fusion-360.md`, `03-apis/fusion-python-api.md` |
| A2 | Fusion threading model: UI thread, async, document state, callbacks | `general-purpose` | `findings/A2-fusion-threading.md` → vault: extend `03-apis/fusion-python-api.md` or new threading entry |
| A3 | Fusion document model: timeline, workspaces (Design/Render/Manufacture/Simulation), object graph | `general-purpose` | `findings/A3-fusion-doc-model.md` → vault: `01-concepts/timeline-vs-feature-tree.md`, related platform entries |
| A4 | Fusion mesh handling: native mesh workspace, BREP/T-Spline/mesh conversion, T-Splines as the parametric path | `general-purpose` | `findings/A4-fusion-mesh.md` → vault: `01-concepts/mesh-in-fusion.md` |
| A5 | Cross-platform reality: Mac vs Windows API differences, sandboxing, filesystem access, local model integration on Mac | `frontier-research` | `findings/A5-fusion-cross-platform.md` → vault: `02-platforms/fusion-360-mac.md` if material differences |

**Success criteria (Wave A as a whole):**
- We can describe Fusion's add-in architecture in 3 sentences without hedging
- We know the equivalent of adze-cad's `IUiThreadInvoker` pattern in Fusion
- We know if/how local model integration works on Mac

---

### Wave B — AI / MCP / Connector Ecosystem (4 streams, parallel)

| ID | Subject | subagent_type | Output target |
|---|---|---|---|
| B1 | Autodesk Assistant: actual capabilities (vs marketing), what it can/can't do today, roadmap signals | `frontier-research` | `findings/B1-autodesk-assistant.md` → vault: `04-tools/autodesk-assistant.md` |
| B2 | MCP for Fusion: `@sockcymbal/autodesk-fusion-mcp-python` analysis (architecture, surface, status), plus any other Fusion MCP servers | `general-purpose` | `findings/B2-fusion-mcp-servers.md` → vault: per-server entries in `05-mcp-servers/` |
| B3 | Backflip and mesh-to-CAD AI: what it does, how it integrates, what they exposed of their architecture | `general-purpose` | `findings/B3-backflip.md` → vault: `04-tools/backflip.md` |
| B4 | Other AI/agentic plugins on Fusion 360 App Store: catalog with feature, pricing, status | `frontier-research` | `findings/B4-fusion-ai-plugins.md` → vault: per-plugin entries in `04-tools/` |

**Success criteria (Wave B):**
- We have a clear map of every AI/agentic tool currently on Fusion, with sources
- We know whether MCP-first architecture is supported by real working examples
- We can identify the differentiated value prop for adze on Fusion

---

### Wave C — Community + Distribution (3 streams, parallel)

| ID | Subject | subagent_type | Output target |
|---|---|---|---|
| C1 | Autodesk App Store mechanics: listing requirements, signing, revenue split, review cycle, free-vs-paid dynamics | `frontier-research` | `findings/C1-autodesk-app-store.md` → vault: `04-tools/autodesk-app-store.md` |
| C2 | Fusion community: Autodesk Forums, Reddit r/Fusion360, YouTube creators, Discord, demographic breakdown | `general-purpose` | `findings/C2-fusion-community.md` → vault: per-community entries in `06-communities/` |
| C3 | Training, resellers, partner programs for Fusion: equivalents to SOLIDWORKS' Hawkridge / Markforged / SolidProfessor — **use `ava-docs` MCP** (was rejected on adze-cad, now unblocked) | `general-purpose` with ava-docs | `findings/C3-fusion-training-partners.md` → vault: per-org entries in `06-communities/` |

**Success criteria (Wave C):**
- We know how an adze Fusion add-in would actually reach users
- We know what monetization paths are realistic (free, paid, freemium, subscription)
- We've drained the long-deferred `nqgeaw7ivx` open handoff

---

### Wave D — Portable Patterns from adze-cad (3 streams, SERIAL after A/B/C started)

Wave D depends partially on A/B/C findings — patterns we want to port should match what we now understand about Fusion's reality.

| ID | Subject | subagent_type | Output target |
|---|---|---|---|
| D1 | Crawl `C:\adze-cad` for portable architectural patterns: agentic loop, write safety lifecycle, recipe capture, trust tiers, error tiers, rate limiting. Map each to "fits Fusion / needs rewrite / doesn't apply" | `Explore` | `findings/D1-adze-cad-patterns.md` → vault: per-pattern entries in `07-patterns/` |
| D2 | Crawl `C:\adze-cad\plans\` for portable decisions and design docs. Specifically: ADR-001 (native UI authority), Decision #21-#30, the MCP design, the opt-in telemetry brief | `Explore` | `findings/D2-adze-cad-decisions.md` → vault: under `07-patterns/` and reference in `09-sources/` |
| D3 | Multi-CAD architecture synthesis: based on A/B/C/D1/D2, propose shape — shared core vs split, where the broker lives, where recipes/memory live, how MCP fits | `general-purpose` (me synthesizing, agent-prepared) | `findings/D3-multi-cad-architecture-sketch.md` → vault: `07-patterns/multi-cad-architecture-shape.md` (draft for ADR-001) |

**Success criteria (Wave D):**
- Every load-bearing adze-cad pattern has a vault entry with a Fusion-port verdict
- D3 produces a coherent multi-CAD shape that ADR-001 can be written against

---

## Triage and promotion flow

After each wave returns:

1. Read every `findings/<id>.md` file
2. For each finding, decide:
   - **Promote** — atomize into vault entries (one concept per file), with full templates, source rigor, cross-links
   - **Defer** — finding is real but not yet vault-ready (more research needed) → keep in inbox with a follow-up question
   - **Reject** — finding didn't meet success criteria → record reason in `findings/<id>.md` and re-dispatch with refined brief
3. After promotion, update `findings/<id>.md` with a header: `Promoted to vault: [[entry-name]] [[another-entry]]`
4. Drain `vault/inbox/` before moving to the next wave

---

## Anti-patterns to avoid in Stage 1

- ❌ Dumping raw agent output into a vault category folder — findings get atomized
- ❌ Promoting findings with `[unverified]` claims unless explicitly tracked
- ❌ Skipping cross-links on new vault entries
- ❌ Letting waves go serial when they could parallelize
- ❌ Starting Wave D before Wave A/B/C have at least started returning
- ❌ Letting `vault/inbox/` grow across waves

---

## What Stage 1 does NOT do

- Build prototype code (Stage 4)
- Make architecture decisions (Stage 3)
- Write the synthesis (Stage 2)
- Cite or copy adze-cad code (extract patterns only via D1/D2)

---

## Closing Stage 1

Stage 1 closes when:
- All findings/ files exist and have a `Promoted to vault: …` header
- All vault entries pass the 4-point quality bar
- `vault/inbox/` is empty
- A short `STAGE-1-COMPLETE.md` is written summarizing: what we learned, what surprised us, what we now know is wrong, where the gaps remain

Then the agent surfaces "Stage 1 complete, ready to begin Stage 2 synthesis" to the user.
