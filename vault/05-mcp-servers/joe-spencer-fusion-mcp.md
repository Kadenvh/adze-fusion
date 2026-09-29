---
title: Joe-Spencer/fusion-mcp-server
category: 05-mcp-servers
tags: [mcp-server, community, fusion-360, embedded]
status: promoted
confidence: high
license: GPL-3.0
language: Python
maturity: alpha
---

# Joe-Spencer/fusion-mcp-server

Community MCP server that **runs the MCP server inside Fusion's embedded Python** — no sidecar process, no localhost TCP bridge. Trades architectural simplicity for runtime constraint (everything pinned to Fusion's Python 3.7).

## First principles

This is the **other** community pattern: skip the sidecar entirely. The MCP server runs in a background thread inside Fusion's Python interpreter, exposing HTTP SSE for client connections. No external process to launch; no port handshake between two binaries. The cost is that every dependency, every library version, everything runtime-related must work inside Fusion's embedded Python 3.7 — no newer stdlib, no async-features-from-3.10+. The architectural cleanness is real; the runtime ceiling is also real.

## Surface

| Aspect | Detail |
|---|---|
| License | GPL-3.0 (note: copyleft) |
| Stars | 42 |
| Commits | 7 |
| Maturity | Active (more stars than commits — likely shared as reference) |
| Language | Python 3.7 (Fusion-embedded) |
| Architecture | **MCP server runs in background thread inside Fusion's Python**; HTTP SSE transport (no separate sidecar process) |
| Tool surface | 3 tools: `message_box`, `create_new_sketch`, `create_parameter` |
| Platforms | Windows paths in docs only (Mac status unverified) |

## Implications for adze

- **GPL-3.0 is a hard fence.** We cannot study + copy code from this without GPL contaminating adze-fusion. We can read it for architectural ideas; we cannot port code.
- **The "no sidecar" architecture is tempting but constrained.** It simplifies install (one add-in, no second process) but constrains adze-fusion to Fusion's Python 3.7 forever. Any shared core with adze-cad (which uses .NET) would need its bridge anyway, so this saves less than it appears.
- **3 tools is the lower bound of useful surface.** Demonstrates that very small surfaces are also viable — useful counterpoint to faust-machines' 84.
- **Mac is unverified.** Windows-only docs in P1; if Mac path is unsupported, this server doesn't represent the cross-platform reality adze-fusion needs.

## Open questions

- Mac support — Windows-only paths in docs, no Mac install verified.
- Whether GPL-3.0 constrains MCP-protocol-level patterns (almost certainly not — protocol is a contract, not derivative work) but **any code copy is GPL-poisoning**.
- Why HTTP SSE specifically — stdio would be the more conventional in-Fusion choice.

## Sources

- [Joe-Spencer/fusion-mcp-server — GitHub](https://github.com/Joe-Spencer/fusion-mcp-server) — fetched 2026-05-15
  > GPL-3.0; in-Fusion-Python MCP server with HTTP SSE; 3 tools (`message_box`, `create_new_sketch`, `create_parameter`); Windows paths in docs; 42 stars, 7 commits.

## Related

- [[../07-patterns/in-process-addin-localhost-mcp]]
- [[faust-machines-fusion360-mcp]]
- [[sockcymbal-fusion-mcp]]
- [[../03-apis/fusion-adsk-api]]
