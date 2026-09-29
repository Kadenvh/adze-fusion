---
title: faust-machines/fusion360-mcp-server
category: 05-mcp-servers
tags: [mcp-server, community, fusion-360, open-source]
status: promoted
confidence: high
license: MIT
language: Python
maturity: beta
---

# faust-machines/fusion360-mcp-server

The largest-surface community MCP server for Fusion 360. **84 tools across 17 categories** (sketch, features, body, CAM, export, parameters, inspection). MIT, cross-platform install docs, the canonical reference implementation of the in-process-add-in + localhost-TCP + MCP-sidecar pattern ([[../07-patterns/in-process-addin-localhost-mcp]]).

## First principles

faust-machines is the **upper bound on what one community-built MCP server has shipped for Fusion**. It is interesting less as a candidate dependency (we will not run a third-party MCP server as part of adze-fusion) and more as: (a) a working reference for the worker-thread → `CustomEvent` → main-thread pattern, (b) a yardstick for "how many tools is sensible," and (c) the prior art that adze-fusion's MCP surface (if it ships one) is implicitly compared against by users.

## Surface

| Aspect | Detail |
|---|---|
| License | MIT |
| Stars | 32 (P0-verified count; P1's table omitted a stars column) |
| Commits | 11 |
| Maturity | Beta |
| Language | Python |
| Architecture | Python MCP server ↔ TCP `:9876` ↔ Fusion add-in (CustomEvent to main thread) |
| Platforms | Mac + Windows (explicit install docs for both) |
| Tool surface | 84 tools / 17 categories — sketch, features, body, CAM, export, parameters, inspection |

The 3-tier separation (MCP stdio process ↔ TCP bridge ↔ Fusion add-in) is the dominant architecture in P1's community survey and the one the Autodesk first-party server [[autodesk-fusion-mcp]] also uses.

## Implications for adze

- **Pattern reference, not dependency.** Adze-fusion's add-in side can crib the `CustomEvent` marshaling and TCP-listener-in-worker patterns from this codebase.
- **84 tools is a lot.** It implies a thin-tool surface (one tool per Fusion API operation) rather than a coarse-tool surface (composite intents). Adze-fusion needs to decide which posture to take — see ADR-001.
- **MIT license** means we can study and adapt without contamination concerns.
- **Beta + 11 commits + 32 stars** signals a hobbyist project, not a sustained one. Expect community-MCP turnover.

## Open questions

- Activity status — 11 commits is small; whether faust-machines is actively maintained or one-shot is not clear from fetched data.
- Whether tools are coarse (one per intent) or fine (one per API call). Tool list breakdown wasn't enumerated in P1.
- Performance — latency under load, tool-result size limits. Not addressed.

## Sources

- [faust-machines/fusion360-mcp-server — GitHub](https://github.com/faust-machines/fusion360-mcp-server) — fetched 2026-05-15
  > "84 tools across 17 categories"; MIT; explicit Mac + Windows install; TCP :9876.

## Related

- [[../07-patterns/in-process-addin-localhost-mcp]]
- [[../03-apis/fusion-adsk-api]]
- [[autodesk-fusion-mcp]]
- [[sockcymbal-fusion-mcp]]
- [[joe-spencer-fusion-mcp]]
