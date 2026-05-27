---
title: Karpathy LLM-Wiki Gist
category: 09-sources
tags: [source, anchor, karpathy, llm-wiki]
url: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
fetched: 2026-05-15
author: Andrej Karpathy
type: github-gist
status: promoted
confidence: high
---

# Karpathy LLM-Wiki Gist

The foundational pattern this vault is built on. Karpathy's published proposal for an LLM-maintained markdown wiki as a replacement for RAG-on-raw-documents.

## Why this source is in the vault

This is the **operational substrate** of the entire adze-fusion vault. The agent's three core operations (ingest / query / lint) come from this gist. The discipline of `vault/index.md` + `vault/log.md` + atomized entries with frontmatter comes from this gist. The "Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase" framing is the project's mental model in one sentence.

Promoted entries that cite this source: every architectural-pattern entry derived from the LLM-Wiki framing. Listing here as `[[]]` once they're written.

## Key extracts

> "Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase."

> "The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Humans abandon wikis because maintenance burden grows faster than value. LLMs don't get bored, don't forget to update a cross-reference."

> *Three core operations:* **Ingest** (new source → LLM reads, writes summary, updates index/entity pages, appends to log) · **Query** (user asks → LLM searches index, reads relevant pages, synthesizes) · **Lint** (periodic health check: contradictions, orphans, missing cross-references).

> *Mandatory files:* `index.md` (catalog), `log.md` (append-only, `## [YYYY-MM-DD] action | title` format), schema document (this project's `CLAUDE.md`).

## Reliability notes

- **Author:** Andrej Karpathy. Anthropic / Tesla / OpenAI alum, deeply credible in this space.
- **Reach:** 16 million views on the source tweet within weeks of publication. The pattern went viral across the LLM developer community in April 2026.
- **Community adoption:** Multiple build-ready scaffolds exist (see [[scrapingart-llm-wiki-stack]]). The pattern is not theoretical — it's being implemented in production by individuals and teams.
- **Recency:** Published 2026-04-03 (Twitter announcement) / 2026-04-04 (gist file timestamp). Recent enough to reflect post-2025 LLM-agent capabilities; long enough now that the patterns have been stress-tested by adopters.
- **Confidence:** High. Primary source, author of strong reputation, content empirically verified by fetching the gist.

## Related

- [[scrapingart-llm-wiki-stack]] — Community implementation scaffold for this pattern
- [[../../CLAUDE]] — The adze-fusion agent operating manual, modeled on this pattern
- [[../overview]]
