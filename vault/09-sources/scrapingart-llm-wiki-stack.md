---
title: ScrapingArt Karpathy-LLM-Wiki-Stack
category: 09-sources
tags: [source, anchor, karpathy, llm-wiki, claude-code, obsidian, scaffold]
url: https://github.com/ScrapingArt/Karpathy-LLM-Wiki-Stack
fetched: 2026-05-15
author: ScrapingArt
type: github-repo
license: MIT
stars: 62
status: promoted
confidence: high
---

# ScrapingArt Karpathy-LLM-Wiki-Stack

Community-built reference implementation of the Karpathy LLM-Wiki pattern for Obsidian + Claude Code. Self-described as *"a comprehensive, build-ready reference for constructing a high-performance personal knowledge system using Obsidian and Claude Code, grounded in Andrej Karpathy's LLM Wiki pattern."* The most relevant external scaffold we have for the adze-fusion project.

## Why this source is in the vault

Several of the patterns adze-fusion adopts at Stage 0.5 come from this scaffold:

- The three-layer architecture: `raw/` (immutable sources) + `wiki/` (LLM-curated) + `CLAUDE.md` (master schema)
- `hot.md` rolling session context cache
- `overview.md` tier-1 navigation hub
- `confidence: high|medium|low` frontmatter field
- `> [!contradiction]` callout convention
- Tree-shaped graph topology + cluster-splitting rule (hubs >15 members get split)
- "10-Source Test" gate — don't query for decision-making until meaningful coverage
- Recommended Claude Code skills: `kepano/obsidian-skills`, `jackal092927/obsidian-official-cli-skills`, `forrestchang/andrej-karpathy-skills`

## Key extracts

On the three-layer architecture:

> Layer 1 (`raw/`) — Immutable human-curated source documents (articles, PDFs, transcripts, data files). LLM reads only; never modifies.
>
> Layer 2 (`wiki/`) — LLM-owned and maintained. Contains source summaries, entity pages, concept pages, comparisons, syntheses, plus critical files: `index.md`, `log.md`, `hot.md`, `overview.md`.
>
> Layer 3 (`CLAUDE.md`) — Master schema read by the LLM at session start.

On graph topology:

> The document strongly advocates for tree-shaped graphs over cluster-shaped graphs. A single "gravity well" hub accumulates too many connections. Instead: `index → overview → cluster hub → member pages → sources/syntheses`.

On Claude Code skill safety:

> Without `kepano/obsidian-skills`, Claude will silently break files. Without `jackal092927/obsidian-official-cli-skills`, you'll hit 13 documented silent failures in Obsidian CLI 1.12.

On vault discipline:

> NEVER write to `raw/`. No exceptions. It is the source of truth.

## Reliability notes

- **License:** MIT (free to study + adapt).
- **Stars:** 62 — modest but legitimate. Newly published April 2026.
- **Maturity:** 3 commits total. This is a reference/blueprint, not a mature codebase. No automated hooks ship.
- **Author credibility:** Independent community author. Pattern is sound; specific recommendations should be evaluated case-by-case for our use.
- **What we adopt:** the architectural skeleton + the operational disciplines.
- **What we hold back on:** the specific recommended Claude Code skills are not yet installed in adze-fusion; we'll evaluate them in Stage 1.

## Confidence: high

The repo exists, was fetched, content verified. Multiple recommendations cross-check with [[karpathy-llm-wiki-gist]] (the primary source) and with P3 findings. Where ScrapingArt diverges from Karpathy (e.g., the specific Claude Code skill recommendations), we treat as community opinion to evaluate, not authoritative.

## Related

- [[karpathy-llm-wiki-gist]] — Primary source this scaffold implements
- [[../overview]] — Navigation hub where this scaffold's patterns are codified
- [[../00-meta/VAULT-RULES]] — Quality bar incorporates this scaffold's discipline
- [[../00-meta/ONTOLOGY]] — Stable taxonomy informed by this pattern
