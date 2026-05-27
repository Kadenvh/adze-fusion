# Research Charter — adze on Fusion 360

**Status:** Draft (Stage 0)
**Owner:** adze-fusion agent
**Decision boundary:** Synthesis (Stage 2) ends with a recommendation; ADR-001 (Stage 3) is the actual decision, made by the user with the agent's research in hand.

---

## Question we are answering

**What does adze look like on Autodesk Fusion 360, and what does the multi-CAD product (SOLIDWORKS + Fusion at minimum) look like end-to-end?**

Sub-questions, in priority order:

1. **What is the actual API surface and runtime model on Fusion 360?** (object model, threading, lifecycle, sandboxing, distribution, cross-platform reality)
2. **What is the existing AI / connector / MCP ecosystem around Fusion 360?** (Autodesk Assistant, Backflip, the `@sockcymbal/autodesk-fusion-mcp-python`, others)
3. **What is the community and distribution reality?** (Fusion 360 user base, forums, creators, App Store mechanics, monetization, free tier dynamics)
4. **What are the architectural implications for multi-CAD adze?** (shared core vs split products, language choice, MCP-first vs chat-first, where the broker / recipes / memory live)
5. **What ports cleanly from `C:\adze-cad` and what doesn't?** (concrete map of patterns and decisions)

---

## What "done" looks like (success criteria)

Stage 1 (research) is done when:

- [ ] Each of the 4 research waves has dispatched all its streams
- [ ] All stream findings landed in `research/findings/`
- [ ] Each finding has been triaged and promoted (or rejected with a written reason) into vault entries
- [ ] `vault/inbox/` is empty
- [ ] The vault has at minimum: 1 entry per Fusion platform subsystem (Wave A), 1 entry per major AI/MCP integration found (Wave B), 1 entry per community/distribution surface (Wave C), 1 entry per portable pattern identified (Wave D)
- [ ] Every vault entry passes the 4-point quality bar in `VAULT-RULES.md` (first-principles, sourced, replicable, load-bearing)

Stage 2 (synthesis) is done when:

- [ ] `RESEARCH-SYNTHESIS.md` exists at project root
- [ ] It reads cleanly cold (no implicit context)
- [ ] It cites every decision-relevant vault entry
- [ ] It identifies what we still don't know and why
- [ ] User has read it and confirmed it's ready to inform ADR-001

Stage 3 (decision) is done when:

- [ ] `vault/08-decisions/001-adze-fusion-architecture.md` exists
- [ ] Status: Accepted
- [ ] User has greenlit the decision
- [ ] An action plan exists for Stage 4 (narrow prototype)

---

## Scope cuts (what this charter does NOT cover)

- **Other CAD platforms beyond Fusion 360.** Onshape, NX, Creo are out of scope for the current research. They may be added in a Stage 5+ ADR.
- **Detailed competitive pricing analysis.** We capture pricing tiers as data, but pricing strategy decisions are user-owned and out of scope here.
- **Brand and visual identity decisions.** Out of scope; user-owned.
- **Partner program applications.** Out of scope; user-owned.
- **Building anything beyond the Stage 4 narrow prototype.** Production work starts after Stage 4 validates the Stage 3 decision.

---

## Constraints

- **Source rigor is non-negotiable.** Every claim cites a source or is marked `[empirical]` or `[unverified]`. No vibes-based findings.
- **Time budget.** Stage 1 should land in ~1 focused session of agent-dispatched research. Stage 2 synthesis: ~1 session. Stage 3 decision: another session with user discussion. Anything taking dramatically longer means scope drift — surface it.
- **Vault is the canonical artifact.** Plans, charters, and synthesis docs are scaffolding around it. The vault is what survives and ports to future projects.
- **`C:\adze-cad` is reference-only.** We extract patterns, not code.

---

## Risks and mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Fusion API docs are out of date / incomplete | High | Medium | Cross-reference official docs with community blogs and recent GitHub repos. Mark uncertainty explicitly. |
| Autodesk's MCP support is announcement-only, not real | Medium | High (if we plan MCP-first) | Specifically test by finding working integrations. The `@sockcymbal` repo is one signal. Look for others. |
| App Store reviews / signing is significantly harder than expected | Medium | High | Distribution research stream surfaces this specifically. Treat as gating before Stage 4. |
| Cross-platform (Mac) becomes a major architectural constraint | Medium | High | Wave A specifically asks for Mac-vs-Windows differences. |
| Research surfaces "Fusion is the wrong target" | Low | Very High (project pivot) | Charter built to surface this honestly. Better to find out at Stage 2 than at Stage 4. |

---

## Out-of-charter follow-ons (track in inbox, decide later)

- Onshape research and shared-core architecture validation against a third platform
- Detailed monetization strategy
- adze-cad sunset planning (separate ADR in `C:\adze-cad`)

---

## Cross-references

- `plans/stage-1-research-orchestration.md` — concrete dispatch plan for the 15 streams
- `vault/00-meta/ONTOLOGY.md` — where findings will land
- `vault/00-meta/VAULT-RULES.md` — how findings get promoted
- `C:\adze-cad\plans\archive\2026-05-15-session-10-curation\research-solidworks-ai-ecosystem.md` — landscape brief from adze-cad with high signal for Wave B
- `C:\adze-cad\plans\design-mcp-server.md` — MCP architecture work from adze-cad, potentially Phase-1 in adze-fusion
