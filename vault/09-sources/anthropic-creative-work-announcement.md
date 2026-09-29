---
title: Claude for Creative Work — Anthropic announcement
category: 09-sources
tags: [source, anthropic, fusion-360, mcp, connector, launch]
url: https://www.anthropic.com/news/claude-for-creative-work
fetched: 2026-05-15
author: Anthropic
type: official-doc
status: promoted
confidence: high
---

# Claude for Creative Work — Anthropic announcement

The 2026-04-28 launch post for the 9 Claude creative-work connectors, including the Autodesk Fusion connector. Primary source for the launch date, the connector list, and the "MCP-accessible to other LLMs" architectural claim.

## Why this source is in the vault

Anchors the launch date and the connector list. Other entries that cite this:

- [[../04-tools/anthropic-fusion-connector]] — launch facts, MCP architecture
- [[../01-concepts/mcp-protocol]] — the "accessible to other LLMs" framing
- [[../05-mcp-servers/autodesk-fusion-mcp]] — connector points at this underlying server
- [[../02-platforms/fusion-360]] — Mac+Windows support context

## Key extracts

> 9 connectors launched 2026-04-28: Ableton, Adobe, Affinity, Autodesk Fusion, Blender, Resolume Arena, Resolume Wire, SketchUp, Splice.

> "Because the connector is built on MCP, it is accessible to other LLMs in addition to Claude." (claim made specifically about Blender — pattern-true for MCP-built connectors generally; P0 flagged the silent generalization in P1).

> Fusion connector requires an active Fusion subscription.

## Reliability notes

- **Author:** Anthropic. First-party, official.
- **Type:** Product launch announcement (news post).
- **Confidence:** High for facts under Anthropic's control (launch date, connector list, Anthropic-quoted phrases). Medium for facts about underlying Autodesk implementation — those are properly sourced to APS blog ([[aps-fusion-claude-creative-work]]).
- **Known gaps:** Does not enumerate per-connector setup, default ports, or subscription tier requirements. P1's "port 27182" claim is NOT in this source.

## Related

- [[../04-tools/anthropic-fusion-connector]]
- [[../05-mcp-servers/autodesk-fusion-mcp]]
- [[aps-fusion-claude-creative-work]]
- [[../01-concepts/mcp-protocol]]
