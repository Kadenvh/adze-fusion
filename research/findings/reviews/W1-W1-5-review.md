# Adversarial review — W1-5 (Cross-platform reality: Mac vs Windows)

**Reviewer:** P0-pattern adversarial sub-agent
**Date:** 2026-06-11
**Target:** `research/findings/W1-5-cross-platform.md`

## Verdict at a glance

**Grade: A- — accept-with-corrections.** All 7 numbered questions answered (0 silent misses). Seven citation spot-checks performed; every quoted passage was found verbatim in the live source. The most load-bearing claim (official Fusion MCP Server is built into Fusion, not an App Store add-in) survived an active refutation attempt against Autodesk primary docs. Corrections are minor: one off-vocabulary confidence tag and two date attributions not carried by the cited sources.

Self-reported [unverified] markers: 3 of 4 are literally present in the text (end-to-end Mac MCP, line "Q4"; AuraFriday listing live status, "Q4"; on-disk folder name, "Q5"). The 4th (Python arm64-native on Apple Silicon) is marked `[inference from the above; no explicit Autodesk statement found]` rather than `[unverified]` — honest, but off the vault's tag vocabulary.

## Spot-checks performed

1. **"Is there a difference between Fusion on Mac and Windows" (Autodesk Support)** — confirmed verbatim via Tavily extract (page 403s on plain fetch): "Fusion on Mac and Windows offer the same features. However, some Add-in may be only compatible with certain OS. Also, using Fusion for manufacturing, the machining simulation may be slower on Mac." Date Nov 18, 2025 confirmed. CLEAN.
2. **System requirements page** — confirmed verbatim: "Windows 11, 23H2 (Build 22631 or newer)"; ARM64 "XtaJIT64/Prism emulation... has not yet completed certification"; macOS 14 Sonoma min ("OpenCore Legacy Patcher is not supported"); rec "macOS 15.7 Sequoia or newer / macOS 26 Tahoe or newer"; Rosetta 2 dependency note word-for-word; RAM 8/32 GB Win, 4/16 GB Mac; "Relief Pattern appearance on macOS 14 w/ Apple silicon". CLEAN.
3. **ADSKMCP help — Autodesk Fusion MCP Server page** — confirmed verbatim: "exposed by Fusion when running locally"; "Local-only"; "Desktop-only. Does not work with the Fusion web client"; enabled via "Preferences > General > API"; dynamic tooling. (Note: this help site serves a 404 shell to plain static fetch; content renders client-side — retrieved via Tavily advanced extract.) CLEAN.
4. **ADSKMCP help — Connecting page** — confirmed verbatim: "Check the Fusion MCP Server (runs locally on this device) checkbox"; "The default port is 27182"; Claude Desktop Settings > Extensions > Fusion flow. Also confirmed the docs are OS-silent, exactly as the finding states. CLEAN.
5. **Apple Silicon FAQ (Fusion blog)** — confirmed verbatim: auto-runs "the native ARM64 version" on M1/M2; "The main Fusion 360 application and its default add-ins now all run natively on Apple silicon. However, some individual processes and back-end services still temporarily require Rosetta 2"; "Will my installed Fusion 360 add-ins be compatible with Apple silicon? Initially, no. You'll need to enable the 'Open using Rosetta'". CLEAN.
6. **Python 3.14 forum thread (forums.autodesk.com, Feb 2026)** — quoted sentence confirmed verbatim (Brian Ekins reply): "Fusion updated to an embedded Python 3.14 interpreter, and that breaks many packages that ship compiled native extensions, especially ones that now include Rust-based binaries (which cryptography does)." Other replies corroborate "the latest release and the Python 3.14 update". CLEAN (see correction 3 on attribution nuance).
7. **App Store listing id 7269770001970905100** — 404 independently reproduced on 2026-06-11 via a second fetch path (Tavily): "404: Page not found" on the Design and Make Marketplace. Web search index still returns the listing and identifies AuraFriday as the publisher ("Windows 10/11 or macOS 10.15+", docs at AuraFriday/Fusion-360-MCP-Server, support ask@aurafriday.com). The finding's treatment — third-party listing, 404 on direct fetch, live status [unverified] — is accurate and now better-evidenced.

## Refutation attempt (most load-bearing claim)

**Claim attacked:** "The official Autodesk Fusion MCP Server is built into Fusion itself (Preferences > General > API, port 27182), not an App Store add-in; the App Store 'MCP Server for Autodesk® Fusion®' listing is third-party (AuraFriday)."

**Result: refutation failed.** Autodesk's own ADSKMCP help confirms every element directly: server "hosted by Fusion and is only available while Fusion is open", enabled by a Preferences checkbox, default port 27182, local-only, desktop-only. The ADSKMCP landing page catalogs exactly three Autodesk MCP servers (Product Help, Fusion local, Fusion Data remote) — none distributed via the App Store. Independent search confirms the App Store listing is AuraFriday's MCP-Link product, announced on the forums by community user OceanHydroAU, not Autodesk. The claim stands on primary sources.

## Dangerous claims

- **"Python add-ins run arm64-native when Fusion is native"** — marked as inference, never confirmed. It quietly feeds the "pure-Python or pay twice" decision implication's support-matrix math (Mac-x86_64 vs Mac-arm64). Synthesis must not treat it as established until the proposed `platform.machine()` empirical check is run. (Properly flagged in the file; listed here so Stage 2 carries the flag forward.)
- No fabricated or misrepresented sources found. Every spot-checked quote was verbatim.

## Corrections

1. **Normalize the tag** on the Apple-Silicon Python claim (Q3) from `[inference from the above; no explicit Autodesk statement found]` to `[unverified]` (optionally `[unverified — inference]`) to match the vault's cited/[empirical]/[unverified] vocabulary.
2. **"ARM64-native since July 2023"** — the date is correct (Autodesk's "July 2023 Product Update — What's New" blog announces native Apple Silicon) but the cited Apple Silicon FAQ does not carry it. Add the explicit citation: https://www.autodesk.com/products/fusion-360/blog/july-2023-product-update-whats-new/
3. **"Python 3.14 in the January 2026 update"** — the cited Feb 2026 thread confirms the 3.14 jump and the breakage but not the month "January"; the quoted diagnosis is a community expert's (Brian Ekins) reply, partially hedged in-thread ("I can't confirm this"). Soften to "early 2026 update" or cite Fusion release notes for the month; attribute the quote to Ekins' reply rather than the thread generically.
4. **Strengthen (optional):** note that the AuraFriday listing 404 was reproduced via a second independent fetch path on 2026-06-11 — live status still [unverified] (could be temporary delisting), but "404 on direct fetch" is now multiply evidenced.

## Missed topics

- **Q6 partial:** the brief named "paths, file dialogs, UI scaling" — paths and HTML-UI divergence are covered; file-dialog and UI/Retina-scaling issues are not specifically addressed. The researcher honestly flagged thin coverage, so this is a gap, not a silent miss.
- The Fusion "AI Assistant capabilities" doc (`AA_Capabilities`, linked from the MCP server page) was not consulted — it may contain the missing OS-support statement for the MCP/AI Assistant surface and is the cheapest next probe for the end-to-end-Mac open question.
- TypeScript add-in path: deferred to open questions without investigation — acceptable for this stream's scope.
