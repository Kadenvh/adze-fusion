---
title: Autodesk Fusion MCP (first-party)
category: 05-mcp-servers
tags: [mcp-server, autodesk, fusion-360, official, first-party]
status: promoted
confidence: medium
license: proprietary
language: unknown
maturity: tech-preview
---

# Autodesk Fusion MCP (first-party)

Autodesk's official MCP server for Fusion 360. Local server, runs while Fusion is active, speaks to any MCP-capable HTTP client (Claude Desktop, Cursor, the [[../04-tools/anthropic-fusion-connector]]). Tech preview as of 2026-05.

## First principles

This is the **MCP server that sits underneath** the Anthropic Fusion connector. The connector is just Claude-side configuration that points at this server. The server runs as a Fusion add-in (App Store listing has Mac variant confirmed), exposes Fusion's runtime as MCP tools, and bridges the worker-thread → main-thread `CustomEvent` hop ([[../03-apis/fusion-adsk-api]]) inside the add-in. As an MCP server, it is host-agnostic by construction — Claude is just one of multiple compatible clients.

## Surface

| Aspect | Detail |
|---|---|
| License | Proprietary (Autodesk) |
| Maturity | Tech Preview (2026) |
| Transport | HTTP localhost |
| Default port | `[unverified]` — `27182` claimed by single third-party blog; Autodesk's own page (App Store listing for "MCP Server for Autodesk Fusion") was not deep-fetched for port detail |
| Distribution | Autodesk App Store (Mac variant ID `7269770001970905100`) |
| Compatible clients | Claude Desktop, Cursor, "any MCP-capable HTTP client" per Autodesk AI page |
| Requires | Active Fusion subscription; Fusion running |
| Sibling | **Fusion Data MCP** — cloud variant, no Fusion required |

The full first-party MCP suite Autodesk has announced:

- **Fusion MCP** (this entry) — tech preview, local, Fusion required
- **Fusion Data MCP** — tech preview, cloud, no Fusion needed
- **Revit MCP** — tech preview
- **Product Help MCP** — read-only across 110+ Autodesk products
- **Fusion Automation MCP** — private beta (application required)
- **InfoWorks Hydraulic Modeling MCP** — coming soon

## Implications for adze

- **The reference standard for any adze-fusion MCP server.** Whatever adze ships must compose alongside this — same transport, compatible permission model, same on-disk port-and-config UX.
- **Subscription gating cascades.** Anything that depends on Fusion-MCP-running-locally inherits the "Fusion subscription required" gate. Pure cloud paths (Fusion Data MCP) avoid this gate but lose access to live geometry.
- **Probable narrowing of the community-MCP space.** Once the first-party server stabilizes out of tech preview, expect community servers ([[faust-machines-fusion360-mcp]], [[sockcymbal-fusion-mcp]], etc.) to lose justification unless they offer something the first-party doesn't (e.g., richer tool surface, governance, observability).
- **Marketplace certification path is for this and its peers.** Autodesk's MCP Publisher Guide and certification ([[../06-communities/autodesk-design-and-make-marketplace]]) are how third-party MCPs become Assistant-callable on the same plane.

## Open questions

- Default port — needs the App Store listing fetched for ground truth. `[unverified]`
- Tool surface — count and category breakdown is not enumerated in fetched sources.
- Auth between Claude (or any client) and the local server — token? localhost-only IP gate? `[unverified]`
- GA timing — tech-preview-to-GA estimate is not published.

## Sources

- [Autodesk MCP Servers — Autodesk AI](https://www.autodesk.com/solutions/autodesk-ai/autodesk-mcp-servers) — fetched 2026-05-15 (403, content via search snippets)
  > "Compatible with Claude Desktop, Cursor, any MCP-capable HTTP client."
- [Building for Agentic AI: What's New in Autodesk Platform Services — APS blog](https://aps.autodesk.com/blog/building-agentic-ai-whats-new-autodesk-platform-services) — fetched 2026-05-15
  > Lists Fusion MCP, Fusion Data MCP, Revit MCP, Product Help MCP (all tech preview), Fusion Automation MCP (private beta), InfoWorks Hydraulic (coming soon).
- [MCP Server for Autodesk Fusion — Autodesk App Store (Mac variant)](https://apps.autodesk.com/FUSION/en/Detail/Index?id=7269770001970905100&ln=en&os=Mac) — fetched 2026-05-15 (via search)

## Related

- [[../01-concepts/mcp-protocol]]
- [[../04-tools/anthropic-fusion-connector]]
- [[../02-platforms/fusion-360]]
- [[faust-machines-fusion360-mcp]]
- [[../06-communities/autodesk-design-and-make-marketplace]]
