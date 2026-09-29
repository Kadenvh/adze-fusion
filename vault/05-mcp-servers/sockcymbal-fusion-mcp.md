---
title: sockcymbal/autodesk-fusion-mcp-python
category: 05-mcp-servers
tags: [mcp-server, community, fusion-360, aps, hybrid]
status: promoted
confidence: high
license: MIT
language: Python
maturity: alpha
---

# sockcymbal/autodesk-fusion-mcp-python

YC MCP Hackathon-era community MCP server. Minimal tool surface (one tool, `generate_cube`) but **architecturally interesting as a 3-tier hybrid that touches both local Fusion and APS cloud** — useful reference for hybrid local+cloud patterns.

## First principles

sockcymbal's value is structural, not functional. Its tool surface is trivial (one generative example) but the underlying stack composes three layers most other community servers don't: a LiveCube add-in on `:18080`, a Python Fusion server on `:8000`, and the MCP stdio bridge — plus an APS OAuth 2.0 client for cloud calls. This makes it the cleanest community example of how a single integration can touch both **local-Fusion** ([[../07-patterns/in-process-addin-localhost-mcp]]) and **cloud-APS** ([[../07-patterns/cloud-rest-aps-oauth]]) — relevant for adze-fusion's hybrid architecture decision.

## Surface

| Aspect | Detail |
|---|---|
| License | MIT |
| Stars | 18 |
| Commits | 5 |
| Origin | YC MCP Hackathon submission (attribution in README) |
| Language | Python |
| Architecture | **3-tier**: LiveCube add-in on `:18080` + `fusion_server.py` on `:8000` + `fusion_mcp.py` stdio MCP — *plus* APS OAuth 2.0 calls (P1's two-tier description omitted port 18080; P0 corrected this) |
| Tool surface | 1 tool: `generate_cube` |
| Platforms | Not explicit in docs |
| Maturity | Alpha / hackathon |

## Implications for adze

- **Reference for hybrid architecture.** The local + cloud composition is exactly what ADR-001 needs to evaluate. Sockcymbal demonstrates the wiring works.
- **Hackathon-tier maintenance.** 5 commits and a hackathon origin suggest no sustained maintenance — useful for study, not as a dependency.
- **APS OAuth 2.0 + local TCP composition matters** beyond this repo. The pattern of "local listener for live geometry, cloud OAuth for data persistence" is the shape adze-fusion likely needs if it wants Fusion-not-running workflows.
- **Three ports is friction.** UX-wise, three ports + OAuth + a third-party app is heavy compared to Anthropic's one-port connector. Hybrid architectures need careful UX work or they lose the install/setup race.

## Open questions

- Whether sockcymbal is actively maintained — 5 commits + hackathon attribution suggest a one-shot.
- Platform support — not explicit in P1's source; needs README re-fetch.
- Whether the APS path uses a publisher's own OAuth client or per-user (relevant to adze-fusion's auth design).

## Sources

- [sockcymbal/autodesk-fusion-mcp-python — GitHub](https://github.com/sockcymbal/autodesk-fusion-mcp-python) — fetched 2026-05-15
  > MIT; 3-tier (LiveCube add-in on :18080 + fusion_server.py on :8000 + fusion_mcp.py stdio); APS OAuth 2.0; `generate_cube` tool; YC MCP Hackathon attribution.

## Related

- [[../07-patterns/in-process-addin-localhost-mcp]]
- [[faust-machines-fusion360-mcp]]
- [[joe-spencer-fusion-mcp]]
- [[../02-platforms/fusion-360]]
