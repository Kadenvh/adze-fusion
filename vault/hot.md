# Hot — Rolling Session Context

Working memory. Maximum ~500 words. Overwritten as the agent works. Read at session start, updated at session end.

**If you're picking up cold:** this file tells you what the agent was just doing. If it's stale (`Last updated` older than ~24 hours), prefer `log.md` for the audit trail.

---

**Last updated:** 2026-05-27 (Stage 0.5 → Stage 1 transition: P1-P4 + P0 promoted, 10-Source Test passed)

## Current focus

P1-P4 findings + P0 adversarial review have been **atomized into 19 vault entries spanning 8 of 9 categories**. The 10-Source Test (≥10 entries / ≥6 categories / no `confidence: low`) has **passed**: 21 total promoted entries (19 new + the 2 anchor sources promoted in Stage 0.5). Stage 1 → Stage 2 transition is structurally unblocked, but **Stage 1 dispatch (3 waves × 5 streams per `plans/stage-1-research-orchestration.md`) has not been run** — the 10-Source Test gates Stage 2 *synthesis*, but Stage 1's actual research streams still need to fire if we want the vault to be more than the bootstrap pass.

## Open questions

- **Default port for the Anthropic Fusion connector** — `27182` is `[unverified]`; only knightli.com sources it and they call it an "example." Needs App Store install-page fetch.
- **Project Salvador discontinuation quote** — `[unverified]`; the P1 quote reads contradictorily. Needs re-fetch of `autodesk.com/products/fusion-360/blog/project-salvador-autodesk-fusion-app-store/`.
- **Marketplace openness to new entrants** — `[unverified]`; certification process exists but no confirmation that it's open to non-pre-selected partners.
- **Whether Stage 1 should fire now** that the bootstrap pass is in the vault, OR whether the user wants to review the 19 promoted entries first.
- Multi-CAD core architecture — shared Python core + per-host adapter, or separate products with shared brand? (Held for Stage 3.)

## Recent decisions

- 2026-05-27 — Path A executed: P1-P4 + P0 atomized into vault. Stage 0.5 ingest pass complete.
- 2026-05-15 — Stage 1 dispatch reshaped to 3 waves × 5 streams (over-parallelization fix per P4).
- 2026-05-15 — Stage 1 → Stage 2 gated by 10-Source Test (now passed).
- 2026-05-15 — Karpathy 3-layer architecture adopted (`raw/` + `vault/` + `CLAUDE.md`).

## Last operations

- 2026-05-27 — `ingest`: 19 vault entries written from P1-P4 + P0; index.md + log.md updated; 10-Source Test passed.
- 2026-05-15 — Stage 0.5 adjustments landed.
- 2026-05-15 — P0 adversarial review dispatched, P1-P4 parallel research streams dispatched.
- 2026-05-15 — Stage 0 scaffold landed (commit `e83dcc0`).

## Active pages

Promoted entries (21 total, 8 categories): see [[index]] for the canonical list.

Findings still in `research/findings/` (kept as citation backstop):

- `P0-adversarial-review.md`, `P1-fusion-connector-ecosystem.md`, `P2-claude-code-project-foundation.md`, `P3-obsidian-llm-vault-curation.md`, `P4-subagent-orchestration.md`

## Next concrete action

**Decision point for the user.** With the 10-Source Test passed, three paths are open:

(a) **Dispatch Stage 1 Wave 1** (5 parallel streams per `plans/stage-1-research-orchestration.md` — Fusion platform deep-dive: runtime, doc model, Anthropic connector deep-dive, App Store mechanics, cross-platform reality).
(b) **Verify the bootstrap pass** — re-fetch the three `[unverified]` items (port 27182, Salvador quote, Marketplace openness) before committing to Stage 1 briefs.
(c) **Audit the promoted entries** — read through the 19 new entries and flag any that don't clear the 4-point quality bar before they're treated as load-bearing.

User picks. Likely (b) is the smallest-step value — three short web fetches close known gaps cheaply. (a) is the obvious "go big" follow-up. (c) is the safety play.

## Related

- [[overview]]
- [[index]]
- [[log]]
- [[_health]]
- [[../CLAUDE]]
