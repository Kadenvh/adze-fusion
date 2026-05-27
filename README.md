# adze-fusion

Research and architecture project for bringing the **adze** product line to Autodesk Fusion 360.

This directory is **the agent** — opening Claude Code at `C:\adze-fusion` activates a focused, initiative-driven research and product-strategy agent. The agent's operating manual is `CLAUDE.md`. Read it first.

## Status

**Stage 0.5** — Scaffold + verification adjustments landed 2026-05-15. The agent is operational, the vault is built on Karpathy LLM-Wiki pattern, and Stage 1 dispatch is planned but not yet executed.

| Stage | Status |
|---|---|
| 0 — Setup | ✅ landed 2026-05-15 (commit `e83dcc0`) |
| 0.5 — Adjustments from verification | ✅ landed 2026-05-15 (raw/ + spine files + Karpathy-method discipline) |
| 1 — Research dispatch | ⏸ planned: 3 waves × 5 streams in `plans/stage-1-research-orchestration.md` |
| 2 — Synthesis | not started — gated by 10-Source Test |
| 3 — Architecture decision (ADR-001) | not started |
| 4 — Narrow prototype | not started |

## Directory layout

Three-layer architecture per Karpathy LLM-Wiki pattern:

```
adze-fusion/
├── CLAUDE.md                          # Layer 3 — master schema + agent operating manual
├── README.md                          # You are here
├── .gitignore
├── raw/                               # Layer 1 — IMMUTABLE source documents (agent reads, never modifies)
├── vault/                             # Layer 2 — LLM-curated wiki (Obsidian)
│   ├── overview.md                    # Tier-1 navigation hub
│   ├── index.md                       # Every promoted entry, one line each
│   ├── log.md                         # Append-only operation log
│   ├── hot.md                         # Rolling session context (~500 words)
│   ├── _health.md                     # Dataview health queries
│   ├── 00-meta/                       # Ontology, vault rules, entry templates
│   ├── 01-concepts/                   # Foundational concepts
│   ├── 02-platforms/                  # Per-CAD-platform entries
│   ├── 03-apis/                       # API surface deep dives
│   ├── 04-tools/                      # Third-party tools, libraries, plugins, repos
│   ├── 05-mcp-servers/                # MCP ecosystem
│   ├── 06-communities/                # Forums, creators, resellers, training
│   ├── 07-patterns/                   # Architectural / design patterns
│   ├── 08-decisions/                  # ADRs
│   ├── 09-sources/                    # Citation entries
│   └── inbox/                         # Fast-capture, drained every session
├── research/
│   ├── 00-charter.md                  # Research charter
│   ├── streams/                       # Per-stream dispatch briefs (Stage 1)
│   └── findings/                      # Raw sub-agent outputs before vault promotion
└── plans/
    └── stage-1-research-orchestration.md  # 3 waves × 5 streams dispatch plan
```

The three core operations (ingest, query, lint) are defined in `CLAUDE.md`.

## How to use this project

### As the agent
Read `CLAUDE.md`. Follow the operating loop at the bottom of that file.

### As a human (the user)

Open `vault/` in [Obsidian](https://obsidian.md/) to navigate the knowledge graph. The vault is plain markdown — it renders fine in any viewer, but Obsidian's graph view + backlinks panel are the real value-add.

Key files to start with:

- `CLAUDE.md` — what the agent is and how it operates
- `research/00-charter.md` — what we're researching and why
- `vault/00-meta/ONTOLOGY.md` — how the vault is organized
- `vault/00-meta/VAULT-RULES.md` — quality bar for vault entries
- `plans/stage-1-research-orchestration.md` — the next stage's plan

## Relationship to `C:\adze-cad`

`C:\adze-cad` is the SOLIDWORKS-side implementation of the same product line. Treat it as **reference-only**:

- Architectural patterns, decisions, and lessons port via the vault (`vault/07-patterns/` will catalogue them after Wave D research)
- Code does not port — the languages, runtimes, and host models are too different
- The shared product brand and the multi-CAD architecture is decided in this project's ADR-001 (Stage 3)

## License

To be decided in Stage 3. The adze-cad repo is MIT-licensed; the default expectation is that this project will follow.
