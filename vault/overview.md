# Overview

Tier-1 navigation hub. Start here. Backlinks (Obsidian panel) and the graph view are the real navigation; this file is the orientation.

---

## What this vault is

The adze-fusion knowledge vault. LLM-curated (Claude Code), human-read (Obsidian). Built on Karpathy's LLM-Wiki pattern adapted to a research-and-architecture project for **AI integration with Autodesk Fusion 360, in the context of a multi-CAD adze product line**.

The agent operating manual is `../CLAUDE.md`. The taxonomy is `00-meta/ONTOLOGY.md`. The quality bar is `00-meta/VAULT-RULES.md`.

## Three operations the agent performs on this vault

- **Ingest** — promote new findings into atomized entries
- **Query** — answer questions by searching/synthesizing across entries
- **Lint** — health-check, drain orphans, resolve contradictions

See `../CLAUDE.md` for definitions.

## Where things live

| Looking for | Go to |
|---|---|
| Master operating doc | `../CLAUDE.md` |
| Taxonomy + category boundaries | `00-meta/ONTOLOGY.md` |
| Quality bar + curation discipline | `00-meta/VAULT-RULES.md` |
| Entry templates | `00-meta/_templates/` |
| Vault health queries | `_health.md` |
| Index of every promoted entry | `index.md` |
| Operation log (append-only) | `log.md` |
| Current session context | `hot.md` |
| Concepts (first-principles entries) | `01-concepts/` |
| CAD platforms | `02-platforms/` |
| API surfaces | `03-apis/` |
| Tools / libraries / packages | `04-tools/` |
| MCP servers | `05-mcp-servers/` |
| Communities | `06-communities/` |
| Patterns (portable architecture) | `07-patterns/` |
| Decisions (ADRs) | `08-decisions/` |
| Sources (pinned external refs) | `09-sources/` |
| Inbox (drained every session) | `inbox/` |

## Pinned anchor sources

Two external references the vault is built against — read these first if you're new:

- [[09-sources/karpathy-llm-wiki-gist]] — Karpathy's LLM-Wiki pattern, the operational substrate
- [[09-sources/scrapingart-llm-wiki-stack]] — community build-ready scaffold for the pattern

## Decision boundary

The vault informs decisions. ADRs live in `08-decisions/` and once accepted are immutable (supersede via new ADR, don't edit). User owns architectural decisions; agent supplies the research.

## Stage tracking

See `../CLAUDE.md` § Mission for the current stage. Stage transitions are gated:

- Stage 1 → Stage 2: **10-Source Test** must pass
- Stage 2 → Stage 3: synthesis doc reads cleanly cold and user confirms
- Stage 3 → Stage 4: ADR-001 accepted

## Related

- [[index]]
- [[_health]]
- [[hot]]
- [[00-meta/ONTOLOGY]]
- [[00-meta/VAULT-RULES]]
