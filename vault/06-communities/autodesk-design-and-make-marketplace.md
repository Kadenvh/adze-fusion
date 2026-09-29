---
title: Autodesk Design and Make Marketplace
category: 06-communities
tags: [community, autodesk, marketplace, certification, distribution]
status: promoted
confidence: medium
---

# Autodesk Design and Make Marketplace

Autodesk's distribution + certification surface for third-party MCP servers and agentic tools. Certification is **ISO 42001-aligned with NIST AI RMF and IEEE CertifAIEd**. Certified MCPs become callable from [[../04-tools/autodesk-assistant]].

## First principles

The Marketplace is **not** the App Store. The App Store ([[autodesk-app-store]] — not yet promoted) distributes per-platform add-ins; the Marketplace certifies AI-shaped artifacts (MCP servers, agents) and threads them into Assistant's orchestration. The certification regime is the gatekeeper: an MCP author files a Tool Manifest (declaring tools, resources, prompts, external connections, APS APIs, AI providers used), a Publisher Declaration (security attestations), submits to `appsubmissions@autodesk.com`, and on approval is published. ISO 42001 + NIST AI RMF + IEEE CertifAIEd alignment signals "we know we are gatekeeping AI in a regulated industry."

## Surface

| Aspect | Detail |
|---|---|
| Run by | Autodesk |
| Purpose | Distribute + certify third-party MCPs and agentic tools across Autodesk products |
| Submission cost | No fee mentioned (per P1) — needs verification |
| Submission email | `appsubmissions@autodesk.com` |
| Process steps | (1) Create MCP Tool Manifest, (2) Complete Publisher Declaration, (3) Submit for approval, (4) Publish and monitor |
| Transport requirement | stdio is the example; HTTPS required for any external connection |
| Standards alignment | ISO 42001, NIST AI RMF, IEEE CertifAIEd |
| Assistant integration | Certified MCPs are callable from Autodesk Assistant |
| Status | Active in 2026; openness to new entrants `[unverified]` |

The full certification pipeline is documented at the [MCP Publisher Guide](https://aps.autodesk.com/marketplace/mcp-publisher-guide).

## Implications for adze

- **The certification path is real distribution, not just badging.** Marketplace-certified MCPs become first-class tools that Assistant orchestrates — which means a properly scoped adze-fusion MCP doesn't compete with Assistant, it gets called by Assistant. This reframes the competitive positioning.
- **ISO 42001 + NIST AI RMF + IEEE CertifAIEd is a real bar.** Adze-fusion will need a security model, data-flow story, and AI-system documentation it doesn't have yet. Worth surfacing for ADR-001 as a non-trivial cost item.
- **Openness to new entrants is the load-bearing unknown.** If certification is restricted to pre-selected partners (e.g., Synera, which already shipped a connector), the Marketplace path is closed for now and adze-fusion ships via App Store and/or direct.
- **Submission is free as of P1** but "free for v1 / Autodesk introduces a revenue share later" is a known platform pattern — assume nothing about long-term economics.

## Open questions

- Is certification currently open to new entrants? `[unverified]`
- Pricing / revenue share when Assistant orchestrates a certified MCP? `[unverified]`
- Are there published examples of approved third-party MCP manifests? (Reference value for adze-fusion's authoring.)
- Are MCPs reviewed by humans, automated tools, or both?

## Sources

- [MCP publisher guide — Autodesk Platform Services](https://aps.autodesk.com/marketplace/mcp-publisher-guide) — fetched 2026-05-15
  > "(1) Create the MCP Tool Manifest (2) Complete the Publisher Declaration (3) Submit for approval (4) Publish and monitor"; submit to `appsubmissions@autodesk.com`.
- [Design and Make Marketplace — FifthRow analysis](https://www.fifthrow.com/blog/from-bottleneck-to-backbone-how-autodesk-s-agentic-ai-design-and-make-marketplace-is-transforming-manufacturing-tech-transfer) — fetched 2026-05-15
  > ISO 42001-aligned certification, NIST AI RMF, IEEE CertifAIEd; Publisher Center as gatekeeper.
- [Building for Agentic AI: What's New in Autodesk Platform Services — APS blog](https://aps.autodesk.com/blog/building-agentic-ai-whats-new-autodesk-platform-services) — fetched 2026-05-15

## Related

- [[../05-mcp-servers/autodesk-fusion-mcp]]
- [[../04-tools/autodesk-assistant]]
- [[../01-concepts/mcp-protocol]]
- [[../09-sources/aps-mcp-publisher-guide]]
