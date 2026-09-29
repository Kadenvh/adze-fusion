# Index

Every promoted vault entry, one line per entry, organized by category. Updated on every **ingest** operation per the Karpathy LLM-Wiki pattern.

This file is **manually maintained by the agent** — not auto-generated. The discipline is what makes the index useful: every promotion adds a line, every removal deletes one. If this file drifts from reality, the vault is rotting.

---

## 01-concepts/

- [[01-concepts/mcp-protocol]] — Open transport + tool-description protocol; the canonical AI-CAD integration boundary.
- [[01-concepts/sub-agent-context-isolation]] — Sub-agent dispatch as a context-budget multiplier, not a speed multiplier.

## 02-platforms/

- [[02-platforms/fusion-360]] — Autodesk Fusion 360 — cross-platform parametric modeler, Python 3.7 in-process add-in, cloud APS sibling.

## 03-apis/

- [[03-apis/fusion-adsk-api]] — Fusion's native `adsk.*` Python API — single-threaded, CustomEvent-marshaled, parametric history first-class.

## 04-tools/

- [[04-tools/anthropic-fusion-connector]] — Anthropic's official Claude → Fusion connector (launched 2026-04-28; MCP underneath).
- [[04-tools/autodesk-assistant]] — Autodesk's in-product agentic AI — writes-and-executes Python via Script Execute; absorbing third-party MCPs.
- [[04-tools/project-salvador]] — Discontinued GPT-4 + DALL-E + Vectorizer Fusion plugin — the cautionary precedent for LLM-wrapper plugins.
- [[04-tools/cyanheads-obsidian-mcp]] — Apache-2.0 MCP server exposing surgical Obsidian-vault operations as JSON-RPC tools.

## 05-mcp-servers/

- [[05-mcp-servers/autodesk-fusion-mcp]] — Autodesk's first-party Fusion MCP server (tech preview, 2026); the substrate of the Anthropic connector.
- [[05-mcp-servers/faust-machines-fusion360-mcp]] — Largest community Fusion MCP — 84 tools, MIT, cross-platform, the canonical reference impl.
- [[05-mcp-servers/sockcymbal-fusion-mcp]] — YC Hackathon 3-tier hybrid (local Fusion + APS OAuth) — interesting for hybrid architecture study.
- [[05-mcp-servers/joe-spencer-fusion-mcp]] — Embedded-in-Fusion-Python MCP server (no sidecar). GPL-3.0; tightly constrained but architecturally clean.

## 06-communities/

- [[06-communities/autodesk-design-and-make-marketplace]] — Autodesk's distribution + ISO 42001-aligned certification surface for third-party MCPs.

## 07-patterns/

- [[07-patterns/in-process-addin-localhost-mcp]] — Dominant Fusion AI-integration pattern: Python add-in + localhost socket + MCP sidecar + CustomEvent.
- [[07-patterns/3-5-parallel-wave]] — Anthropic-documented sub-agent dispatch sweet spot; the shape of Stage 1.
- [[07-patterns/karpathy-llm-wiki-pattern]] — Three-layer raw/vault/schema; three operations ingest/query/lint. The vault's operational substrate.

## 08-decisions/

*(empty — ADR-001 will land in Stage 3)*

## 09-sources/

- [[09-sources/karpathy-llm-wiki-gist]] — Karpathy's LLM-Wiki pattern (gist, 2026-04-03, 16M views). The operational substrate this vault is built on.
- [[09-sources/scrapingart-llm-wiki-stack]] — Community-built Obsidian + Claude Code reference scaffold for the Karpathy pattern.
- [[09-sources/anthropic-creative-work-announcement]] — Anthropic's 2026-04-28 launch of 9 creative-work connectors including Autodesk Fusion.
- [[09-sources/aps-fusion-claude-creative-work]] — APS companion blog clarifying that Autodesk owns the Fusion MCP, Anthropic owns the connector.
- [[09-sources/anthropic-multi-agent-research-system]] — Anthropic Engineering's primary source for sub-agent dispatch discipline.

---

## Related

- [[overview]]
- [[log]]
- [[_health]]
