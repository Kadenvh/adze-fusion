---
title: Anthropic Claude Fusion 360 Connector
category: 04-tools
tags: [tool, anthropic, claude, fusion-360, mcp, connector]
status: promoted
confidence: medium
license: proprietary
language: N/A
maturity: production
---

# Anthropic Claude Fusion 360 Connector

The first-party Claude integration for Fusion 360. Launched 2026-04-28 as one of 9 "Claude for Creative Work" connectors. Built on MCP and uses Autodesk's first-party Fusion MCP server underneath.

## First principles

The connector is **not its own AI logic**. It is a Claude-side configuration that points the Claude client at a locally-running Autodesk Fusion MCP server [[../05-mcp-servers/autodesk-fusion-mcp]]. Anthropic shipped the connector; Autodesk shipped the MCP server. The Anthropic news page describes Blender connectivity as "built on MCP… accessible to other LLMs in addition to Claude" — the same architectural truth applies to the Fusion connector (it is MCP underneath, so other MCP-capable clients can use the same server). P0 review flagged that Anthropic did not assert "jointly authored" with Autodesk; Anthropic shipped the connector integration, Autodesk shipped the MCP.

## Surface

| Aspect | Detail |
|---|---|
| Launch date | 2026-04-28 (Claude for Creative Work — 9 connectors) |
| Distribution | Built into Claude Desktop / Claude.ai connector UI |
| Underlying server | Autodesk's official Fusion MCP add-in (local) |
| Setup | Enable MCP server in Fusion → Preferences → General → API; add Fusion connector in Claude with localhost + port |
| Default port | **`[unverified]` — `27182` reported by one third-party blog (knightli.com) which itself calls it an "example"**; Anthropic news + APS blog do not mention a port; only verified at install time |
| Subscription | Active Fusion subscription required |
| OS reach | Theoretically Mac + Windows (Fusion runs on both); end-to-end Mac install not verified in P1 |
| Capabilities | Read geometry, modify parts, run Fusion API operations, write parametric history to active file |

## Implications for adze

- **The "Claude → Fusion" use case is covered out of the box.** Adze-fusion cannot differentiate on basic connectivity.
- **The published UX bar is high.** Single-click connection from Claude is what the user already experiences; any adze-fusion add-in must match or exceed that or has no reason to exist.
- **Connector ≠ adze-fusion's distribution path.** Anthropic's connector list is curated by Anthropic, not a marketplace. Adze-fusion ships via Autodesk's Design and Make Marketplace ([[../06-communities/autodesk-design-and-make-marketplace]]) and/or App Store — not as an Anthropic connector.
- **MCP composability is the lever.** Because the connector is MCP underneath, adze-fusion can ship as a separate MCP server that runs alongside Autodesk's — both visible to Claude, with adze adding the layers (governance, recipes, multi-CAD, ribbon UX) that the bare connector lacks.

## Open questions

- Actual default port — needs Autodesk App Store listing fetch or Autodesk-published doc. `[unverified]`
- Mac install verified end-to-end — Anthropic's Mac claim is via Fusion's Mac support, not via Anthropic-published Mac install walkthrough. `[unverified]`
- Whether the connector exposes any UX hook for third-party MCP composition (or if it's a single-MCP-per-connector model).
- Licensing of the connector itself — is it Pro/Max-gated or available on free Claude? Not addressed in P1.

## Sources

- [Claude for Creative Work — Anthropic news](https://www.anthropic.com/news/claude-for-creative-work) — fetched 2026-05-15
  > Launch 2026-04-28; 9 connectors (Ableton, Adobe, Affinity, Autodesk Fusion, Blender, Resolume Arena, Resolume Wire, SketchUp, Splice).
- [Bringing Fusion onto Claude for Creative Work — APS blog](https://aps.autodesk.com/blog/bringing-fusion-claude-creative-work) — fetched 2026-05-15
  > "Autodesk Fusion MCP connects Claude directly to the Fusion environment for Fusion subscribers."
- [Connecting Claude to Fusion 360 — knightli.com](https://knightli.com/en/2026/05/14/claude-fusion-360-mcp-step-model-edit/) — fetched 2026-05-15
  > "The default port `27182`" — phrased as "example" by author; **single-source claim, treat as [unverified]**.

## Related

- [[../01-concepts/mcp-protocol]]
- [[../05-mcp-servers/autodesk-fusion-mcp]]
- [[../02-platforms/fusion-360]]
- [[../09-sources/anthropic-creative-work-announcement]]
- [[../09-sources/aps-fusion-claude-creative-work]]
