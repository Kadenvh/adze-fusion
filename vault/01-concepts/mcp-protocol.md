---
title: Model Context Protocol (MCP)
category: 01-concepts
tags: [concept, mcp, protocol, integration]
status: promoted
confidence: high
---

# Model Context Protocol (MCP)

An open transport + tool-description protocol that lets an AI client (Claude, Cursor, any compatible host) invoke external "tools" exposed by a separate server process, over either stdio or HTTP.

## First principles

MCP is **not** an LLM, an agent framework, or a vendor lock-in. It is a thin contract: a server declares tools, resources, and prompts as JSON schemas; a client speaks JSON-RPC over a transport (stdio for local subprocesses, HTTPS for remote services); the client decides when to call which tool. Because the protocol is open, the same server can be reused across LLM hosts — Autodesk explicitly notes that the Fusion connector "built on MCP… is accessible to other LLMs in addition to Claude" [[09-sources/anthropic-creative-work-announcement]] (claim originally made about Blender, generalizes to all MCP-built connectors). MCP is the integration boundary, not the intelligence.

## Surface

| Aspect | Detail |
|---|---|
| Transports | stdio (local subprocess), HTTP/SSE (remote / cloud) |
| Server declares | Tools (callable), Resources (readable), Prompts (templates) |
| Manifest | JSON schema per tool: name, description, input schema |
| Auth | Out-of-band: env vars for stdio, headers/OAuth for HTTP |
| Discovery | Client lists servers; project-scope via `.mcp.json` for Claude Code |

Autodesk's MCP Publisher Guide [[09-sources/aps-mcp-publisher-guide]] formalizes a 4-step submission for first-party-grade MCPs: (1) author a Tool Manifest, (2) complete a Publisher Declaration (security attestations), (3) submit to `appsubmissions@autodesk.com`, (4) publish + monitor. HTTPS is required for any external connection; stdio is acceptable for purely local servers.

The CAD ecosystem has standardized on MCP rapidly: Anthropic shipped 9 MCP-built creative-work connectors on 2026-04-28 [[09-sources/anthropic-creative-work-announcement]]; Autodesk shipped Fusion MCP, Fusion Data MCP, Revit MCP, and Product Help MCP in 2026 [[09-sources/aps-building-agentic-ai]]; the open-source community has produced 6+ independent Fusion MCP servers (see [[05-mcp-servers/faust-machines-fusion360-mcp]], [[05-mcp-servers/sockcymbal-fusion-mcp]], [[05-mcp-servers/joe-spencer-fusion-mcp]]).

## Implications for adze

- **MCP-first is the default architectural posture.** Both Anthropic and Autodesk treat MCP as the canonical CAD-integration boundary; non-MCP architectures swim upstream and lose access to Autodesk Assistant orchestration + the Design and Make Marketplace [[06-communities/autodesk-design-and-make-marketplace]].
- **Protocol-level differentiation is impossible.** The protocol is open; adze-fusion cannot win by "speaking MCP better." Differentiation has to come from what's behind the protocol (governance, observability, multi-CAD reach, ribbon UX) — see ADR-001 (forthcoming).
- **Cross-LLM reach is free.** Because MCP is host-agnostic, an adze-fusion MCP server is usable by Claude, Cursor, GPT-class clients, and future hosts — provided the server is well-scoped.

## Open questions

- MCP protocol version drift: the spec evolved across 2024 → 2025 → 2026; compatibility windows for Autodesk-published servers vs community servers are not enumerated in fetched sources. `[unverified]`
- Whether Autodesk's MCP Publisher submission process is currently open to new entrants or restricted to pre-selected partners. `[unverified]`

## Sources

- [Claude for Creative Work — Anthropic news](https://www.anthropic.com/news/claude-for-creative-work) — fetched 2026-05-15
  > "Because the connector is built on MCP, it is accessible to other LLMs in addition to Claude." (claim made about Blender; pattern-true for all MCP-based connectors)
- [Bringing Fusion onto Claude for Creative Work — APS blog](https://aps.autodesk.com/blog/bringing-fusion-claude-creative-work) — fetched 2026-05-15
  > "Fusion Model Context Protocols (MCPs) lets third-party AI systems connect to Fusion, enabling them to access design context and perform actions securely."
- [MCP publisher guide — Autodesk Platform Services](https://aps.autodesk.com/marketplace/mcp-publisher-guide) — fetched 2026-05-15

## Related

- [[../07-patterns/in-process-addin-localhost-mcp]]
- [[../05-mcp-servers/autodesk-fusion-mcp]]
- [[../04-tools/anthropic-fusion-connector]]
- [[../06-communities/autodesk-design-and-make-marketplace]]
- [[../09-sources/anthropic-creative-work-announcement]]
