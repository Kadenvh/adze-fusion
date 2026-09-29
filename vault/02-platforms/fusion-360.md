---
title: Autodesk Fusion 360
category: 02-platforms
tags: [platform, fusion-360, autodesk, cad]
status: promoted
confidence: high
---

# Autodesk Fusion 360

Autodesk's cloud-connected parametric CAD/CAM/CAE host. Runs cross-platform (Windows + macOS Intel + macOS ARM-native) with an in-process Python add-in runtime (Python 3.14 since January 2026) as the canonical native extensibility surface.

## First principles

Fusion 360 is, for adze's purposes, a **cross-platform parametric modeler with three integration surfaces**: a local in-process Python/C++ add-in API (`adsk.*`), a cloud REST API (Autodesk Platform Services / APS, formerly Forge) for design-data and automation, and — newly, in 2026 — a first-party MCP server suite that exposes a subset of Fusion's runtime as JSON-tool calls [[../09-sources/aps-fusion-claude-creative-work]]. Unlike SOLIDWORKS (its sibling project's host), there is no COM/.NET-Windows-only surface; Fusion's native runtime is the same on Mac. The threading model is single-threaded UI-and-document state, so any add-in that wants to talk to an external process bridges through a fixed localhost socket and marshals work back to Fusion's main thread via `CustomEvent`.

## Surface

| Aspect | Detail |
|---|---|
| Hosts | Windows, macOS Intel, macOS ARM (verified [[../09-sources/anthropic-creative-work-announcement]]) |
| Native extensibility | Python (primary) or C++ add-ins; in-process. Embedded Python is **3.14** since Build v.2606.1.22 (2026-01-21); Autodesk upgrades it aggressively (3.7 → 3.9.7 → 3.12 → 3.14), breaking native-extension add-ins at each jump |
| Cloud surface | Autodesk Platform Services (APS) REST APIs + OAuth 2.0 |
| First-party AI | Autodesk Assistant ([[../04-tools/autodesk-assistant]]) — in-product agentic AI with Python "Script Execute" |
| First-party MCP | [[../05-mcp-servers/autodesk-fusion-mcp]] (local, requires Fusion running) and Fusion Data MCP (cloud) |
| Distribution | Autodesk App Store (per-add-in listings) + Design and Make Marketplace (certified MCPs) |
| Subscription gate | Anthropic's Fusion connector + the built-in MCP server require a commercial Fusion subscription; **personal-use (free hobbyist) licenses are officially excluded** from the built-in MCP server (Autodesk Support, 2026-05-05) [[../04-tools/anthropic-fusion-connector]] |

Threading discipline is identical to SOLIDWORKS in shape (single owner thread) but achieved differently: Fusion's pattern is `app.fireCustomEvent` from the worker thread back to the main thread, where document mutations actually run. Multiple community MCP servers ([[../05-mcp-servers/faust-machines-fusion360-mcp]], [[../05-mcp-servers/sockcymbal-fusion-mcp]]) all implement this same pattern — a TCP/HTTP listener on the worker side, `CustomEvent` to hop to main, mutation, response.

## Implications for adze

- **Cross-platform from day one is non-negotiable.** Adze-cad's Windows-only COM posture does not transfer — Fusion is genuinely Mac+Windows, and community MCP servers already ship both. A Windows-only adze-fusion competes against shipped cross-platform alternatives.
- **The embedded Python version is a moving target, not a fixed floor.** Fusion runs Python 3.14 today but Autodesk upgrades the interpreter with product builds, and each jump has broken add-ins that ship compiled native extensions. Safe postures: pure-Python in-process code, or isolate version-sensitive logic in a sidecar (the dominant pattern — see [[../07-patterns/in-process-addin-localhost-mcp]]).
- **Three integration surfaces means three architecture choices.** Local-only (add-in + MCP sidecar, the [[../04-tools/anthropic-fusion-connector]] / [[../05-mcp-servers/faust-machines-fusion360-mcp]] pattern), cloud-only (Fusion Data MCP, no Fusion running), or hybrid ([[../05-mcp-servers/sockcymbal-fusion-mcp]]). ADR-001 must pick.
- **Autodesk Assistant absorbs the simple cases.** Wrapping an LLM for prompt-to-sketch loses to Assistant the moment Assistant ships the same capability — [[../04-tools/project-salvador]] is the cautionary precedent.

## Open questions

- Whether the Anthropic Fusion connector is OS-restricted in practice (Mac install was not verified end-to-end; official MCP docs are OS-neutral). `[unverified]`
- Whether Fusion's Python 3.14 build is standard-GIL or free-threaded. `[unverified]`

## Sources

- [Bringing Fusion onto Claude for Creative Work — APS blog](https://aps.autodesk.com/blog/bringing-fusion-claude-creative-work) — fetched 2026-05-15
- [Fusion 2026 Release Notes](https://help.autodesk.com/cloudhelp/ENU/Fusion-ReleaseNotes/files/FIC-REL-NOTES-INC.htm) — fetched 2026-06-11
  > "Updated the embedded Python version from 3.12 to 3.14" — Build v.2606.1.22 (2026-01-21). (The Fusion Help "Python Specific Issues" page states no version; the P1-era "3.7" claim was stale.)
- [Claude for Creative Work — Anthropic news](https://www.anthropic.com/news/claude-for-creative-work) — fetched 2026-05-15

## Related

- [[../03-apis/fusion-adsk-api]]
- [[../04-tools/anthropic-fusion-connector]]
- [[../04-tools/autodesk-assistant]]
- [[../07-patterns/in-process-addin-localhost-mcp]]
- [[../09-sources/aps-fusion-claude-creative-work]]
