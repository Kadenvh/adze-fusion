# W1-3 — Anthropic Fusion connector deep-dive: capabilities, security model, licensing, extensibility

**Date:** 2026-06-11
**Researcher:** workflow sub-agent (Stage 1 Wave 1)

## Summary (3-5 sentences)

The port question is settled: **27182 is officially documented by Autodesk as the default port**, in both the Autodesk MCP Server Help ("The default port is `27182`") and a May 2026 Autodesk support article — the knightli.com-only sourcing is obsolete. **Fusion for personal use is officially excluded**: Autodesk support states third-party AI tools "cannot be integrated into Autodesk Fusion with a personal use license." The connector is a **local-only, unauthenticated HTTP MCP endpoint (`http://127.0.0.1:27182/mcp`) built into Fusion itself** (preferences checkbox, not an App Store add-in — the old App Store listing now 404s), consumed in Claude via a Claude Desktop extension. The tool surface is officially **dynamic** — tools are discovered at connection time and Autodesk publishes no enumeration. No Claude-plan gating is documented on Anthropic's side; desktop extensions are available to all Claude Desktop users.

## First principles (what the thing IS, stripped of marketing)

The "Anthropic Fusion connector" is two pieces, neither exotic:

1. **Autodesk's side:** an MCP server hosted *inside* the Fusion desktop process, enabled by a checkbox at Preferences > General > API ("Fusion MCP Server — runs locally on this device"). When enabled, Fusion listens on localhost HTTP (`http://127.0.0.1:27182/mcp`, port configurable). It is local-only, desktop-only (not the Fusion web client), session-based (acts on the live open document), and exposes a dynamically discovered tool set. No authentication. ([Autodesk Fusion MCP Server help](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_autodesk_fusion_mcp_server_html) — fetched 2026-06-11: "Local-only. Runs on the same machine as Fusion — no remote connections or authentication required.")
2. **Anthropic's side:** a Claude Desktop **extension** (Settings > Extensions > Fusion) that points Claude Desktop's MCP client at that localhost URL. It is an ordinary MCP client connection with Claude Desktop's standard per-tool permission UI. ([Connecting to the Autodesk Fusion MCP Server](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_connecting_to_the_fusion_mcp_server_html) — fetched 2026-06-11)

Because the protocol is open MCP over localhost HTTP, any MCP client (Cursor, VS Code, custom clients) can connect the same way — "Claude connector" is branding for one client of a generic Autodesk-owned endpoint.

## Findings

### 1. The port question — SETTLED: 27182 is officially documented

Two primary Autodesk sources document the default port:

- [Autodesk MCP Server Help — Connecting to the Autodesk Fusion MCP Server](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_connecting_to_the_fusion_mcp_server_html) — fetched 2026-06-11: "**The default port is `27182`.** If you change the port in Fusion preferences, update the URL in your client configuration to match."
- [Autodesk Support: Third-Party AI Tools Fail to Connect to Autodesk Fusion MCP](https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/Claude-MCP-connector-failing-to-connect-to-Autodesk-Fusion.html) (dated May 5, 2026) — fetched 2026-06-11: "If **Port 27182** is blocked, choose another port. Note: This will also need changing to match in the Third party AI tool."
- The full endpoint URL is documented in [Troubleshooting](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_troubleshooting_html) — fetched 2026-06-11: "Verify your client is configured with `http://127.0.0.1:27182/mcp`."

The App Store listing is no longer a usable source: `apps.autodesk.com/FUSION/en/Detail/Index?id=7269770001970905100` (both Mac and Win64 variants) returns **404 "Page not found"** on the renamed Design and Make Marketplace [empirical, fetched 2026-06-11]. Consistent with this, the help docs treat the MCP server as **built into Fusion** ("Update Fusion to the latest version. The Autodesk Fusion MCP Server option is only available in recent releases" — Troubleshooting page), not as an installable add-in. (Incidentally, 27182 ≈ the digits of *e*; observation only.)

### 2. Fusion subscription tiers — personal use is excluded

[Autodesk Support article](https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/Claude-MCP-connector-failing-to-connect-to-Autodesk-Fusion.html) — fetched 2026-06-11, listed as a *cause* of connection failure: "**Third-Party AI Tools cannot be integrated into Autodesk Fusion with a personal use license.**" The prescribed solution: "Switch to a commercial Fusion Subscription. Upgrade you[r] personal Fusion license to a commercial Fusion Subscription." The [APS blog](https://aps.autodesk.com/blog/bringing-fusion-claude-creative-work) — fetched 2026-06-11 — frames access as "for Fusion subscribers." **The free hobbyist tier does NOT get MCP access.** Education / startup tier status: could not determine — no primary source addresses them.

### 3. Claude plan availability — no plan gating documented

There is **no Fusion-specific article on support.claude.com** (searched 2026-06-11, two query variants — only generic connector articles surface). Primary-source picture:

- [Anthropic, Claude for Creative Work](https://www.anthropic.com/news/claude-for-creative-work) — fetched 2026-06-11 — conditions access only on the Autodesk side: "allows designers and engineers **with a Fusion subscription** to create and modify 3D models through conversations with Claude." No Claude plan named.
- [Use connectors to extend Claude's capabilities](https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities) — fetched 2026-06-11: "**Desktop extensions are available to all users on Claude Desktop.**" The Fusion connector is delivered as a desktop extension (Autodesk's walkthrough: Claude Desktop > Settings > Extensions > install Fusion).

Inference (flagged): the connector is available on **all Claude plans including Free**, since desktop extensions are not plan-gated — but no source states "Fusion connector works on Free" explicitly [unverified]. Practical constraint that IS sourced: the server is local-only, and claude.ai custom connectors connect "from Anthropic's cloud infrastructure, rather than from your local device" ([Get started with custom connectors using remote MCP](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp) — fetched 2026-06-11), so **Claude Desktop is effectively required**; claude.ai web cannot reach `127.0.0.1`.

### 4. Security / data-flow model

- **Transport boundary:** "Local-only. Runs on the same machine as Fusion — **no remote connections or authentication required**" ([Fusion MCP Server help](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_autodesk_fusion_mcp_server_html), fetched 2026-06-11). Corollary [inference, unverified]: the localhost endpoint is unauthenticated, so any local process could connect while enabled; the off-by-default preferences checkbox is the primary gate.
- **Server-side posture:** "Regardless of security configuration, MCP servers remain responsible for validating requests, enforcing defined execution boundaries, and ensuring predictable behavior" and they "execute only what is explicitly requested by an AI application" ([About Autodesk MCP Servers](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_CommonContent_about_autodesk_mcp_servers_html) — fetched 2026-06-11).
- **Autodesk's user-control statement:** "You stay in control of what's accessed and how it's used, with protections aligned to Autodesk's privacy, security, and product standards" ([APS blog](https://aps.autodesk.com/blog/bringing-fusion-claude-creative-work) — fetched 2026-06-11).
- **Consent gates on the Claude side:** per-tool permissions — Autodesk's walkthrough verifies connection via Claude Desktop's "**Tool permissions** list" under the Fusion extension config ([Connecting page](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_connecting_to_the_fusion_mcp_server_html)). Anthropic's general connector model: "Claude inherits each person's permissions from the connected service" ([connectors article](https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities)).
- **What egresses to Anthropic:** no primary source enumerates this. By MCP mechanics, geometry/feature data returned by tool calls enters the conversation context and is processed on Anthropic infrastructure under the user's Claude data settings [inference, unverified]. A third-party security vendor maintains a risk profile for this connector ([PromptArmor — Autodesk Fusion Connector Risk](https://www.promptarmor.com/connectors/autodesk-fusion) — fetched 2026-06-11, community), confirming the directory description and requirements but the full report is paywalled.

### 5. Tool surface — officially dynamic, no published enumeration

"**Dynamic tooling.** Available tools are discovered by your MCP client at connection time and may change as capabilities evolve" ([Fusion MCP Server help](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_autodesk_fusion_mcp_server_html) — fetched 2026-06-11). The help home contrasts Product Help MCP ("2 Tools") with Fusion MCP ("Dynamic Tools"). Functional categories per primary sources: "perform real-time modeling, and execute command-based operations" against the live session (same page); "create, modify, and query design geometry and features inside Fusion" ([Claude for Autodesk Fusion campaign page](https://www.autodesk.com/campaigns/fusion-360/claude-fusion) — fetched 2026-06-11 via Tavily; direct fetch 403). The Connectors Directory description is "Create, modify, and inspect CAD geometry in Fusion" (via PromptArmor mirror, community). **Exact tool count: could not determine** — no official enumeration exists, and no credible community dump of the official server's tool list was found; the official docs defer to the [Autodesk Assistant capabilities page](https://help.autodesk.com/view/fusion360/ENU/?guid=AA_Capabilities) for supported workflows.

### 6. Extensibility — coexistence undocumented but structurally unobstructed

No primary source documents conflicts or composition patterns for running a third-party Fusion MCP server alongside the official one. What IS documented: Claude Desktop supports multiple simultaneously installed extensions and custom local MCP servers ([Getting Started with Local MCP Servers on Claude Desktop](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop) — fetched 2026-06-11). Community Fusion MCP servers bind distinct ports (frankhommers `127.0.0.1:8765/mcp`, ndoo :7654, faust-machines :9876 — GitHub/LobeHub, fetched 2026-06-11, community), so no port collision with 27182. Residual conflict surface [inference, unverified]: overlapping tool names/semantics confusing the model when two Fusion-control servers are active, and contention for the single live Fusion session. No empirical test of simultaneous operation was found.

### 7. Mac — no official end-to-end Mac walkthrough

Could not find an official Mac-specific install walkthrough. The Autodesk help docs are **OS-neutral** (same Preferences > General > API path; same Claude Desktop Settings > Extensions flow), and Anthropic documents Claude Desktop for macOS ([Deploy Claude Desktop for macOS](https://support.claude.com/en/articles/12611117-deploy-claude-desktop-for-macos), linked from the local-MCP article — fetched 2026-06-11). The former App Store Mac listing (the prior pass's Mac evidence) now 404s [empirical, 2026-06-11]; since the server ships inside Fusion and Fusion runs on macOS, the built-in path presumably works identically on Mac [inference, unverified — no primary source confirms or denies Mac-specific limitations].

### 8. Known limitations

From primary sources ([Fusion MCP Server help](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_autodesk_fusion_mcp_server_html) + [Troubleshooting](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_troubleshooting_html), fetched 2026-06-11):
- **Desktop-only**: "Does not work with the Fusion web client."
- **Fusion must be running**: server "is only available while Fusion is open"; connection drops when Fusion closes.
- **Open document required**: "Commands do not execute — No document or workspace is open in Fusion."
- **Version-gated**: MCP checkbox "only available in recent releases."
- **Dynamic tools may change** between connections — no stable contract.
- **Local-only** — no remote access path.
- General AI caveat: "AI-generated responses may occasionally be incomplete or incorrect" ([What can Autodesk Assistant do?](https://help.autodesk.com/view/fusion360/ENU/?guid=AA_Capabilities) — fetched 2026-06-11).

Community (clearly marked): knightli.com's hands-on gear-modification walkthrough reported tolerance/precision limitations when Claude edited STEP geometry ([knightli.com](https://knightli.com/en/2026/05/14/claude-fusion-360-mcp-step-model-edit/) — fetched 2026-06-11; limitations section per prior pass P1, re-fetch returned partial content). **No primary or credible community source quantifies large-assembly behavior, latency, or rate limits — could not determine.**

## Decision implications for adze-fusion

- **The official connector ships inside Fusion, zero-install on the Autodesk side.** Any adze add-in competes against a preferences checkbox. Differentiation cannot be "connect Claude to Fusion."
- **Personal-use exclusion carves out the hobbyist market.** The free-tier cohort officially cannot use MCP-based AI tools — either an underserved segment (if Autodesk's block is tier-enforcement adze could not bypass either) or a contractual no-go zone; needs a licensing-terms read before targeting hobbyists.
- **Unauthenticated localhost is a governance gap.** Official posture is "no authentication required" on 27182. An adze product offering authenticated, audited, policy-gated access to the same session is a sourced differentiation surface (cf. ndoo's community bridge already adds Bearer tokens).
- **Dynamic tool discovery means no stable tool manifest** — adze designs depending on the official server's tool surface must handle tool-set drift at connection time.
- **Claude Desktop is the de-facto client**; claude.ai web cannot reach the local server. Multi-client (Cursor, VS Code, custom) support is officially sanctioned — an adze orchestrator is not protocol-blocked from the official endpoint.
- **Port 27182 can be treated as a stable default** in any adze documentation or auto-discovery logic, with the documented caveat that users may rebind it.

## Open questions

- Do education or startup Fusion tiers include MCP access? (Only personal use is explicitly excluded.)
- Exact tool enumeration of the official Fusion MCP server — needs an empirical connection dump (Stage 4 candidate).
- Why did the App Store "MCP Server for Autodesk Fusion" listing disappear — folded into core Fusion, or relocated on marketplace.autodesk.com?
- Does the Fusion connector appear in the claude.ai Connectors Directory with any plan badge? (Directory is paginated; Fusion entry not directly observed.)
- Empirical behavior when official + third-party Fusion MCP servers run simultaneously against one session.
- Rate limits / large-assembly performance of the official server — no data anywhere.

## Sources (full list, URLs + fetch dates)

Primary — Autodesk:
- [Autodesk MCP Server Help — home](https://help.autodesk.com/view/ADSKMCP/ENU/) — fetched 2026-06-11
- [Autodesk Fusion MCP Server](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_autodesk_fusion_mcp_server_html) — fetched 2026-06-11
- [Connecting to the Autodesk Fusion MCP Server](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_connecting_to_the_fusion_mcp_server_html) — fetched 2026-06-11
- [Troubleshooting (Fusion MCP)](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_troubleshooting_html) — fetched 2026-06-11
- [About Autodesk MCP Servers](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_CommonContent_about_autodesk_mcp_servers_html) — fetched 2026-06-11
- [Third-Party AI Tools (Claude, Cursor, VS Code) Fail to Connect to Autodesk Fusion MCP — Autodesk Support, 2026-05-05](https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/Claude-MCP-connector-failing-to-connect-to-Autodesk-Fusion.html) — fetched 2026-06-11
- [Bringing Fusion onto Claude for Creative Work — APS blog](https://aps.autodesk.com/blog/bringing-fusion-claude-creative-work) — fetched 2026-06-11
- [Claude for Autodesk Fusion — campaign page](https://www.autodesk.com/campaigns/fusion-360/claude-fusion) — fetched 2026-06-11 (direct fetch 403; content via Tavily extract)
- [What can Autodesk Assistant do? — Fusion Help](https://help.autodesk.com/view/fusion360/ENU/?guid=AA_Capabilities) — fetched 2026-06-11 (partial extract)
- [MCP Server for Autodesk Fusion — former App Store listing, Mac](https://apps.autodesk.com/FUSION/en/Detail/Index?id=7269770001970905100&ln=en&os=Mac) — fetched 2026-06-11 — **404** [empirical]
- [Autodesk MCP Servers — Autodesk AI](https://www.autodesk.com/solutions/autodesk-ai/autodesk-mcp-servers) — fetched 2026-06-11 (via search snippets)

Primary — Anthropic:
- [Claude for Creative Work — Anthropic news](https://www.anthropic.com/news/claude-for-creative-work) — fetched 2026-06-11
- [Use connectors to extend Claude's capabilities — Claude Help Center](https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities) — fetched 2026-06-11
- [Getting Started with Local MCP Servers on Claude Desktop — Claude Help Center](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop) — fetched 2026-06-11
- [Get started with custom connectors using remote MCP — Claude Help Center](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp) — fetched 2026-06-11

Community (used only where primary sources are silent, marked in text):
- [Autodesk Fusion Connector Risk — PromptArmor](https://www.promptarmor.com/connectors/autodesk-fusion) — fetched 2026-06-11
- [Connecting Claude to Fusion 360 — knightli.com](https://knightli.com/en/2026/05/14/claude-fusion-360-mcp-step-model-edit/) — fetched 2026-06-11 (partial re-fetch; limitations detail per P1 prior pass of 2026-05-15)
- [frankhommers/autodesk-fusion-mcp — GitHub](https://github.com/frankhommers/autodesk-fusion-mcp) — fetched 2026-06-11 (via search)
- [ndoo/fusion360-mcp-bridge — GitHub](https://github.com/ndoo/fusion360-mcp-bridge) — fetched 2026-06-11 (via search)
- [Fusion 360 MCP Bridge — LobeHub](https://lobehub.com/mcp/ndoo-fusion360-mcp-bridge) — fetched 2026-06-11 (via search)
