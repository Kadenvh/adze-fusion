# Vault Ontology

The vault is organized as a flat-ish taxonomy of **stable** top-level categories. New categories are amended decisions, not silent folder additions.

## Three-layer architecture (Karpathy LLM-Wiki pattern)

The project root has three distinct surfaces:

```
C:\adze-fusion\
├── raw/        ← Immutable source documents. Agent reads, never modifies. (Karpathy Layer 1)
├── vault/      ← LLM-curated wiki. THIS file's taxonomy applies here. (Layer 2)
└── CLAUDE.md   ← Master schema / agent operating manual. (Layer 3)
```

- **`raw/`** holds fetched articles, archived gists, source PDFs, agent-finding artifacts. The agent reads these but never edits them. New material gets dropped into `raw/` and then the **ingest** operation atomizes it into vault entries.
- **`vault/`** is what this file governs. Everything below is `vault/` taxonomy.
- **`CLAUDE.md`** is the schema. Changes to it are agent edits, committed deliberately.

This three-layer split is borrowed from [[../09-sources/scrapingart-llm-wiki-stack]] which itself implements [[../09-sources/karpathy-llm-wiki-gist]].

## Top-level categories

| # | Category | Folder | What goes here | Example entries |
|---|---|---|---|---|
| 01 | **Concepts** | `vault/01-concepts/` | Foundational concepts the rest of the vault references — first-principles explainers, not opinions | `agentic-loop.md`, `mcp-protocol.md`, `compute-tool.md`, `write-safety-lifecycle.md` |
| 02 | **Platforms** | `vault/02-platforms/` | CAD platforms and their reality (one entry per platform, plus sub-entries for specific subsystems) | `fusion-360.md`, `solidworks.md`, `onshape.md` |
| 03 | **APIs** | `vault/03-apis/` | Per-API surface deep dives — what's exposed, threading model, doc model | `fusion-adsk-core.md`, `fusion-adsk-fusion.md`, `solidworks-com.md` |
| 04 | **Tools** | `vault/04-tools/` | Third-party tools, libraries, packages, plugins, repos worth knowing about | `openmcdf.md`, `autodesk-app-store.md`, `backflip.md` |
| 05 | **MCP servers** | `vault/05-mcp-servers/` | MCP ecosystem entries (existing servers, protocol notes, integration patterns) | `autodesk-fusion-mcp-python.md`, `mcp-tool-annotations.md` |
| 06 | **Communities** | `vault/06-communities/` | Forums, creators, resellers, training organizations | `fusion-360-forums.md`, `hawkridge-systems.md`, `reddit-fusion360.md` |
| 07 | **Patterns** | `vault/07-patterns/` | Architectural / design patterns — portable across platforms | `agentic-loop.md`, `trust-tier-progression.md`, `recipe-capture.md`, `compatibility-probe.md` |
| 08 | **Decisions** | `vault/08-decisions/` | Architecture Decision Records (ADRs) — numbered, immutable once recorded | `001-adze-fusion-architecture.md` |
| 09 | **Sources** | `vault/09-sources/` | External references with full citation metadata. Other entries link here for sourcing | `autodesk-fusion-api-reference.md`, `sockcymbal-fusion-mcp-repo.md` |
| -- | **Inbox** | `vault/inbox/` | Fast-capture for unsorted research output. **Drained every session.** Never grows. | n/a |

## How to choose a category

When promoting a finding to the vault, ask in order:

1. **Is it a foundational concept that other entries will reference?** → `01-concepts/`
2. **Is it about a specific CAD platform (the host, not its API)?** → `02-platforms/`
3. **Is it about a specific API surface?** → `03-apis/`
4. **Is it about a specific tool / library / repo / plugin?** → `04-tools/`
5. **Is it about an MCP server or MCP-pattern?** → `05-mcp-servers/`
6. **Is it about a community / forum / creator / reseller?** → `06-communities/`
7. **Is it a portable architectural or design pattern?** → `07-patterns/`
8. **Is it a recorded decision?** → `08-decisions/`
9. **Is it a citation that other entries reference but isn't a concept itself?** → `09-sources/`

If none fit, the entry needs to be atomized into multiple entries that DO fit. If atomization fails too, propose an ONTOLOGY amendment via the user.

## Naming conventions

- **kebab-case filenames** — `fusion-360.md`, not `Fusion 360.md` or `fusion_360.md`. Obsidian links work either way; kebab-case is portable to URLs.
- **Singular nouns** for entries — `fusion-360.md` not `fusion-360s.md`. Plurals are for index pages only.
- **No version numbers in entry names** — versioning is recorded inside the entry. `fusion-api.md` not `fusion-api-2026.md`.
- **ADR names** — `NNN-decision-slug.md`, three-digit zero-padded, sequential.

## Cross-linking

Every entry has a `Related:` section at the bottom listing `[[entry-name]]` for relevant entries in other categories. Aim for 2–6 backlinks per entry. Too few = silo; too many = noise.

Index pages (e.g., `02-platforms/README.md`) are optional and not authoritative — the ontology + cross-links are. Don't try to maintain manual lists.

## Graph topology — tree-shaped, not cluster-shaped

Per [[../09-sources/scrapingart-llm-wiki-stack]] and reinforced by P3 findings: prefer a **tree-shaped graph** over a cluster-hub graph. A single "gravity well" hub that accumulates too many connections degrades navigability. The desired shape:

```
index → overview → cluster hub → member pages → sources/syntheses
```

**Cluster-splitting rule:** when any hub has accumulated **>15 inlinks**, split it into sub-hubs. The `_health.md` Dataview queries surface hubs over the threshold automatically; lint operation acts on them.

**Anti-pattern:** an entry with 50 inlinks is not a "successful index" — it's a hub that has degraded into a god-node. Atomize it.

## When the taxonomy is wrong

If you find yourself force-fitting entries into categories that don't really fit, **stop**. Surface the misfit to the user. Categories should change as recorded decisions in `08-decisions/`, not by silent reorganization.

The taxonomy is load-bearing. Drift here is how vaults rot.
