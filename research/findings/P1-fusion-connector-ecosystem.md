# P1 — Fusion 360 Connector + MCP + AI Integration Ecosystem

**Date:** 2026-05-15
**Researcher:** general-purpose sub-agent

## Summary

The Fusion 360 AI/MCP ecosystem matured dramatically in April 2026. Anthropic shipped an official Claude Fusion connector on 2026-04-28 as part of "Claude for Creative Work," built on MCP and authored jointly with Autodesk. Autodesk itself shipped a first-party stack of MCP servers (Fusion MCP, Fusion Data MCP, Revit MCP, Product Help MCP — all tech preview; Fusion Automation MCP in private beta) and announced that its in-product **Autodesk Assistant** will orchestrate certified third-party MCPs published via the Design and Make Marketplace under an ISO 42001-aligned certification regime. A vibrant GitHub community (10+ independent Fusion MCP servers, ranging from 84-tool feature-rich to 2-tool minimal) predates and runs alongside the official path. Backflip's mesh-to-CAD Fusion plugin remains roadmap-only — Onshape and SOLIDWORKS shipped first.

## First principles

Fusion 360's integration surfaces, stripped of marketing:

1. **Python/C++ add-ins inside Fusion's process** — the native API (`adsk.*`) runs cross-platform (Windows + macOS Intel + macOS ARM) with Python 3.7. This is the local equivalent of adze-cad's COM add-in but cross-platform out of the box.
2. **TCP/HTTP localhost from add-in** — the entrenched pattern for AI bridges: a Python add-in opens a localhost socket (ports observed: 9876, 18080, 8765, 7654, and now 27182 for Autodesk's official). An external MCP sidecar process speaks stdio MCP to the AI client and HTTP to the add-in.
3. **Autodesk Platform Services (APS, formerly Forge)** — cloud REST APIs with OAuth 2.0 for design automation and data management. Used when Fusion does not need to be running locally (Fusion Data MCP is cloud-only).
4. **MCP as the new lingua franca** — both Autodesk and Anthropic now treat MCP (stdio or HTTP) as the canonical AI-integration boundary.
5. **Autodesk Assistant** — an in-product agentic AI orchestrator that already writes-and-executes Fusion API Python at runtime ("Script Execute") and is being opened to certified third-party MCPs.

## Findings

### Anthropic Fusion 360 Connector

**Confirmed.** Anthropic launched the Autodesk Fusion connector on **2026-04-28** as one of 9 creative-work connectors (alongside Blender, SketchUp, Adobe, Ableton, Affinity, Resolume Arena/Wire, Splice). It is built on MCP — Autodesk authored the Fusion MCP, Anthropic integrated it into Claude. Setup: enable MCP server in Fusion 360 → Preferences → General → API; default port **27182**; in Claude, add the Fusion connector with that address and port. Requires an active Fusion subscription. Capabilities include reading geometry, modifying parts, running Fusion API operations, and writing parametric history to the active file. Because it is MCP-based, "the connector is accessible to other LLMs in addition to Claude." [Sources: Anthropic news; APS blog; knightli.com hands-on]

### Community MCP servers

A non-exhaustive census of independent GitHub Fusion MCP servers as of 2026-05-15:

| Repo | Architecture | Tools | License | Platforms | Status |
|------|--------------|-------|---------|-----------|--------|
| `sockcymbal/autodesk-fusion-mcp-python` | Three-tier: add-in (LiveCube) + Python server on :8000 + MCP stdio; also calls APS via OAuth 2.0 | 1 (`generate_cube`) | MIT | Not specified | YC MCP Hackathon, 18 stars, 5 commits, active |
| `faust-machines/fusion360-mcp-server` | Python MCP ↔ TCP :9876 ↔ Fusion add-in (CustomEvent to main thread) | **84 tools** across 17 categories (sketch, features, body, CAM, export, params, inspection) | MIT | Mac + Windows | Beta, 11 commits, active |
| `Joe-Spencer/fusion-mcp-server` | Add-in runs MCP server in background thread inside Fusion's Python; HTTP SSE | 3 (`message_box`, `create_new_sketch`, `create_parameter`) | GPL-3.0 | Windows (paths only) | Active, 42 stars, 7 commits |
| `ndoo/fusion360-mcp-bridge` | Python MCP ↔ HTTP ↔ Fusion add-in; arbitrary script execution + viewport screenshot | 2 (`fusion_execute`, `fusion_screenshot`) | MIT | Mac + Windows | Early, 1 commit, 7 stars |
| `AuraFriday/Fusion-360-MCP-Server` | Add-in connecting to remote Aura Friday MCP-Link cloud broker | Not enumerated | Not enumerated | Not enumerated | Listed in search results |
| `Misterbra/fusion360-claude-ultimate` | French-localized fork of Kanbara Tomonori's concept | Not enumerated | Not enumerated | Not enumerated | Listed |
| `ArchimedesCrypto/fusion360-mcp-server` | Listed in search results | — | — | — | Listed |
| `zkbkb/fusion-mcp` | Listed in search results | — | — | — | Listed |
| `JustusBraitinger/Autodesk-Fusion-360-MCP-Server` | Listed (lobehub mirror) | — | — | — | Listed |
| `frankhommers/autodesk-fusion-mcp` | Listed in search results | — | — | — | Listed |

Note on `sockcymbal`: confirmed real; the user's prior reference is accurate. Minimal surface (just `generate_cube`), but it is one of the few that integrates **both** local Fusion and cloud APS — useful as a reference for hybrid architectures.

### Autodesk Platform Services + first-party MCP

Autodesk shipped a public MCP suite in 2026 (announcement: APS blog 2026-04-15):

- **Fusion MCP** (tech preview) — "create, modify, and inspect 3D geometry"; runs locally while Fusion is active; speaks to Claude Desktop, Cursor, or any MCP HTTP client.
- **Fusion Data MCP** (tech preview) — cloud, operates without Fusion running; queries/manages design data via APS.
- **Revit MCP** (tech preview) — model inspection.
- **Product Help MCP** (tech preview) — read-only access to Autodesk Help across 110+ products.
- **Fusion Automation MCP** (private beta, application required) — remote workflow execution.
- **InfoWorks Hydraulic Modeling MCP** (coming soon).

There is also an MCP **Publisher Guide** at `aps.autodesk.com/marketplace/mcp-publisher-guide` describing a 4-step process: create an MCP Tool Manifest (JSON declaring tools, resources, prompts, external connections, APS APIs, AI providers used), complete a Publisher Declaration with security attestations, submit to `appsubmissions@autodesk.com`, publish and monitor. Stdio transport is the example; HTTPS is required for any external connections. No fees mentioned. Submissions feed the Design and Make Marketplace and become eligible to be called by Autodesk Assistant. Certification aligns to ISO 42001, NIST AI RMF, and IEEE CertifAIEd.

### Autodesk Assistant

Autodesk Assistant is Autodesk's in-product agentic AI (Fusion, Revit, Construction Cloud). Capabilities now include:
- **Script Execute** — Assistant writes Python against the Fusion API and runs it in-workflow without user-written code.
- **Image generation** — render-style visuals from designs.
- Natural-language modeling commands, project/permission management, manufacturing insights.
- Will be **opened to third-party MCPs** — certified solutions on the Design and Make Marketplace become callable directly from Assistant.

This is the closest analog to "AI doing CAD work" rather than chatting about it that Autodesk ships natively, and it parallels the founder's adze-cad "embedded, not chat" framing.

### App Store AI plugins

The App Store AI-specific surface is sparse compared to the MCP ecosystem:

- **Project Salvador (Beta)** — Autodesk's own Fusion plug-in, chains GPT-4 Turbo + DALL-E/Stable Diffusion + Vectorizer AI for text-to-image, image-to-image, and text-to-sketch. **Discontinued.** Per the Fusion blog: "Project Salvador is not currently being developed and is not planned to be superseded by Autodesk Assistant." Effectively absorbed back into Assistant.
- **MCP Server for Autodesk Fusion** — appears on the App Store with Mac variant (search result `apps.autodesk.com/FUSION/en/Detail/Index?id=7269770001970905100&os=Mac`). Likely the official Autodesk MCP add-in installer.
- Generative Design and AutoConstrain are native Autodesk features, not add-ins.

The App Store has not become a major AI-distribution channel; the action is in MCP servers and the Marketplace certification pipeline.

### Backflip

Backflip's Mesh-to-CAD product is cloud-GPU based. Architecture: user uploads STL → cloud generates 4 parametric variations from a model trained on 100M+ synthetic geometries → CAD-host plugin rebuilds with a native feature tree.

Current supported hosts (2026-05): **Onshape** (sidebar plugin) and **SOLIDWORKS** (side-panel plugin, beta). Fusion 360 is **roadmap, not shipped** — listed alongside Siemens NX, PTC Creo as "future plugins coming." Standalone web app issues STEP files as a stopgap. Freemium model planned (Onshape-style: free public, paid private). Founder: Greg Mark (Markforged founder).

### Integration architecture patterns

Three patterns emerge:

1. **In-process add-in + localhost socket + MCP sidecar** — dominant. Fusion add-in (Python) exposes HTTP/TCP on a fixed localhost port; an external MCP server process bridges stdio MCP to that port. Examples: Anthropic official (port 27182), `faust-machines` (:9876), `ndoo` (HTTP), `sockcymbal` (:8000 + :18080). This is the canonical adze-fusion-shaped pattern.
2. **Cloud REST via APS / OAuth 2.0** — used when local Fusion is not required. Fusion Data MCP and parts of `sockcymbal` use this. Required for any "design data" or admin surface.
3. **Add-in-hosts-MCP-server-directly** — server runs inside Fusion's Python (`Joe-Spencer`). Eliminates the sidecar but constrains tooling to Fusion's Python 3.7 and runtime.
4. **Cloud GPU + plugin** (Backflip pattern) — UI plugin is thin; compute is cloud. Mac/Windows parity is automatic since the add-in is mostly a shell.

### Mac vs Windows reality

| Integration | Windows | Mac |
|-------------|---------|-----|
| Fusion Python API itself | Yes | Yes (Intel + ARM) |
| Anthropic Claude Fusion connector | Yes | Yes (MCP is OS-neutral; Fusion runs on Mac) |
| Autodesk official Fusion MCP | Yes | Yes (App Store listing has Mac variant) |
| `faust-machines/fusion360-mcp-server` | Yes | Yes (explicit Mac install docs) |
| `ndoo/fusion360-mcp-bridge` | Yes | Yes (explicit Mac docs) |
| `Joe-Spencer/fusion-mcp-server` | Yes | Unknown (Windows paths only in docs) |
| `sockcymbal` | Likely both | Not explicit |
| Backflip Fusion plugin | Not shipped | Not shipped |
| Autodesk Assistant | Yes | Yes (native to Fusion) |

**Adze-cad's Windows-only COM add-in posture does not transfer.** Fusion is genuinely cross-platform; adze-fusion must be Mac + Windows from day one or it competes against community MCPs that already ship both.

### Roadmap signals

- **2026-04-15** APS blog announces the public MCP server suite and "agentic AI" platform direction.
- **2026-04-28** Anthropic Claude for Creative Work ships 9 connectors including Fusion.
- **Autodesk DevCon 2026** keynote highlights MCP servers + Design and Make Marketplace.
- **Neural CAD** — Autodesk roadmap item: generative AI model trained specifically for 3D CAD that emits editable BREP from text prompts. Future, not shipped.
- **Synera connector** — Autodesk has shipped a Fusion connector for Synera (agentic AI design optimization), signaling more first-party agent integrations.

## Decision implications for adze-fusion

- **MCP-first is the obvious architecture.** Both Anthropic and Autodesk have standardized on MCP; the publishing pipeline, the connector model, the third-party orchestration via Autodesk Assistant — all assume MCP. A non-MCP architecture would be swimming upstream.
- **The market is crowded above and below.** Anthropic's official connector covers the "Claude → Fusion" use case directly. Autodesk's Fusion MCP + Assistant cover the in-product agentic case. 10+ open-source MCPs exist. The "general Fusion-MCP add-in" space is saturated. Adze-fusion must differentiate on **governance, multi-provider, observability, learning, or a non-MCP surface (ribbon-direct UI like Phase 11 of adze-cad)** — not on "connect Fusion to Claude."
- **Cross-platform from day one is non-negotiable.** Fusion is genuinely cross-platform (Win + Intel Mac + ARM Mac). Mac users are a real cohort. Python add-in is the path; .NET COM is not.
- **Distribution paths are clearer than SOLIDWORKS.** App Store + Design and Make Marketplace certification (ISO 42001, NIST AI RMF, IEEE CertifAIEd) gives a defined path; submission is free; Autodesk Assistant integration is a real distribution lever.
- **Local + cloud is a real split.** Fusion Data MCP (cloud, no Fusion needed) and Fusion MCP (local, Fusion running) are separate products. Adze-fusion should decide which side or both; the answer affects auth (APS OAuth 2.0 vs none) and offline behavior.
- **Anthropic's connector raises the bar on UX.** Default port 27182, single-click connection, native Claude UI — any third-party tool must match or exceed that UX or it has no reason to exist.
- **Project Salvador's discontinuation is a warning.** Third-party "wrap-an-LLM" Fusion plugins lose to native Autodesk Assistant the moment Assistant absorbs the same capability. Adze-fusion must be deeper than a prompt-to-sketch wrapper.
- **Backflip's absence from Fusion is an opening, but a narrow one.** Mesh-to-CAD on Fusion is unclaimed; competing on that single vertical against a roadmap-promised Backflip is risky but viable for ~12 months.

## Open questions

- Exact ports, install location, and tool manifest of Autodesk's first-party Fusion MCP add-in (the Autodesk dev forum thread and Autodesk Assistant blog returned 403).
- Whether Autodesk's MCP Publisher submission process is currently open to new entrants or restricted to pre-selected partners.
- Pricing/licensing model for being callable from Autodesk Assistant — is there a revenue share via the Marketplace?
- Whether the Anthropic Fusion connector is OS-restricted in practice (theoretically Mac works because Fusion runs on Mac, but no Mac install was verified end-to-end in fetched sources).
- Last-commit dates on the community MCP repos (GitHub UI is JS-rendered; only commit counts were extractable).
- Whether Project Salvador's discontinuation note is recent or refers to its original beta sunset — relevant to whether the App Store still permits LLM-wrapper add-ins.
- Whether `sockcymbal` is actively maintained or was a one-shot hackathon submission (5 commits, hackathon attribution suggest the latter).

## Sources

- [Bringing Fusion onto Claude for Creative Work — Autodesk Platform Services blog](https://aps.autodesk.com/blog/bringing-fusion-claude-creative-work) — fetched 2026-05-15 — "Fusion Model Context Protocols (MCPs) lets third-party AI systems connect to Fusion, enabling them to access design context and perform actions securely"; "Autodesk Fusion MCP connects Claude directly to the Fusion environment for Fusion subscribers."
- [Claude for Creative Work — Anthropic news](https://www.anthropic.com/news/claude-for-creative-work) — fetched 2026-05-15 — Launch 2026-04-28; 9 connectors (Ableton, Adobe, Affinity, Autodesk Fusion, Blender, Resolume Arena, Resolume Wire, SketchUp, Splice); "because the connector is built on MCP, it is accessible to other LLMs in addition to Claude."
- [Connecting Claude to Fusion 360: An Example of Editing STEP Models With AI — knightli.com](https://knightli.com/en/2026/05/14/claude-fusion-360-mcp-step-model-edit/) — fetched 2026-05-15 — "Enable the MCP server" in Fusion → Preferences → General → API; "the default port `27182`"; hands-on gear-modification walkthrough; tolerance limitations.
- [Building for Agentic AI: What's New in Autodesk Platform Services — APS blog](https://aps.autodesk.com/blog/building-agentic-ai-whats-new-autodesk-platform-services) — fetched 2026-05-15 — Lists Fusion MCP, Fusion Data MCP, Revit MCP, Product Help MCP (all tech preview), Fusion Automation MCP (private beta), InfoWorks Hydraulic (coming soon); "building a path for certified third-party MCPs published on the Design and Make Marketplace to be callable directly inside Autodesk Assistant."
- [Autodesk MCP Servers — Autodesk AI](https://www.autodesk.com/solutions/autodesk-ai/autodesk-mcp-servers) — fetched 2026-05-15 (403, content derived from search snippets) — Fusion MCP compatible with Claude Desktop, Cursor, "any MCP-capable HTTP client."
- [MCP publisher guide — Autodesk Platform Services](https://aps.autodesk.com/marketplace/mcp-publisher-guide) — fetched 2026-05-15 — "(1) Create the MCP Tool Manifest (2) Complete the Publisher Declaration (3) Submit for approval (4) Publish and monitor"; stdio transport example; HTTPS required for external connections; submit to `appsubmissions@autodesk.com`.
- [Autodesk announces Fusion MCP servers and more AI updates — Engineering.com](https://www.engineering.com/autodesk-announces-fusion-mcp-servers-and-more-ai-updates/) — fetched 2026-05-15 — Local Fusion MCP + cloud Fusion Data MCP; "connect Fusion to their internal systems, automate multi-step engineering workflows, or query and reuse design data across projects, all powered by AI agents."
- [Design and Make Marketplace — FifthRow analysis](https://www.fifthrow.com/blog/from-bottleneck-to-backbone-how-autodesk-s-agentic-ai-design-and-make-marketplace-is-transforming-manufacturing-tech-transfer) — fetched 2026-05-15 — ISO 42001-aligned certification, NIST AI RMF, IEEE CertifAIEd; Publisher Center as gatekeeper.
- [Autodesk Assistant — Autodesk AI](https://www.autodesk.com/solutions/autodesk-ai/autodesk-assistant) — fetched 2026-05-15 (403, content derived from search snippets) — "Assistant can now write and execute scripts against the Fusion API directly, inside your workflow, in response to plain-language requests"; "Assistant will be open to third-party MCPs and agents."
- [Introducing Project Salvador for Autodesk Fusion — Fusion blog](https://www.autodesk.com/products/fusion-360/blog/project-salvador-autodesk-fusion-app-store/) — fetched 2026-05-15 (via search) — "Project Salvador is not currently being developed and is not planned to be superseded by Autodesk Assistant"; GPT-4 Turbo + DALL-E + Stable Diffusion + Vectorizer AI.
- [sockcymbal/autodesk-fusion-mcp-python — GitHub](https://github.com/sockcymbal/autodesk-fusion-mcp-python) — fetched 2026-05-15 — MIT license; three-tier (LiveCube add-in + fusion_server.py on :8000 + fusion_mcp.py stdio); calls APS OAuth 2.0; `generate_cube` tool; YC MCP Hackathon attribution; 18 stars, 5 commits.
- [faust-machines/fusion360-mcp-server — GitHub](https://github.com/faust-machines/fusion360-mcp-server) — fetched 2026-05-15 — MIT; Python MCP ↔ TCP :9876 ↔ Fusion add-in (CustomEvent); 84 tools across 17 categories; Mac + Windows install docs; beta, 11 commits.
- [Joe-Spencer/fusion-mcp-server — GitHub](https://github.com/Joe-Spencer/fusion-mcp-server) — fetched 2026-05-15 — GPL-3.0; in-Fusion-Python MCP server with HTTP SSE; 3 tools (`message_box`, `create_new_sketch`, `create_parameter`); Windows paths in docs; 42 stars, 7 commits.
- [ndoo/fusion360-mcp-bridge — GitHub](https://github.com/ndoo/fusion360-mcp-bridge) — fetched 2026-05-15 — MIT; Python MCP ↔ HTTP ↔ Fusion add-in; 2 tools (`fusion_execute`, `fusion_screenshot`); explicit Mac + Windows install; 7 stars, 1 commit.
- [Backflip Mesh-to-CAD — backflip.ai](https://www.backflip.ai/mesh-to-cad) — fetched 2026-05-15 — Onshape plugin (sidebar), web app; SOLIDWORKS and Fusion 360 not on current page; "currently in active beta"; waitlist.
- [Backflip's new AI-based plug-in for SolidWorks — mechnexus.com (via search)](https://mechnexus.com/backflips-new-ai-based-plug-in-for-solidworks/) — fetched 2026-05-15 — SOLIDWORKS side-panel plugin in beta; cloud GPU; 4 parametric variations; Fusion + NX + Creo "future plugins."
- [Backflip Demo Showcases Scan-to-CAD's Revolutionary Capabilities — 3DPrint.com (via search)](https://3dprint.com/317148/backflip-demo-showcases-scan-to-cads-revolutionary-capabilities/) — fetched 2026-05-15 — 100M+ synthetic geometry training set; freemium roadmap.
- [Fusion Help — Python Specific Issues](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/PythonSpecific_UM.htm) — fetched 2026-05-15 (via search) — Fusion uses Python 3.7; cross-platform Mac (Intel + ARM) and Windows.
- [Plugin, Add-on, Extension for Fusion — Autodesk App Store](https://apps.autodesk.com/FUSION/en/List/Search) — fetched 2026-05-15 (via search) — App Store catalog entry point.
- [MCP Server for Autodesk Fusion — Autodesk App Store (Mac variant)](https://apps.autodesk.com/FUSION/en/Detail/Index?id=7269770001970905100&ln=en&os=Mac) — fetched 2026-05-15 (via search) — Confirms official MCP add-in has Mac distribution.
- [Driving Fusion via AI using MCP server add-in (announcement) — Autodesk Community forum (via search)](https://forums.autodesk.com/t5/fusion-api-and-scripts-forum/driving-fusion-via-ai-using-mcp-server-add-in-announcement/td-p/13881165) — fetched 2026-05-15 (via search) — Official Autodesk forum thread for MCP add-in announcement; full content gated behind 403.
- [Autodesk Assistant Enters a New Era — Fusion blog](https://www.autodesk.com/products/fusion-360/blog/autodesk-assistant-enters-a-new-era/) — fetched 2026-05-15 (via search; direct fetch 403) — Agentic era framing, Assistant as orchestrator.
- [How Autodesk shaped MCP for enterprises — Autodesk News](https://adsknews.autodesk.com/en/views/how-autodesk-helped-make-the-model-context-protocol-enterprise-ready/) — fetched 2026-05-15 (via search) — Autodesk's role in MCP enterprise readiness.
- [Autodesk launches Product Help MCP Server — Autodesk News](https://adsknews.autodesk.com/en/news/product-help-mcp-server/) — fetched 2026-05-15 (via search) — Product Help MCP covers 110+ products.
- [Autodesk DevCon 2026 Highlights — APS blog](https://aps.autodesk.com/blog/autodesk-devcon-2026-highlights) — fetched 2026-05-15 (via search) — DevCon MCP announcements timing.
- [Autodesk Fusion Connector for Synera — Fusion blog](https://www.autodesk.com/products/fusion-360/blog/autodesk-fusion-connector-for-synera/) — fetched 2026-05-15 (via search) — Example of Autodesk-shipped first-party agentic connector.
- [Claude for CAD arrives with Blender and Autodesk Fusion connectors — DEVELOP3D](https://develop3d.com/ai/claude-for-cad-blender-autodesk-fusion/) — fetched 2026-05-15 (via search) — Industry press confirmation of launch.
- [Anthropic releases 9 Claude connectors — 9to5Mac](https://9to5mac.com/2026/04/28/anthropic-releases-9-new-claude-connectors-for-creative-tools-including-blender-and-adobe/) — fetched 2026-05-15 (via search) — Launch date 2026-04-28 confirmed.
