---
title: Project Salvador (discontinued)
category: 04-tools
tags: [tool, autodesk, ai, fusion-360, discontinued, cautionary]
status: promoted
confidence: medium
license: proprietary
language: N/A
maturity: abandoned
---

# Project Salvador (discontinued)

Autodesk's discontinued in-Fusion plugin that chained GPT-4 Turbo + DALL-E / Stable Diffusion + Vectorizer AI to do text-to-image, image-to-image, and text-to-sketch. **No longer being developed**; its capabilities are being absorbed by [[autodesk-assistant]]. The single most important cautionary precedent in P1.

## First principles

Salvador's architecture was a third-party-style LLM wrapper that **happened to be authored by Autodesk** — chain external image and text models, glue them to the Fusion sketch surface, ship as an App Store plugin. The death-mechanism for it was simple: the same first-party that built it built a structurally deeper alternative (Assistant with Script Execute against the native API) and chose the deeper one. The blog phrasing fetched in P1 ("not currently being developed and is not planned to be superseded by Autodesk Assistant") is **`[unverified]` — internally contradictory in the form quoted, likely transcription-corrupted; the actual blog should be re-fetched before any vault entry leans on the exact wording**. The pattern is sound regardless: Salvador stopped shipping; Assistant ships in its place.

## Surface

| Aspect | Detail |
|---|---|
| Type | Fusion App Store plugin (Beta) |
| Authors | Autodesk |
| Stack | GPT-4 Turbo (text) + DALL-E / Stable Diffusion (image generation) + Vectorizer AI (image-to-sketch) |
| Capabilities | text → image, image → image, text → sketch |
| Status | Discontinued; capabilities absorbed by [[autodesk-assistant]] |
| Lifespan | Beta in App Store; no GA |

## Implications for adze

- **Wrapping a frontier LLM in a CAD plugin is a fragile thesis when the host vendor competes.** Salvador had Autodesk-internal advantages (App Store featuring, brand) and still lost to Assistant. A third-party wrapper has none of those advantages.
- **The win condition for adze-fusion is depth that Assistant structurally cannot reach** — cross-CAD reach (Assistant is per-host), vendor-neutral observability, recipe portability across CAD platforms, governance for multi-engineer teams. Anything Assistant can absorb, it eventually will.
- **App Store as a sole distribution path is risky.** Salvador was on the App Store; that didn't save it. Adze-fusion needs the Marketplace certification path ([[../06-communities/autodesk-design-and-make-marketplace]]) and/or a non-Autodesk-channel surface (direct, GitHub, partner resellers).

## Open questions

- Exact wording of the discontinuation note — the P1 quote reads contradictorily and is likely transcription-corrupted. Needs re-fetch from `autodesk.com/products/fusion-360/blog/project-salvador-autodesk-fusion-app-store/`. `[unverified]`
- Whether Salvador's beta sunset note refers to its original Beta-only launch or to a more recent discontinuation announcement.
- Whether any Salvador codebase or design patterns are publicly documented (post-mortem material useful for adze).

## Sources

- [Introducing Project Salvador for Autodesk Fusion — Fusion blog](https://www.autodesk.com/products/fusion-360/blog/project-salvador-autodesk-fusion-app-store/) — fetched 2026-05-15 (via search); direct content `[unverified]` due to apparent transcription corruption in P1
- [research/findings/P1-fusion-connector-ecosystem.md](../../research/findings/P1-fusion-connector-ecosystem.md) — internal finding
- [research/findings/P0-adversarial-review.md](../../research/findings/P0-adversarial-review.md) — flags the Salvador quote as transcription-suspect

## Related

- [[autodesk-assistant]]
- [[../02-platforms/fusion-360]]
- [[../06-communities/autodesk-design-and-make-marketplace]]
