---
title: 3-5 parallel workers per wave
category: 07-patterns
tags: [pattern, agents, orchestration, dispatch]
status: promoted
confidence: high
---

# 3-5 parallel workers per wave

The empirical sweet spot for sub-agent dispatch. Anthropic's own multi-agent research system documents 3-5 parallel workers per wave as the band where context-isolation gain outpaces coordination overhead, with up to 90% wall-clock savings vs serial execution.

## First principles

The choice is **not** "how many can I spawn" — token budget grows ~linearly with workers and stays well within harness limits even at 15. The choice is "how many can the orchestrator usefully integrate." Above 5, briefs start overlapping (workers run duplicate searches), source carving gets ragged (no clean disjoint partition), and synthesis quality degrades because the orchestrator can no longer hold one mental model of what each worker did. Anthropic's documented failure mode of "early agents would spawn 50 subagents for simple queries" is the canonical anti-pattern.

## Pattern

```
Wave A (breadth):  5 workers || 5 disjoint source-carved questions
                   ↓
Wave B (depth):    3-5 workers || sharper questions on Wave A gaps
                   ↓
Wave C (synth):    no new dispatch; orchestrator integrates only
```

Each wave's discipline:

| Element | Rule |
|---|---|
| Worker count | 3-5; never more |
| Output | File-write to `research/findings/<slug>.md`; return ~100-word summary |
| Brief | Role + objective + numbered questions + source rules + output spec |
| Source carving | Each worker assigned a disjoint source surface |
| Verification | Each return graded against the numbered questions before acceptance |

## Stage 1 application

Adze-fusion's Stage 1 dispatch ([[../../plans/stage-1-research-orchestration]]) is structured as **3 waves × 5 streams**, restructured from an earlier 4 waves × ~4 design that over-parallelized per P4. The 3×5 shape produces 15 total streams across the stage — enough to populate the vault categories, few enough that each wave's synthesis fits in the orchestrator's context window.

## When to break the rule

- **Trivial parallel lookups** ("does library X support Y" × 10) — these don't synthesize; sub-agents are wasted on them. Use a single tool call or a script.
- **One genuinely-broad question** that doesn't decompose into 3-5 disjoint sub-questions — split the question, don't widen the worker count.
- **A wave that needs >5 workers** because the source carving says so — pause and re-triage instead. The question is probably not actually 6+ disjoint slices.

## Implications for adze

- **Stage 1 sticks at 5 per wave.** The plan is right; do not let urgency push to 8+.
- **Wave A briefs must source-carve.** Each worker gets a non-overlapping source surface (e.g., one to Autodesk docs, one to community GitHub, one to vendor blogs) so duplicate searches don't bloat findings.
- **Verification gate before promotion.** Findings drain into `research/findings/`; only those that pass adversarial review (per [[../../research/findings/P0-adversarial-review.md]]) get atomized into vault entries.

## Sources

- [Anthropic — How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — fetched 2026-05-15
  > "3-5 parallel workers"; "90% time savings"; "~15× more tokens than chats"; "early agents would spawn 50 subagents for simple queries" — anti-pattern.
- [research/findings/P4-subagent-orchestration.md](../../research/findings/P4-subagent-orchestration.md) — internal finding (P4)

## Related

- [[../01-concepts/sub-agent-context-isolation]]
- [[../09-sources/anthropic-multi-agent-research-system]]
- [[karpathy-llm-wiki-pattern]]
