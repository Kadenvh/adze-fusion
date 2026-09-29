---
title: Bringing Fusion onto Claude for Creative Work — APS blog
category: 09-sources
tags: [source, autodesk, aps, fusion-360, mcp, claude]
url: https://aps.autodesk.com/blog/bringing-fusion-claude-creative-work
fetched: 2026-05-15
author: Autodesk Platform Services
type: official-doc
status: promoted
confidence: high
---

# Bringing Fusion onto Claude for Creative Work — APS blog

Autodesk's companion post to Anthropic's 2026-04-28 launch. Primary source for the framing that Autodesk authored the **Fusion MCP** and Anthropic integrated it as the Fusion connector.

## Why this source is in the vault

Anchors the **architectural ownership split**: Autodesk built the MCP server, Anthropic built the connector that uses it. This is the source that disambiguates P1's loose "jointly authored" framing (which P0 flagged as not in any primary source).

Other entries that cite this:

- [[../04-tools/anthropic-fusion-connector]] — architectural ownership
- [[../05-mcp-servers/autodesk-fusion-mcp]] — what the connector underlies
- [[../01-concepts/mcp-protocol]] — MCP positioning per Autodesk
- [[../02-platforms/fusion-360]] — subscription gating context

## Key extracts

> "Fusion Model Context Protocols (MCPs) lets third-party AI systems connect to Fusion, enabling them to access design context and perform actions securely."

> "Autodesk Fusion MCP connects Claude directly to the Fusion environment for Fusion subscribers."

> The post does **NOT** mention port `27182` or any specific default port — P1's port claim is sourced elsewhere (`knightli.com`), not here.

## Reliability notes

- **Author:** Autodesk Platform Services (official blog).
- **Type:** Product blog accompanying Anthropic's launch.
- **Confidence:** High for Autodesk-side facts (subscription requirement, MCP framing, server ownership).
- **Limitation:** Doesn't enumerate technical install details — port, transport, auth between client and server. Those would need an Autodesk install-doc fetch (which returned 403 in P1).

## Related

- [[../04-tools/anthropic-fusion-connector]]
- [[../05-mcp-servers/autodesk-fusion-mcp]]
- [[anthropic-creative-work-announcement]]
- [[../02-platforms/fusion-360]]
