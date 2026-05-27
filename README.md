# adze-fusion

Research and architecture project for bringing the **adze** product line to Autodesk Fusion 360.

This directory is **the agent** — opening Claude Code at `C:\adze-fusion` activates a focused, initiative-driven research and product-strategy agent. The agent's operating manual is `CLAUDE.md`. Read it first.

## Status

**Stage 0** — Scaffold landed 2026-05-15. The agent is operational. Stage 1 (research dispatch) is planned but not yet executed.

| Stage | Status |
|---|---|
| 0 — Setup | ✅ landed 2026-05-15 |
| 1 — Research dispatch | ⏸ planned in `plans/stage-1-research-orchestration.md` |
| 2 — Synthesis | not started |
| 3 — Architecture decision (ADR-001) | not started |
| 4 — Narrow prototype | not started |

## Directory layout

```
adze-fusion/
├── CLAUDE.md                          # The agent — operating manual
├── README.md                          # You are here
├── .gitignore
├── vault/                             # Obsidian-first knowledge vault
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
│   ├── 00-charter.md                  # Research charter (Stage 1+)
│   ├── streams/                       # Per-stream dispatch briefs
│   └── findings/                      # Raw sub-agent outputs before vault promotion
└── plans/
    └── stage-1-research-orchestration.md  # The Stage 1 dispatch plan
```

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
