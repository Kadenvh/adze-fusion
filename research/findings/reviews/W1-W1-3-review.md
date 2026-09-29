# Adversarial review — W1-3 (Anthropic Fusion connector deep-dive)

**Reviewer:** P0-pattern adversarial sub-agent
**Date:** 2026-06-11
**Target:** `research/findings/W1-3-anthropic-fusion-connector.md`

## Verdict at a glance

**Grade: A- — accept-with-corrections.**

All 8 numbered questions are answered; gaps are explicitly declared ("could not determine"), not silently dropped. Scope-drift count: 0. Eight citations were re-fetched; **every quoted primary-source string was found verbatim** — no fabrication, no misquotation. The headline claim (port 27182 officially documented) survived an active refutation attempt from two independent Autodesk primary surfaces. Two real but fixable issues: (1) the §2 conclusion "the free hobbyist tier does NOT get MCP access" is broader than what the evidence scopes — community evidence shows third-party add-in bridges run on personal licenses; (2) the knightli tolerance-limitation detail carries a provenance caveat but not a literal `[unverified]` tag. All six self-reported `[unverified]` inferences were confirmed marked in the file text except #6 (knightli), which is flagged by provenance note only.

## Spot-checks performed

All re-fetched 2026-06-11. Note: `help.autodesk.com` is a JS-rendered SPA — direct WebFetch returns a 404 shell; Tavily advanced extract returns full content. Direct `autodesk.com` fetches 403 (matches the finding's own note). Neither is evidence of fabrication.

1. **Connecting to the Autodesk Fusion MCP Server** (help.autodesk.com ADSKMCP) — VERBATIM: "The default port is `27182`. If you change the port in Fusion preferences, update the URL in your client configuration to match." Claude Desktop Settings > Extensions walkthrough and "If the Tool permissions list is populated, Claude is successfully connected" both confirmed.
2. **Autodesk Fusion MCP Server help** — VERBATIM: "Local-only. Runs on the same machine as Fusion — no remote connections or authentication required." / "Dynamic tooling. Available tools are discovered by your MCP client at connection time and may change as capabilities evolve." / "Desktop-only. Does not work with the Fusion web client." Preferences > General > API requirement confirmed.
3. **Autodesk Support article (dated May 5, 2026)** — VERBATIM: "Third‑Party AI Tools cannot be integrated into Autodesk Fusion with a personal use license." / "Switch to a commercial Fusion Subscription." / "If Port 27182 is blocked, choose another port. Note: This will also need changing to match in the Third party AI tool."
4. **Troubleshooting (Fusion MCP)** — VERBATIM: "Verify your client is configured with `http://127.0.0.1:27182/mcp`." / cause "No document or workspace is open in Fusion" for "Commands do not execute" / "Update Fusion to the latest version. The Autodesk Fusion MCP Server option is only available in recent releases."
5. **App Store Mac listing** (`apps.autodesk.com/.../id=7269770001970905100&os=Mac`) — CONFIRMED 404 "Page not found" on Design and Make Marketplace. Corroborated independently by an Autodesk forum thread ("The add-in has disappeared from the autodesk app store", forums.autodesk.com td-p/13881165).
6. **Anthropic, Claude for Creative Work** — VERBATIM: "Autodesk Fusion allows designers and engineers with a Fusion subscription to create and modify 3D models through conversations with Claude." No Claude plan named — confirmed.
7. **support.claude.com connectors article (11176164)** — VERBATIM: "Desktop extensions are available to all users on Claude Desktop." / "Claude inherits each person's permissions from the connected service."
8. **APS blog (bringing-fusion-claude-creative-work)** — VERBATIM: "You stay in control of what's accessed and how it's used, with protections aligned to Autodesk's privacy, security, and product standards." / "for Fusion subscribers" framing confirmed ("Autodesk Fusion MCP connects Claude directly to the Fusion environment for Fusion subscribers...").

**Refutation attempt (most load-bearing claim):** "27182 is officially documented as the default port." Attacked via independent re-fetch of all three Autodesk surfaces — the port appears verbatim on the Connecting page, the Troubleshooting page (full URL, twice), and the May 2026 support article. Refutation FAILED; the claim stands on multiple independent primary sources.

## Dangerous claims

1. **§2 / decision implications: "The free-tier cohort officially cannot use MCP-based AI tools."** Overreach relative to evidence scope. The support-article sentence is literally that broad, but it appears in a troubleshooting article about the official built-in MCP server. Counter-evidence found during review: ndoo's `fusion360-mcp-bridge` README states "**Free personal licence is sufficient**" for its add-in-based bridge (community, unverified), and community bridges (frankhommers, faust-machines, JustusBraitinger) document personal-tier usage of the regular add-in API. The sourced claim is "the *official built-in* Fusion MCP Server is gated to commercial subscriptions"; whether the gate is technical tier-enforcement or ToS-only, and whether it extends to third-party add-in bridges, is undetermined. The finding's own hedge ("needs a licensing-terms read before targeting hobbyists") is correct — but the bald §2 bolded sentence should be narrowed before vault promotion.
2. **knightli tolerance/precision limitation** — relies partly on the P1 prior pass (2026-05-15); the 2026-06-11 re-fetch returned partial content. Flagged in text by provenance note but lacks the literal `[unverified]` / confidence tag the vault rules require. Low stakes (clearly marked community) but the tag is missing.

All other self-reported `[unverified]` inferences confirmed genuinely marked in the file text: all-plans-including-Free (§3), any-local-process corollary (§4), Mac-identical (§7), egress-to-Anthropic-infrastructure (§4), tool-name/session-contention (§6).

## Corrections

1. **Narrow §2's bolded conclusion** to: "The free hobbyist tier does NOT get access to the *official built-in* Fusion MCP Server [Autodesk support, 2026-05-05]. Third-party add-in-based bridges reportedly work on personal licenses (ndoo bridge README: 'Free personal licence is sufficient') [community, unverified] — enforcement mechanism (technical vs ToS) undetermined." Propagate the same scoping into the "Personal-use exclusion carves out the hobbyist market" decision implication.
2. **Add a confidence tag** to the knightli limitation sentence in §8, e.g. `[community, partially re-verified — limitations detail per P1 pass 2026-05-15]`.
3. Minor enrichment for §3: the connectors article's only plan gating is "Free users are limited to one custom connector" — which applies to *custom connectors*, not desktop extensions; citing it would strengthen the no-plan-gating conclusion.
4. Trivial: the help page calls the deferred page "AI Assistant capabilities documentation" (finding says "Autodesk Assistant capabilities page"); same guid (`AA_Capabilities`), no action needed beyond awareness.

## Missed topics

- **Autodesk-side data handling/telemetry for MCP traffic** (Q4 edge): the finding correctly says no source enumerates egress to Anthropic, but did not consult Autodesk's trust center / AI transparency documentation for whether Fusion logs or transmits MCP tool-call telemetry to Autodesk. Small gap; current "could not determine" stance is honest.
- **Autodesk forum thread td-p/13881165** ("Driving Fusion via AI using MCP server add-in") independently corroborates the App Store listing disappearance and documents the transitional add-in era (cert errors, missing Windows download) — useful provenance for the "why did the listing disappear" open question.
- Education/startup tier status — already explicitly flagged as open; not a silent miss.
