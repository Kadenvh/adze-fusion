# Adversarial review — W1-4 (App Store mechanics + monetization + Marketplace certification)

**Reviewer:** P0-pattern adversarial sub-agent
**Date:** 2026-06-11
**Target:** `research/findings/W1-4-app-store.md`

## Verdict at a glance

**accept-with-corrections (grade A-).** All 8 numbered questions answered — zero scope drift. Seven citations re-fetched or independently corroborated; every verbatim quote checked out, including the implausible-looking "0.005 seconds" load-time figure and even a grammatical error copied faithfully from the source page ("the standard the Autodesk App Store app installer"). No fabrication, no misquoting. Refutation attempts on the two most load-bearing claims (zero commission; open-and-empty Fusion MCP certification slot) failed to refute — both corroborated. Three fixable issues: one mis-tagged confidence label, one over-confident economic framing on a pre-rebrand legal document, and one competitive-landscape omission that inflates the "first-mover" implication.

## Spot-checks performed

| # | Source | Result |
|---|---|---|
| 1 | [MCP publisher guide](https://aps.autodesk.com/marketplace/mcp-publisher-guide) | **Verified.** Manifest JSON, Publisher Declaration security attestations, "All external connections must use HTTPS", "Only access the minimum data required for functionality", appsubmissions@autodesk.com, resubmission allowed — all present verbatim. Confirmed: NO ISO 42001 mention on this page (supports the researcher's downgrade). No fees/waitlist mentioned. |
| 2 | [Marketplace blog](https://aps.autodesk.com/blog/design-and-make-marketplace-where-your-solutions-meet-industry-agentic-ai-workflows) | **Verified.** "Open to every developer, MCP author, APS builder", "isn't an open directory… quality and standards review", Assistant-eligibility quote — all verbatim. Pub date 11 May 2026. |
| 3 | [Marketplace getting started](https://aps.autodesk.com/marketplace/getting-started) | **Verified.** Requirements checklist (guidelines / privacy policy / PayPal if charging), MCP Tool Manifest + Publisher Declaration, "Autodesk creates the final package file", 24-hour response, ADN positioned as optional. |
| 4 | [Fusion publishing guidelines](https://aps.autodesk.com/marketplace/publisher-center/fusion-publisher-guidelines) | **Verified verbatim**, all six claims: "Keep your load time under 0.005 seconds", `APPLOG_FOR_PERFORMANCE=yes`, `.bundle` + `PackageContents.xml`, Autodesk-built Win/Mac installers, version number per submission, latest-Fusion relevance. Confirmed: no publisher code-signing requirement on the page. |
| 5 | [ADN membership](https://aps.autodesk.com/developer/overview/autodesk-developer-network-membership) | **Verified.** 1 user 1,750 / 2–5 users 3,000 / 6+ 5,500, currency by region, calendar year not prorated, startup 3-yr free, university free, betas + 1:1 support. |
| 6 | [AUTOM8LABS listing](https://marketplace.autodesk.com/apps/0639863e-7269-49a3-aade-a49442c010d4) | Direct fetch returns blank (JS-rendered) — but **corroborated by 5 independent sources**: vendor site (autom8labs.io), [apps.autodesk.com detail page](https://apps.autodesk.com/RVT/en/Detail/Index?id=6080935186426081466), rvtplugins.com review, LinkedIn announcement, aidirectory.aecmag.com. 36 tools, free, Claude Desktop + Cursor → Revit — all match. |
| 7 | [Publisher FAQ PDF](https://damassets.autodesk.net/content/dam/autodesk/www/adn/pdf/frequently-asked-questions.pdf) | Extraction fails (binary/font streams) — exactly as the researcher disclosed. Zero-commission / up-to-30% **corroborated independently** via search-indexed text of the [Publisher Agreement](https://apps.autodesk.com/Content/pdf/Publisher.pdf) ("commission rate of 0.0%… may be modified") and the AutoCAD DevBlog FAQ mirror. |

**Refutation attempts (most load-bearing claims):**
- *"Zero commission today"* — searched for any Design-and-Make-Marketplace-era fee/revenue-share terms superseding the 2024 agreement. None found. Not refuted, but see Dangerous claim 1.
- *"Open MCP certification + empty Fusion slot"* — searched for any third-party Fusion MCP on the Autodesk Marketplace. None found; the claim stands as stated. However, numerous third-party Fusion MCP servers exist OUTSIDE the marketplace (GitHub Joe-Spencer/fusion-mcp-server, mcpmarket.com, PulseMCP, LobeHub) — see Dangerous claim 2.

**[unverified] self-report audit:** 3 of 4 self-reported claims are genuinely tagged [unverified] in the file (ISO 42001 third-party, ADN-not-required, installer signing). The 4th — "Fusion AI shelf first-party-only" — is tagged `[empirical — absence of evidence…]`, not `[unverified]`. Mismatch; see Corrections.

## Dangerous claims

1. **"Distribution economics are a non-issue: zero commission today."** True per available documents, but the Publisher Agreement is dated **August 13, 2024 — ten months before the Marketplace rebrand**. No Marketplace-era publisher agreement was located by researcher or reviewer. The summary states the favorable economics more confidently than a pre-rebrand legal doc supports, especially for the new MCP listing class (the file's own open questions #3–4 acknowledge this, but the summary doesn't carry the caveat).
2. **"A first-mover certification slot exists" / "nearly empty for Fusion."** The *certified Marketplace* slot is empty, but the *niche* is not: multiple uncertified third-party Fusion MCP servers circulate on GitHub and MCP registries. Any strategy built on this finding should read "first certified mover," not "first mover."
3. **Review-timeline figures ("24hr to 48hr" contact, "2 to 3 working days" testing)** rest on search snippets of a legacy FAQ PDF that resists text extraction; reviewer could not independently re-confirm the figures, and they predate the Marketplace (which separately promises feedback "within 24 hours"). Disclosed in the file, but should stay caveated through vault promotion.

## Corrections

1. Line 64: re-tag "the Fusion AI shelf is effectively first-party-only" from `[empirical]` to `[unverified]`. Absence-of-evidence from an unreliable JS-rendered search is not an empirical observation of zero under the project's source-rigor rules; the researcher's own self-report classifies it as unverified.
2. Section 2 / Decision implications: add the caveat that the zero-commission terms come from the Aug-2024 Publisher Agreement and no Marketplace-era agreement superseding it was located.
3. Section 5 / Decision implications: note that uncertified third-party Fusion MCPs exist outside the Marketplace (GitHub, mcpmarket.com, PulseMCP) — the certification slot is empty, the competitive niche is not.

## Missed topics

- Whether legacy App Store publisher contracts/terms carry over verbatim to the Design and Make Marketplace, or a new publisher agreement is issued at submission time (directly bears on the commission claim).
- The off-marketplace Fusion MCP ecosystem as competitive context for the "first-mover" implication (arguably another stream's territory, but the decision implication leans on it).
- Whether the legacy FAQ's review timeline still applies post-rebrand vs. the Marketplace's "within 24 hours" promise — the two are quoted side by side without reconciliation.

No numbered questions were silently dropped: 8/8 answered (Q8's count honestly reported as undeterminable with method limitation stated).
