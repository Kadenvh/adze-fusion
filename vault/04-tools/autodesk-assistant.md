---
title: Autodesk Assistant
category: 04-tools
tags: [tool, autodesk, ai, assistant, fusion-360, in-product]
status: promoted
confidence: medium
license: proprietary
language: N/A
maturity: production
---

# Autodesk Assistant

Autodesk's in-product agentic AI, native to Fusion (also Revit and Construction Cloud). Writes-and-executes Python against the Fusion API from natural-language prompts and is being opened to certified third-party MCPs as orchestratable tools.

## First principles

Assistant is the **closest first-party analog to "AI does CAD work" rather than "AI talks about CAD."** Its load-bearing primitive is **Script Execute** — the Assistant authors Python against the `adsk.*` API ([[../03-apis/fusion-adsk-api]]) and runs it inside the active Fusion session, producing geometry as a side effect of the conversation, not as a separate code-then-paste step. The forward-looking lever: Autodesk has announced Assistant will be opened to **certified third-party MCPs** published via the [[../06-communities/autodesk-design-and-make-marketplace]], making Assistant the orchestrator and third-party MCPs the tools it calls.

## Surface

| Aspect | Detail |
|---|---|
| Hosts | Fusion 360, Revit, Autodesk Construction Cloud (native, not a plugin) |
| Primary capabilities | Natural-language modeling, Script Execute, image generation (render-style visuals), project/permission admin, manufacturing insights |
| Extensibility model | Certified third-party MCPs callable from Assistant (via Design and Make Marketplace certification) |
| Distribution | Built into Fusion; no separate install |
| Subscription | Active Fusion subscription |

The Script Execute capability is functionally what `[[project-salvador]]` attempted as a third-party LLM wrapper but at runtime-author-and-execute granularity, not paste-the-code granularity. The DEVELOP3D + APS coverage frames Assistant as Autodesk's bet on agentic-AI-as-CAD-feature rather than agentic-AI-as-app-store-plugin.

## Implications for adze

- **Wrapping an LLM for prompt-to-CAD is a dead positioning.** [[project-salvador]] was discontinued precisely because Assistant absorbed the capability. Adze-fusion must offer something Assistant structurally cannot — multi-CAD reach, vendor-neutral governance, observability across CAD hosts, recipe portability — not better prompt-to-sketch.
- **Adze-fusion's MCP server is potentially callable from Assistant.** If certified via the Marketplace, adze-fusion becomes an Assistant tool, not a competitor. This is a real distribution lever — Assistant is the orchestrator, adze provides specialized capability.
- **Assistant's pricing model is opaque.** Whether Assistant orchestrating a third-party MCP triggers any revenue share, throttle, or rate limit is unaddressed in fetched sources — material to adze-fusion's monetization. `[unverified]`

## Open questions

- Is there a published certification cost or revenue share for third-party MCPs that Assistant calls? `[unverified]`
- Does Assistant prompt the user before invoking a third-party MCP, or is it automatic once a tool is enabled?
- Mac vs Windows feature parity on Assistant — fetched sources assert both, but Mac-specific verification is `[unverified]`.

## Sources

- [Autodesk Assistant Enters a New Era — Fusion blog](https://www.autodesk.com/products/fusion-360/blog/autodesk-assistant-enters-a-new-era/) — fetched 2026-05-15 (via search; direct fetch 403)
  > Agentic era framing, Assistant as orchestrator.
- [Autodesk Assistant — Autodesk AI](https://www.autodesk.com/solutions/autodesk-ai/autodesk-assistant) — fetched 2026-05-15 (403, content via search snippets)
  > "Assistant can now write and execute scripts against the Fusion API directly, inside your workflow, in response to plain-language requests"; "Assistant will be open to third-party MCPs and agents."
- [Building for Agentic AI: What's New in Autodesk Platform Services — APS blog](https://aps.autodesk.com/blog/building-agentic-ai-whats-new-autodesk-platform-services) — fetched 2026-05-15

## Related

- [[../02-platforms/fusion-360]]
- [[../03-apis/fusion-adsk-api]]
- [[project-salvador]]
- [[../06-communities/autodesk-design-and-make-marketplace]]
- [[../05-mcp-servers/autodesk-fusion-mcp]]
