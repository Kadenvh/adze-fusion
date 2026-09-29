---
title: In-process add-in + localhost socket + MCP sidecar
category: 07-patterns
tags: [pattern, mcp, fusion-360, integration, architecture]
status: promoted
confidence: high
---

# In-process add-in + localhost socket + MCP sidecar

The dominant pattern for AI-driven CAD operations on Fusion 360. A Python add-in opens a localhost TCP/HTTP listener; an external MCP server process bridges stdio MCP to that port; the add-in marshals work back to Fusion's main thread via `CustomEvent`.

## First principles

CAD hosts have a single-threaded document model (touching document state from a non-UI thread is a crash class) and AI clients want to call into the host via a clean protocol boundary. The cheapest reconciliation: **two processes, one socket, one CustomEvent**. The add-in lives inside Fusion and owns thread-marshaling. The MCP sidecar lives outside Fusion and owns protocol-speaking. Everything Fusion-runtime-sensitive (the Python 3.7 floor, the `adsk.*` namespace, the main-thread requirement) stays inside the add-in. Everything client-facing (MCP protocol versions, transport, future LLM clients) stays in the sidecar, which can iterate independently.

## Implementations using this pattern

| Server | Add-in port | Notes |
|---|---|---|
| [[../05-mcp-servers/autodesk-fusion-mcp]] | `[unverified]` (`27182` from single source) | First-party |
| [[../05-mcp-servers/faust-machines-fusion360-mcp]] | `9876` | 84 tools, MIT, Mac+Windows |
| [[../05-mcp-servers/sockcymbal-fusion-mcp]] | `18080` + `8000` (3-tier with APS) | Hybrid local + cloud |
| [[../05-mcp-servers/joe-spencer-fusion-mcp]] | in-process (no separate sidecar) | GPL-3.0, no localhost TCP — different pattern variant |

## Components

1. **Fusion add-in (Python 3.7)** — registers a `CustomEvent` handler on the main thread; spawns a worker thread that owns a TCP/HTTP listener on a fixed localhost port.
2. **Localhost socket** — between add-in worker thread and MCP sidecar. JSON over TCP/HTTP. No auth needed because loopback-only.
3. **MCP sidecar (external process)** — speaks MCP stdio to the AI client; speaks HTTP/TCP to the add-in. Manifests its tools to the client.
4. **CustomEvent marshal** — worker thread receives a tool call, calls `app.fireCustomEvent(eventId, jsonPayload)`, blocks on a future; main-thread handler runs the actual `adsk.*` API code, returns the result.

## When to use

- Live geometry interaction (live BRep, live timeline edits) — requires Fusion running, single-thread marshaling.
- AI client expects MCP — which they all do as of 2026.
- Cross-platform (Mac + Windows) — the pattern is OS-neutral.

## When not to use

- Fusion not running — use cloud APS / [[../05-mcp-servers/autodesk-fusion-mcp]]'s Fusion Data MCP sibling instead.
- The add-in must run in the same Python interpreter as the MCP server — see Joe-Spencer's variant; simpler install, but pins everything to Fusion's 3.7.

## Implications for adze

- **This is the default architecture posture** if adze-fusion ships any live-geometry tooling.
- **The port choice is a real UX decision.** Conflicts with Anthropic's connector port (`[unverified]`, possibly 27182) or other community servers create install hell. Pick a stable, unique port; document it.
- **The sidecar is where adze-fusion's distinctive logic lives.** The add-in is mostly a Fusion adapter; the sidecar is the agent-facing surface where governance, recipe capture, observability, and multi-CAD bridging live.
- **Three processes (add-in + sidecar + client) is the UX cost.** Match the Anthropic connector's one-port-one-click bar or lose the install race.

## Sources

- [research/findings/P1-fusion-connector-ecosystem.md](../../research/findings/P1-fusion-connector-ecosystem.md) — internal finding
- [faust-machines/fusion360-mcp-server](https://github.com/faust-machines/fusion360-mcp-server) — fetched 2026-05-15 (canonical reference implementation)
- [sockcymbal/autodesk-fusion-mcp-python](https://github.com/sockcymbal/autodesk-fusion-mcp-python) — fetched 2026-05-15 (3-tier variant with APS)

## Related

- [[../01-concepts/mcp-protocol]]
- [[../03-apis/fusion-adsk-api]]
- [[../02-platforms/fusion-360]]
- [[../05-mcp-servers/autodesk-fusion-mcp]]
