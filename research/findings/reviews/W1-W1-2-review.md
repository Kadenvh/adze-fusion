# Adversarial review — W1-2 (Fusion document model + workspaces + threading)

**Reviewer:** P0 adversarial sub-agent
**Date:** 2026-06-11
**Target:** `research/findings/W1-2-fusion-doc-model.md`

## Verdict at a glance

**ACCEPT — grade A.** All 8 numbered questions answered (0 silently missing). 8 spot-checks performed (3 required); every quoted statement found verbatim at the cited source, including the load-bearing threading, transaction, object-graph, units, and entityToken quotes. All 5 self-reported [unverified] claims are genuinely marked in the file text. An active refutation attempt on the most load-bearing claim (single-threaded API, fireCustomEvent as sole cross-thread primitive) failed — the claim stands. No fabricated or misrepresented sources found. Corrections below are advisory, not blocking.

## Spot-checks performed

1. **Threading_UM** (direct fetch) — all six quoted statements verbatim: "you should not call any Fusion API functions within the worker thread", messageBox crash warning, "single queue where everyone has to wait their turn", "nothing else is happening inside Fusion", doEvents long-loop crash warning, fireCustomEvent as the worker-thread mechanism. PASS.
2. **Application_fireCustomEvent** (direct fetch) — additionalInfo is a string; return-true-means-queued-not-handled verbatim; "handled in the main thread when Fusion is idle" verbatim; page confirms **no documented payload size limit**, matching the finding's claim of absence. PASS.
3. **Commands_UM** (direct fetch) — all four transaction statements verbatim, including "bundled within a single transaction and can be undone with one undo" and the executePreview auto-abort passage. PASS.
4. **ComponentsProxies_UM** (direct fetch) — all eight object-graph quotes verbatim (the file even faithfully reproduces the source typo "geometry of it own"). PASS.
5. **BRepFace_entityToken** (direct fetch) — "saved and used at a later time", token-string instability, never-compare-tokens, isTemporary-false validity all verbatim. Page is **confirmed silent on cross-session/file-round-trip persistence** — independently validating the researcher's [unverified] flag on durability. PASS, with integrity bonus.
6. **Render forum thread** (td-p/9618616) — direct fetch 403 (consistent with researcher's note); quote confirmed verbatim via search: "The API doesn't support the Render workspace. However, you can use the API to switch to the workspace and to call commands, including the 'In-Canvas Render' command..." Thread dates to **July 2020**. PASS (see corrections).
7. **Units_UM** (direct fetch) — cm / radians / kg, "The internal units always use these types without any exceptions", CAM degrees / mm/min / seconds all verbatim. PASS.
8. **April 2026 product update** (via search; direct fetch 403 as researcher reported) — "The API for Metal Powderbed Process Simulation is now provided as a preview feature... access and execute Process Simulation workflows programmatically" confirmed. PASS.

## Refutation attempt (most load-bearing claim)

**Claim attacked:** "one process, one thread... all cross-thread communication funneled through `Application.fireCustomEvent`" — this drives the entire adze bridge-architecture implication. Searched for any alternative documented thread-safe entry point. Result: official docs and community sources consistently identify fireCustomEvent as the only documented mechanism for worker-thread → API communication; no alternative exists in the reference manual. **Refutation failed; claim stands.**

## Dangerous claims

None rise to load-bearing-and-wrong. Watch-list items (all already honestly flagged [unverified] in the file):

- entityToken cross-session durability — correctly flagged; the cited reference page is verifiably silent on it. The Stage 4 empirical test recommendation is the right call. Do not let the vault entry harden "saved and used at a later time" into "survives file round-trips."
- fireCustomEvent payload ceiling — correctly flagged as undocumented; absence of a documented limit is not a guarantee of unbounded payloads.

## Corrections (advisory, non-blocking)

1. **Render forum answer is from July 2020** — the finding does not date it. On vault promotion, add the date and a staleness caveat: the no-Render-API state was re-checked against the current reference manual by the researcher (no Render API found), so the conclusion holds today, but the quote itself is 6 years old.
2. Two quotes are silently truncated: additionalInfo description omits "...to the add-in in the primary thread" (the full text actually strengthens the main-thread claim) and the Render quote drops the leading "However,". Cosmetic; mark ellipses on promotion.

## Missed topics

Minor, none constitute scope drift against the 8 assigned questions:

- `registerCustomEvent`/`unregisterCustomEvent` lifecycle mechanics (the receiving side of fireCustomEvent) are implied but not enumerated.
- Palette (HTML UI) `sendInfoToHTML`/`incomingFromHTML` as an adjacent IPC surface — not asked, but relevant to the bridge-architecture implication and worth a follow-up note.
- Drawing product API status — Q3 listed only Design/Manufacture/Simulation/Render, so its omission is acceptable.
