---
title: Sub-agent context isolation
category: 01-concepts
tags: [concept, agents, orchestration, context-engineering]
status: promoted
confidence: high
---

# Sub-agent context isolation

A sub-agent dispatch is a separate LLM inference with its own context window, its own tool budget, and its own opaque reasoning trace — the orchestrator pays the brief in and receives only a distilled summary out, never the raw exploration.

## First principles

Sub-agents are **not** primarily a speed multiplier. They are a context-budget multiplier. Anthropic's multi-agent research post [[09-sources/anthropic-multi-agent-research-system]] frames it bluntly: "Each subagent might explore extensively, using tens of thousands of tokens or more, but returns only a condensed, distilled summary of its work (often 1,000-2,000 tokens)." The speed gain (Anthropic reports up to 90% wall-clock reduction with 3-5 parallel workers) is real, but the strategic gain is keeping the orchestrator's window clean enough to synthesize across many findings.

## Surface

What a sub-agent call costs and produces:

| Dimension | Reality |
|---|---|
| Token cost | ~15× a single chat for equivalent work (Anthropic confirmed) |
| Time savings | Up to ~90% wall-clock for complex multi-source queries |
| Failure mode | Most failures are **invocation failures** (bad brief), not execution failures |
| Parallel sweet spot | 3-5 workers per wave; ≥10 is documented anti-pattern |
| Brief minimum | Role + objective + numbered questions + source rules + output spec |

The four invariants of a workable brief per Anthropic: **objective, output format, guidance on tools and sources, clear task boundaries.** Omitting any one produces documented failure modes — identical searches across workers, scope drift, "scouring the web endlessly for nonexistent sources."

Context isolation makes one decision sharp: **workers must write to files and return short summaries** when more than ~3 dispatch in parallel. 15 workers × 1000-word inline returns = 20K orchestrator tokens just on agent output; 15 × 100-word summaries = 1.5K. The orchestrator pulls full findings into context only when synthesizing.

## Implications for adze

- **Stage 1 dispatch is 3 waves × 5 streams** per [[../../plans/stage-1-research-orchestration]], not one fan-out of 15. Wave A breadth, Wave B depth, Wave C synthesis-only. See [[../07-patterns/3-5-parallel-wave]].
- **Every research worker writes to `research/findings/<slug>.md`** and returns ~100 words. Non-negotiable for context survival across waves.
- **Source-carve workers explicitly** (one to official docs, one to community, one to GitHub) so they don't run duplicate searches.
- The orchestrator (this agent) **must still synthesize** — sub-agents are a context tool, not a delegation of understanding.

## Open questions

- Optimal model split for orchestrator vs workers in Claude Code's harness (Opus lead + Sonnet workers is Anthropic's pattern; whether `subagent_type` exposes the choice is `[unverified]`).
- Whether a "reviewer" sub-agent that audits each finding before synthesis is worth the ceremony at adze-fusion's scale.

## Sources

- [Anthropic — How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — fetched 2026-05-15
  > "Each subagent needs an objective, an output format, guidance on the tools and sources to use, and clear task boundaries."
- [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — fetched 2026-05-15

## Related

- [[../07-patterns/3-5-parallel-wave]]
- [[../07-patterns/karpathy-llm-wiki-pattern]]
- [[../09-sources/anthropic-multi-agent-research-system]]
