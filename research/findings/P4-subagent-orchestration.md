# P4 — Sub-agent Orchestration Best Practices

**Date:** 2026-05-15
**Researcher:** general-purpose sub-agent

## Summary

Sub-agent dispatch in Claude Code is fundamentally a context-isolation pattern: each worker gets a fresh window, returns only a condensed summary, and the orchestrator never sees the raw exploration. The orchestration disciplines that matter most are (1) writing briefs that specify objective, output format, tool/source guidance, and task boundaries, (2) parallelizing 3–5 workers per wave instead of fanning out indiscriminately, (3) requiring file-write outputs for any wave that exceeds ~3 agents so the orchestrator's context survives, (4) verifying outputs against criteria the orchestrator defines up front rather than trusting agent self-reports, and (5) iterating in narrow waves rather than re-asking broad questions. Anthropic's own multi-agent research system reports up to 90% time savings with this pattern but uses ~15× the tokens of a single chat — the discipline pays for itself only when the synthesis value is high.

## First principles

A sub-agent call is a separate LLM inference with its own context window, its own tool budget, and its own opaque reasoning trace. The orchestrator pays a fixed input cost (the brief) and a variable output cost (the returned summary). Everything the sub-agent reads, every tool result it observes, every dead-end search it runs — none of that enters the orchestrator's context. This is the whole point: "Each subagent might explore extensively, using tens of thousands of tokens or more, but returns only a condensed, distilled summary of its work (often 1,000-2,000 tokens)" (Anthropic, *Effective Context Engineering*).

This makes sub-agents a context-budget multiplier, not a parallelism speed multiplier per se. The speed gain is real (Anthropic reports up to 90% wall-clock reduction with 3–5 parallel workers) but the strategic gain is keeping the orchestrator's window clean enough to *synthesize* across many findings. For a 15-agent Stage 1 dispatch like adze-fusion's, the synthesis ceiling is set by how compactly each worker reports back, not by how thoroughly each worker searches.

## Prompt design

The Anthropic multi-agent research post is unambiguous: **"Each subagent needs an objective, an output format, guidance on the tools and sources to use, and clear task boundaries."** Briefs that omit any of these four produce the documented failure modes: workers running identical searches, scope drift into unrelated territory, or "scouring the web endlessly for nonexistent sources."

Concrete brief structure that works:

1. **Role + one-line mission** ("You are a research sub-agent. Map best practices for X.")
2. **Context block** — only what the worker cannot infer. The "smart colleague who just walked in" frame: give them the project's purpose in 2–3 sentences and the stake in the answer, nothing more. Do *not* paste the entire CLAUDE.md or the full plan; the worker doesn't need it and it pollutes their first-turn attention.
3. **The numbered question list** — explicit, non-overlapping, each phrased so a wrong answer is detectable. Vague questions ("research the ecosystem") produce vague outputs.
4. **Source rigor rules** — which sources count as primary, what to cite, when to fall back to community sources.
5. **Output spec** — exact section structure, length budget in words, whether to write a file or return text, and where (absolute path).
6. **Length budget for body vs sources** — sub-agents over-write by default. Cap the body and leave sources unconstrained.

Terse command-style briefs work for narrow lookups ("find X in Y, return Z"). Detailed context-laden briefs are required when the worker must make judgment calls (evaluating tradeoffs, picking among approaches). The wrong move is the middle: a long brief full of project lore but no operational specificity, which produces florid but un-actionable outputs.

Omit: full file contents the worker can read itself, conversation history from the orchestrator, decision rationale the worker doesn't need to act on, and stylistic instructions that aren't about output structure.

## Choosing subagent_type

The built-in types map to a cost/capability ladder:

- **`Explore`** — read-only, Haiku-default, optimized for "where is X in this codebase" lookups. Use when the answer is a file path, a symbol location, or a pattern occurrence. Cheapest and fastest. **Not for research.** It can't reason about external sources and isn't equipped for synthesis.
- **`general-purpose`** — Sonnet-class, full toolset, default for research and multi-step reasoning. **This is the right choice for adze-fusion's research waves.** It can WebFetch, WebSearch, read repos, and synthesize.
- **`Plan`** — runs in plan mode, gathers context to produce a strategy document. Use when the deliverable is an implementation plan, not a research finding. Wrong for P-series research; right for "design the Stage 2 dispatch."
- **`claude` / catch-all** — falls back to general-purpose behavior. No reason to prefer it over `general-purpose` explicitly.
- **`statusline-setup`, `claude-code-guide`** — domain-specific built-ins; ignore for research.

**Declare a custom agent in `.claude/agents/` when**: (a) you spawn the same kind of worker repeatedly with the same instructions (Anthropic's stated trigger), (b) you want to lock tool access (e.g., a research-only agent that can't write code), or (c) you want to enforce a specific output format every time without re-specifying it in every dispatch. For adze-fusion, a custom `research-worker` agent that hardcodes "WebSearch + WebFetch + Write only; output to `research/findings/{slug}.md`; 1000–1800 word body" would eliminate ~300 tokens of repeated brief boilerplate per dispatch and reduce drift.

## Parallel vs serial

Decision framework:

- **Parallel when**: (a) outputs are independent — no worker needs another's findings, (b) the orchestrator can hold all returns in context simultaneously (or workers write to files), (c) source domains don't overlap enough to cause duplicated searches.
- **Serial when**: a later question's framing depends on an earlier answer (e.g., "given the chosen framework from wave A, evaluate its testing story").
- **Parallel-after-A (the wave pattern)**: Stage 1 of adze-fusion's design. Wave A workers fan out on independent baseline questions; once they return, the orchestrator picks 2–4 follow-up threads and dispatches Wave B in parallel against the now-clarified problem space.

Anthropic's empirical sweet spot: **3–5 parallel workers per wave**. Beyond that, "coordination overhead and complexity of managing parallel git worktrees tends to outpace the gains" (community consensus) and Anthropic specifically called out their early system "spawning 50 subagents for simple queries" as a documented anti-pattern. For a 15-agent stage, that means structuring as 3–4 waves of 4–5 agents each, not one fan-out of 15.

Hard rule: never serialize work that could parallelize. Inside a single orchestrator message, multiple Task tool calls run concurrently. Use that.

## Output format design

Structures that triage well:

```
## Summary (3–5 sentences, scannable)
## Findings (bulleted, each with a source)
## Sources (URLs, full citations)
## Implications (what this means for the project)
## Open questions (what wasn't answerable)
```

The Summary is the orchestrator's first read — it must compress the body into a synthesis-ready paragraph. The Open Questions section is the dispatch trigger for Wave B: if multiple workers flag the same gap, that's the next wave's brief.

**File-write vs text-return decision**: write to a file whenever (a) the output exceeds ~500 words, (b) the orchestrator will dispatch multiple parallel workers in the same turn (15 × 1K words returned inline would consume ~20K orchestrator tokens just on agent outputs), or (c) the finding belongs in a persistent vault. Return text only for short, throwaway answers ("does X library support Y?"). For adze-fusion's Stage 1, **every research worker should write to `research/findings/{slug}.md` and return only a ~100-word summary**. This keeps the orchestrator's context lean enough to survive 4 waves.

Length budgets: 1000–1800 words body is the right band for substantive research. Below 800, the finding is usually too thin to act on. Above 2000, the worker padded.

## Evaluating outputs

Signal-vs-noise grading checklist for each return:

1. **Did it answer the numbered questions?** If question 7 is silently missing, that's drift, not synthesis.
2. **Are findings cited?** Uncited specific claims are the hallucination tell. Anthropic-domain facts especially require `anthropic.com` or `docs.anthropic.com` URLs.
3. **Are the sources real?** Spot-check 1–2 URLs per return. Fabricated source URLs are a known sub-agent failure mode.
4. **Does the Summary match the Findings?** The known pathology: "agent's summary describes what it intended to do, not what it did." If the body is thin but the summary is confident, the body is the truth.
5. **Scope drift**: did the worker answer adjacent questions instead of the asked questions? Common when briefs are too open-ended.

Re-dispatch (don't accept) when: more than 2 numbered questions are unanswered, sources are missing for load-bearing claims, or the worker reports it "couldn't find" something a 30-second search would have found. Refine the brief with the specific gap; don't just re-run the same prompt.

Accept and move on when: questions are answered, sources check out, and remaining gaps are noted in Open Questions for Wave B.

## Common failure modes

- **Over-broad briefs → generic outputs.** Mitigation: numbered explicit questions, not open prompts.
- **Context bloat in the orchestrator.** Mitigation: workers write to files; orchestrator reads only summaries.
- **Mid-research mode drift** (worker starts researching, ends up planning implementation). Mitigation: state explicitly "do not propose implementation; this is research only."
- **Premature synthesis.** Worker hand-waves a conclusion before completing the search. Mitigation: require N sources minimum for any synthesis claim.
- **The "intended vs did" gap.** Mitigation: require evidence in every section — a finding without a quote or URL is suspect.
- **Duplicate searches across parallel workers.** Anthropic's documented case. Mitigation: each brief explicitly carves the source domain (worker A: official docs only; worker B: community blogs; worker C: GitHub issues).
- **Silent failure cascade**: a worker fails, returns nothing useful, but the orchestrator integrates the empty output as if it were a finding. Mitigation: workers must explicitly say "I could not find X" rather than omitting it.

## Foreground vs background execution

Foreground (synchronous Task call) is correct for research dispatch: the orchestrator is blocked anyway because Wave B depends on Wave A. Background execution makes sense for long-running implementation work the human will check on later (e.g., "migrate 2000 files"), not for synchronous research synthesis.

For adze-fusion: foreground all the way. Parallel Task calls inside a single orchestrator turn give parallelism without async management overhead.

## Orchestrator context budget management

The 15 × 1K words = 15K words problem is real. Mitigations, in priority order:

1. **Workers write to files; return short summaries.** This is the single biggest lever. 15 × 100-word summaries = 1.5K orchestrator tokens; 15 × full reports = 20K+.
2. **Read findings on demand**, not eagerly. The orchestrator pulls findings into context only when it's ready to synthesize that domain.
3. **Run `/clear` between stages.** Stage 1's full context is not needed for Stage 2 synthesis if the findings vault is well-organized.
4. **Compaction with explicit preservation**: `/compact preserve all research/findings/ paths and open questions`.
5. **Custom statusline showing context usage** so the orchestrator knows when it's getting full.

## Iterative dispatch patterns

Wave B briefs are *narrower* than Wave A. The combinatorial explosion happens when each wave widens the question set. Discipline:

- Wave A: breadth (10–15 workers, broad questions).
- Wave B: depth on 2–4 specific gaps Wave A revealed, with sharper questions and tighter source rules.
- Wave C: synthesis-only — no new external research, just cross-referencing.

If Wave B briefs are still broad, the orchestrator skipped triage. Pause and triage before dispatching.

## Coordination across agents

Brief workers to NOT overlap by carving territory explicitly:

- **Source carving**: "Worker A: Anthropic official docs only. Worker B: community blog posts. Worker C: GitHub issues and discussions."
- **Question carving**: each worker gets a disjoint subset of the master question list.
- **Output namespace**: each worker writes to a uniquely named file. Never two workers writing the same file path.
- **Explicit non-overlap clause**: "You are one of several workers researching adjacent topics. Stick to your assigned questions; do not generalize."

Anthropic's documented failure: workers "performed the exact same searches as other agents" because their briefs were not source-carved.

## Integrating outputs into the vault

The dispatch-side handoff: each worker's brief specifies the absolute output path (`C:\adze-fusion\research\findings\P{N}-{slug}.md`) so findings land in a predictable structure. The orchestrator then:

1. Reads the Summary section of each finding.
2. Cross-references Open Questions across workers to find the Wave B questions.
3. Promotes Implications into a synthesis doc (`research/synthesis/stage-1-synthesis.md`).
4. Leaves the per-worker findings as durable references — they are the citation layer the synthesis points back to.

Vault curation (renaming, deduplication, cross-linking to Obsidian) is downstream of dispatch and belongs to P3, not P4.

## Anti-patterns

From Anthropic's official multi-agent research post and Claude Code docs:

- **Spawning many workers for simple queries.** "Early agents would spawn 50 subagents for simple queries." Sub-agents have fixed overhead; use them when context isolation pays for itself.
- **Endless search.** Workers "scouring the web endlessly for nonexistent sources." Mitigation: cap iterations, require workers to declare when a question is unanswerable.
- **Distraction via excessive updates.** "Distracting each other with excessive updates." Workers shouldn't talk to each other; they report to the orchestrator.
- **Continuing when sufficient.** "Continuing when they already had sufficient results." Workers should stop when the brief is answered, not when the token budget runs out.
- **Don't delegate understanding.** Claude Code docs explicitly warn: do not use sub-agents to avoid reasoning yourself. The orchestrator must still synthesize.
- **Vague invocation.** "Most sub-agent failures aren't execution failures — they're invocation failures." Bad brief in, bad finding out.
- **Trust-then-verify gap.** "Claude produces a plausible-looking implementation that doesn't handle edge cases." Always include verification criteria.
- **The kitchen sink session.** Mixing unrelated work in one orchestrator context. `/clear` between stages.

## Decision implications for adze-fusion

1. **Use `general-purpose` for all P-series research workers.** Not `Explore` (read-only, can't synthesize), not `Plan` (wrong deliverable).
2. **Declare a custom `research-worker` agent in `.claude/agents/`** that hardcodes tool access (`WebSearch, WebFetch, Read, Write`), output path convention, and length budget. Saves ~300 tokens per dispatch and eliminates drift.
3. **Stage 1's 15 agents = 3 waves of 5**, not one fan-out. Wave A on independent breadth questions, Wave B on depth, Wave C synthesis-only.
4. **Every research worker writes to a file and returns a ~100-word summary.** Non-negotiable for context budget survival across 4 waves.
5. **Brief structure is fixed**: Role + Context (2–3 sentences) + Numbered Questions + Source Rules + Output Spec + Length Budget. No prose preamble.
6. **Source-carve parallel workers** to prevent duplicate searches. Assign each worker a non-overlapping source domain.
7. **Verification is the orchestrator's job**: each return is graded against the numbered questions before being accepted. Re-dispatch with refined brief if more than 2 questions are unanswered.
8. **Cap workers at 5 per wave.** Anthropic's documented sweet spot; beyond that, returns diminish.
9. **No background execution for research.** Foreground parallel Task calls only.
10. **`/clear` between stages, not between waves.** Within a stage, context cohesion matters; across stages, it doesn't.

## Open questions

- Optimal model split for the orchestrator vs workers in adze-fusion's stack (Anthropic uses Opus lead + Sonnet workers — does the harness expose that choice?).
- Whether the custom `research-worker` agent should have an explicit "no implementation suggestions" clause to prevent mode drift.
- How to detect a worker that returned a confident-but-wrong summary without manually fact-checking every claim.
- Whether to introduce a "reviewer" sub-agent that audits each finding before synthesis, or whether that's premature ceremony for a research stage.

## Sources

- [Anthropic — How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Claude Code Docs — Best practices for Claude Code](https://code.claude.com/docs/en/best-practices)
- [Claude Code Docs — Create custom subagents](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Docs — Subagents in the SDK](https://platform.claude.com/docs/en/agent-sdk/subagents)
- [Anthropic — Building agents with the Claude Agent SDK](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk)
- [Anthropic — Claude Code Advanced Patterns: Subagents, MCP, and Scaling (PDF)](https://resources.anthropic.com/hubfs/Claude%20Code%20Advanced%20Patterns_%20Subagents,%20MCP,%20and%20Scaling%20to%20Real%20Codebases.pdf)
- [MindStudio — Sub-Agents in Claude Code for Context Management and Research](https://www.mindstudio.ai/blog/sub-agents-claude-code-context-management)
- [MindStudio — Inside Claude Code's Shared Task List](https://www.mindstudio.ai/blog/claude-code-agent-teams-shared-task-list)
- [claudefa.st — Sub-Agents: Parallel vs Sequential Patterns](https://claudefa.st/blog/guide/agents/sub-agent-best-practices)
- [claudefa.st — Background Agents & Parallel Tasks](https://claudefa.st/blog/guide/agents/async-workflows)
- [Tembo — Claude Code Subagents: A 2026 Practical Guide](https://www.tembo.io/blog/claude-code-subagents)
- [Builder.io — Claude Code Subagents: How to Create, Use, and Debug](https://www.builder.io/blog/claude-code-subagents)
- [GitHub — Claude Code Issue #19739 (Unified Bug Report: Systematic Failure Patterns)](https://github.com/anthropics/claude-code/issues/19739)
- [Amit Kothari — When to use Task tool vs subagents](https://amitkoth.com/claude-code-task-tool-vs-subagents/)
- [frr.dev — Subagents in Claude Code: Delegating without losing control](https://www.frr.dev/posts/subagents-claude-code-tutorial/)
- [Rick Hightower / Towards AI — Claude Code Subagents and Main-Agent Coordination](https://medium.com/@richardhightower/claude-code-subagents-and-main-agent-coordination-a-complete-guide-to-ai-agent-delegation-patterns-a4f88ae8f46c)
- [Code With Seb — Claude Code Sub-agents: The 90% Performance Gain](https://www.codewithseb.com/blog/claude-code-sub-agents-multi-agent-systems-guide)
- [Shrivu Shankar — Building Multi-Agent Systems (Part 2)](https://blog.sshh.io/p/building-multi-agent-systems-part)
- [aiwithgrant — Context Engineering for Agents (Anthropic summary)](https://www.aiwithgrant.com/guides/anthropic-context-engineering-agents)
