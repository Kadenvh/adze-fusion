---
title: Karpathy LLM-Wiki pattern
category: 07-patterns
tags: [pattern, knowledge-base, llm, curation, karpathy]
status: promoted
confidence: high
---

# Karpathy LLM-Wiki pattern

A three-layer architecture for LLM-curated knowledge bases: immutable raw sources, an LLM-curated wiki, and a master schema. Published by Andrej Karpathy as a gist on 2026-04-04; the operational substrate this vault is built on.

## First principles

The pattern's core insight is that **the LLM is good at the bookkeeping humans abandon wikis over**. Cross-references, atomization, lint passes, periodic split/merge — these are mechanical for an LLM and tedious for humans. Karpathy's framing: "Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase." The wiki is a *persistent compounding artifact*; knowledge is compiled once and kept current, not re-derived on every query. This contrasts directly with RAG-on-raw-documents, where every query re-pays the synthesis cost.

## Three layers

```
raw/      ← Immutable source documents. LLM reads, never modifies.
vault/    ← LLM-curated knowledge. The wiki.
CLAUDE.md ← Master schema (operating manual + ontology).
```

Adze-fusion implements this directly: `raw/` for source archives, `vault/` for the curated layer, `CLAUDE.md` for the schema. See [[../09-sources/karpathy-llm-wiki-gist]] and [[../09-sources/scrapingart-llm-wiki-stack]] for the published version of this pattern.

## Three operations

Per the gist:

| Operation | When | What |
|---|---|---|
| **Ingest** | New source / agent finding lands | Read, atomize into entries, update `index.md`, append to `log.md` |
| **Query** | User asks a question or agent prepares synthesis | Search index, read entries, write the answer with citations |
| **Lint** | End of session | Surface orphans, contradictions, broken links, source gaps |

These are operations on the *wiki* (the curated layer), not the raw sources. The raw layer is immutable.

## Required spine

The gist specifies these files:

- `index.md` — content catalog, one line per entry
- `log.md` — append-only chronological record, `## [YYYY-MM-DD] action | title` format
- Schema document — the operating manual (adze-fusion uses `CLAUDE.md`)

ScrapingArt's community implementation [[../09-sources/scrapingart-llm-wiki-stack]] adds: `overview.md` (tier-1 nav), `hot.md` (rolling session context), `_health.md` (Dataview health queries), `inbox/` (drained every session). Adze-fusion adopts all of these.

## Curation discipline

- **Atomicity is a direction, not an admission criterion** — write entries; split during lint, don't gate on perfect atomicity at write time.
- **Tree-shaped graph beats cluster-hub graph.** Hubs over ~15 inlinks degrade into god-nodes; split.
- **Source rigor** — every factual claim has a citation, an `[empirical]` marker, or an `[unverified]` marker. No vibes.
- **Wikilinks to non-existent entries are OK** — they mark intent, render gray in Obsidian, and become "to be written" markers.

## Implications for adze

- **The vault structure is non-negotiable** while this pattern is the chosen substrate. Categories, spine files, three operations are load-bearing — see [[../00-meta/ONTOLOGY]] and [[../00-meta/VAULT-RULES]].
- **10-Source Test gates Stage 2 transition.** ≥10 promoted entries across ≥6 of 9 categories, all clearing the 4-point quality bar — a concrete pre-synthesis discipline derived from this pattern.
- **The pattern is independent of host.** Obsidian is the read surface; Claude Code is the write surface; the markdown files are the source of truth. Any of the three can swap without rewriting the others.

## Sources

- [Karpathy LLM-Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — fetched 2026-05-15 — see [[../09-sources/karpathy-llm-wiki-gist]]
- [ScrapingArt/Karpathy-LLM-Wiki-Stack](https://github.com/ScrapingArt/Karpathy-LLM-Wiki-Stack) — fetched 2026-05-15 — see [[../09-sources/scrapingart-llm-wiki-stack]]
- [research/findings/P3-obsidian-llm-vault-curation.md](../../research/findings/P3-obsidian-llm-vault-curation.md) — internal finding

## Related

- [[../09-sources/karpathy-llm-wiki-gist]]
- [[../09-sources/scrapingart-llm-wiki-stack]]
- [[../04-tools/cyanheads-obsidian-mcp]]
- [[3-5-parallel-wave]]
- [[../01-concepts/sub-agent-context-isolation]]
