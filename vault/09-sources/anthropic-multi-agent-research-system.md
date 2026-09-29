---
title: How we built our multi-agent research system — Anthropic Engineering
category: 09-sources
tags: [source, anthropic, agents, orchestration, engineering]
url: https://www.anthropic.com/engineering/multi-agent-research-system
fetched: 2026-05-15
author: Anthropic Engineering
type: official-doc
status: promoted
confidence: high
---

# How we built our multi-agent research system — Anthropic Engineering

The primary source for sub-agent dispatch discipline. Establishes the empirical "3-5 parallel workers per wave" sweet spot, the 90% time savings figure, the 15× token cost figure, the four-element brief contract, and the documented anti-patterns (50-subagent spawn, scope drift, endless search).

## Why this source is in the vault

Underpins every claim about sub-agent orchestration adze-fusion makes. Other entries that cite this:

- [[../01-concepts/sub-agent-context-isolation]] — first-principles framing
- [[../07-patterns/3-5-parallel-wave]] — dispatch shape
- The Stage 1 dispatch plan ([[../../plans/stage-1-research-orchestration]]) is built on this source's guidance.

## Key extracts

> "Each subagent might explore extensively, using tens of thousands of tokens or more, but returns only a condensed, distilled summary of its work (often 1,000-2,000 tokens)."

> "Each subagent needs an objective, an output format, guidance on the tools and sources to use, and clear task boundaries."

> 3-5 parallel workers per wave is the empirical sweet spot. ~15× tokens vs single chat. Up to ~90% wall-clock time savings on complex queries.

> Documented anti-pattern: "early agents would spawn 50 subagents for simple queries." Sub-agents have fixed overhead; use them when context isolation pays for itself.

## Reliability notes

- **Author:** Anthropic Engineering team.
- **Type:** First-party engineering blog.
- **Confidence:** High. P0 explicitly verified all major quoted numbers against the source.
- **Caveat:** P4's "1000-1800 word body band" heuristic and "~300 tokens of repeated brief boilerplate" are estimates, not quotes from this source — P0 flagged both as `[unverified]` heuristics rather than Anthropic-quoted facts.

## Related

- [[../01-concepts/sub-agent-context-isolation]]
- [[../07-patterns/3-5-parallel-wave]]
- [[karpathy-llm-wiki-gist]]
