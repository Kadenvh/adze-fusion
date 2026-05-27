# P3 — Obsidian for LLM-Curated Vaults

**Date:** 2026-05-15
**Researcher:** general-purpose sub-agent

## Summary

The adze-fusion vault sits squarely inside the **Karpathy LLM Wiki pattern** Karpathy himself published as a gist on 2026-04-04 — "Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase." For that pattern the high-leverage plugins are **Dataview** (querying vault state for health checks), **Templater** (enforcing the 5 entry templates), **Linter** (frontmatter/format hygiene), and an **Obsidian MCP server** (Claude writes via tools, not raw filesystem). **Smart Composer / Smart Connections are mostly redundant** when Claude Code already curates the vault. The vault's 9 stable categories are sufficient structure; layer thin **Maps of Content (MOCs)** rather than reorganizing. Discipline-as-data is the hard part: orphan/hub/broken-link Dataview queries, status frontmatter, periodic lint passes.

## First principles

An Obsidian vault is **a folder of plain markdown files** with YAML frontmatter and `[[wikilinks]]` — nothing more. No DB, no server, no proprietary format ([Obsidian Help — File formats](https://help.obsidian.md/file-formats)). This matters for LLM curation because: (1) the LLM can read/write with any filesystem tool, no SDK needed; (2) wikilinks are *text*, so hallucinated targets compile silently as broken links — they must be verified; (3) the graph is *derived* from text, not stored, so graph hygiene means text hygiene.

## Plugins worth installing

### Templater
`SilentVoid13/Templater` — JavaScript-evaluated templates with prompts, file ops, dynamic frontmatter. For an LLM-curated vault it matters less for runtime prompting than for **enforcing the 5 entry templates** (concept, tool, api, source, decision). When a human (or the agent via MCP) creates a new note, Templater inserts the canonical YAML stub. Gotcha: Templater runs *client-side*; if the LLM writes via MCP or raw filesystem it bypasses Templater entirely. Solution: have Claude reference the template files in `vault/templates/` directly when creating notes. [AI-templates for Obsidian Templater — forum](https://forum.obsidian.md/t/ai-templates-for-obsidian-templater/111251)

### Dataview
`blacksmithgu/obsidian-dataview` — query language over frontmatter and wikilinks. This is the **single most important plugin** for the adze-fusion workflow. It surfaces vault state the LLM can't see at write time:

- Orphan notes: `LIST FROM "" WHERE length(file.inlinks) = 0 AND length(file.outlinks) = 0` ([safjan.com](https://safjan.com/list-unlinked-orphaned-notes-obsidian/))
- Stub/draft entries: `LIST FROM "" WHERE status = "draft" SORT file.mtime DESC`
- Hub detection: `LIST FROM "" WHERE length(file.inlinks) > 20 SORT length(file.inlinks) DESC`
- Per-category counts: `TABLE length(rows) AS Count FROM "concepts" GROUP BY status`
- Broken wikilinks: use `dataviewjs` to read `app.metadataCache.unresolvedLinks` ([forum thread](https://forum.obsidian.md/t/find-all-non-existant-notes-and-the-note-theyre-mentioned-in/50412))

Gotcha: Dataview is read-only at the file level; it doesn't fix anything, it surfaces. Pair with Linter or an MCP write pass.

### Linter
`platers/obsidian-linter` — opinionated formatter for markdown + YAML. Critical rules for adze-fusion: YAML array formatting (single vs. multi-line consistency), tag normalization (no `#` inside frontmatter), blank line after frontmatter, heading consistency, escape-character normalization. Configure to run on-save so LLM-generated notes are normalized immediately ([Linter YAML rules docs](https://platers.github.io/obsidian-linter/settings/yaml-rules/)). Gotcha: don't enable "YAML Title Alias" and "Format YAML aliases" together — they fight.

### Smart Composer / Smart Connections (verdict)
- **Smart Composer** (`glowingjade/obsidian-smart-composer`) — vault-aware chat with RAG over your notes. Multi-provider including Anthropic ([repo](https://github.com/glowingjade/obsidian-smart-composer)).
- **Smart Connections** — embedding-based "related notes" suggestions.

**Verdict for adze-fusion: skip both.** Claude Code is already the curator with full repo context and MCP tools. Adding an in-Obsidian RAG layer over the same vault produces two competing curation agents with conflicting signals. Plus, as of Jan 2026 Anthropic restricted third-party OAuth — using Smart Composer with a Max subscription risks an account ban (see search result for Smart Composer above; the project README also notes API-key vs subscription paths). If the user wants in-Obsidian Q&A later, revisit, but the canonical write path is Claude Code → MCP → vault.

### Others worth mentioning
- **Obsidian Local REST API** (`coddingtonbear/obsidian-local-rest-api`) — required by the best MCP server (cyanheads).
- **Tag Wrangler** — rename tags safely across the vault when the taxonomy drifts.
- **Breadcrumbs** — explicit `up::`/`down::` hierarchy fields if you eventually want a parent-child overlay on top of MOCs.
- **Excalidraw** — only if visual concept maps become part of curation. Skip for now.
- **Git** (core/community) — autosync the `vault/` folder; the repo already has git.

## MCP servers for Obsidian

State of the art is **`cyanheads/obsidian-mcp-server`** (Apache-2.0, v3.2.2 May 2026, 561 stars). It exposes:

- **Read:** `obsidian_get_note`, `obsidian_list_notes`, `obsidian_list_tags`, `obsidian_search_notes` (text / JSONLogic / BM25 via Omnisearch), `obsidian_list_commands`.
- **Write:** `obsidian_write_note`, `obsidian_append_to_note`, `obsidian_patch_note` (surgical edits at headings/blocks/frontmatter), `obsidian_replace_in_note` (regex), `obsidian_manage_frontmatter` (atomic key-level get/set/delete), `obsidian_manage_tags`, `obsidian_delete_note`, `obsidian_execute_command`.

Requires the **Obsidian Local REST API v4.0.0+** plugin with an API key. Supports folder-scoped permissions via env vars and a global read-only kill switch — relevant for protecting `vault/sources/` immutability ([repo](https://github.com/cyanheads/obsidian-mcp-server)).

Alternatives: `MarkusPfundstein/mcp-obsidian` (older, also REST-API-based), `StevenStavrakis/obsidian-mcp` (simpler), `bitbonsai/mcpvault` (filesystem-direct, doesn't require Obsidian running). For adze-fusion: pick cyanheads — surgical frontmatter writes matter for status workflow transitions.

## Vault structure patterns

**PARA** (Projects/Areas/Resources/Archives) is task-oriented — wrong fit for a research wiki. **LATCH** (Location/Alphabet/Time/Category/Hierarchy) is too coarse. **Zettelkasten** is conceptually aligned but its UID-based filenames (`202605151430.md`) defeat human-readable graphs. **MOCs** (Maps of Content, Nick Milo / LYT) are the lightest-weight overlay and they compose with the existing folder structure.

For adze-fusion: **keep the 9 folder categories, add 9 thin MOC notes** — `concepts-MOC.md`, `platforms-MOC.md`, etc. — each a hand-/LLM-curated index of the most load-bearing notes in that category with one-line annotations. A note can be folder-located *and* MOC-referenced from anywhere ([obsidian.rocks — MOCs](https://obsidian.rocks/maps-of-content-effortless-organization-for-notes/), [Aidan Helfant — PARA Not Working? Create MOCs](https://medium.com/@aidan.helfant/para-not-working-create-mocs-in-obsidian-3e16c176bf46)). MOCs also give Claude an entry point — "read `concepts-MOC.md` first" beats "scan all of `concepts/`".

## Atomization patterns

Zettelkasten doctrine: atomicity is a **direction**, not an admission criterion ([zettelkasten.de — Principle of Atomicity](https://zettelkasten.de/posts/principle-of-atomicity-difference-between-principle-and-implementation/)). Don't gate writes on atomicity — split after the fact. Detection signals:

- **Word count outlier:** `TABLE file.size FROM "concepts" WHERE file.size > 8000 SORT file.size DESC` — flag for split review.
- **H2 multiplicity:** a note with 5+ top-level H2 headings is probably 5 notes. Detect with a script reading file content, not pure Dataview.
- **Wikilink density inversion:** atomic notes have *high outlink count per word*. Long notes with few outlinks are typically un-atomized prose dumps.

The Karpathy gist explicitly endorses splitting on demand: the LLM's job during periodic lint is to find these and propose splits.

## Link quality enforcement

Concrete Dataview queries (drop in a `vault/_health.md` dashboard):

```dataview
TABLE length(file.inlinks) AS "In", length(file.outlinks) AS "Out"
FROM "" WHERE length(file.inlinks) = 0 AND length(file.outlinks) = 0
SORT file.mtime DESC
```

```dataview
TABLE length(file.inlinks) AS "Inbound"
FROM "" WHERE length(file.inlinks) > 25
SORT length(file.inlinks) DESC
```

Broken-link detection needs DataviewJS:

```dataviewjs
const broken = app.metadataCache.unresolvedLinks;
for (const [src, targets] of Object.entries(broken)) {
  for (const t of Object.keys(targets)) {
    dv.paragraph(`- [[${src}]] → **${t}** (missing)`);
  }
}
```

This is the canonical pattern for catching LLM-hallucinated wikilinks ([forum reference for unresolvedLinks](https://forum.obsidian.md/t/find-all-non-existant-notes-and-the-note-theyre-mentioned-in/50412)). Hallucination prevention pattern from the LLM Wiki community: "LLM-proposed named references are accepted only when the exact text appears in the title, filename, or body" ([green-dalii/obsidian-llm-wiki dedup docs](https://deepwiki.com/green-dalii/obsidian-llm-wiki/6.3-duplicate-detection)).

## Templates and frontmatter

Canonical frontmatter shape for LLM-friendly vaults:

```yaml
---
title: "Mass properties API"
type: api                # concept | tool | api | source | decision
status: draft            # draft | reviewed | promoted
category: APIs
tags: [fusion360, geometry]
sources: ["[[s-fusion360-api-ref-2026-05]]"]
created: 2026-05-15
updated: 2026-05-15
aliases: ["MassProps", "GetMassProperties"]
---
```

Conventions: lowercase singular `type`, explicit `status` lifecycle (draft → reviewed → promoted), `sources` as wikilinks (not free strings) so backlinks light up source notes, `aliases` for dedup matching, ISO dates. Tag taxonomy: keep to 2-level max (`platform/fusion`, not `platform/fusion/python/api/geometry`); tags are signals, structure lives in folders + MOCs ([blog.shuvangkardas.com](https://blog.shuvangkardas.com/obsidian-note-organization/)).

## Karpathy method — concrete grounding

Karpathy's relevant published work (cited):

1. **"LLM Knowledge Bases" tweet, 2026-04-03** ([x.com/karpathy/status/2039805659525644595](https://x.com/karpathy/status/2039805659525644595)): "a large fraction of my recent token throughput is going less into manipulating code, and more into manipulating" knowledge bases. His research wiki on one topic: ~100 articles, ~400k words.
2. **"llm-wiki" gist, 2026-04-04** ([gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)). Direct quotes: *"the wiki is a persistent, compounding artifact"*, *"knowledge is compiled once and then kept current, not re-derived on every query"*, *"Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase."* Three-layer architecture (raw sources / wiki / schema), `index.md` content catalog, `log.md` append-only chronological record with parsable syntax (`## [YYYY-MM-DD] action | title`), periodic lint pass.
3. **"Software Is Changing (Again)" / Software 3.0 talk, AI Startup School, 2025-06-17** ([summary at ikyle.me](http://ikyle.me/blog/2025/andrej-karpathy-software-is-changing-again)): LLM-as-OS framing — context window is RAM, RAG is the filesystem. Implication: the vault *is* the LLM's filesystem, so vault hygiene = OS hygiene.
4. **Claude Code skills / agentic engineering, 2026** ([Augment Code](https://www.augmentcode.com/blog/karpathy-skills-on-openclaw-agents-don-t-write-better-code-but-they-do-it-more-efficiently)): "state assumptions, surface tradeoffs, ask if unclear; simplicity first with no speculative features; surgical changes that don't touch adjacent code; and goal-driven execution with verifiable success criteria." Applied to vault writes: small atomic notes beat sprawling re-writes; verify wikilink targets exist; lint after each batch.

Concrete adze-fusion applications:
- Treat `vault/` like a codebase: every "compile" = ingest, every "test" = lint pass, every "PR" = promotion from draft to promoted.
- Maintain `vault/index.md` and `vault/log.md` per the gist; the log is greppable session history.
- Claude.md / charter functions as the gist's "schema" layer.

## Graph hygiene metrics

No published canonical thresholds (search confirmed — Obsidian doesn't expose cluster coefficient). Practical heuristics from PKM practitioners:

- **Average outlinks per note: 3–7** is healthy. Below 2 = silo risk; above 15 = hub degeneracy.
- **Orphan ratio: <5%** of total notes. Track in a Dataview dashboard.
- **Graph view becomes useful at ~200 notes** ([medium.com — Steven Thompson](https://medium.com/a-voice-in-the-conversation/obsidian-graph-view-searching-nonhierarchical-networks-229630185e17)).
- **Bridge nodes** (high betweenness centrality) are MOC candidates. Tools like InfraNodus can surface them but require external indexing.
- Failure modes: **hub-and-spoke degeneracy** (everything links to one MOC, nothing links sideways) — fix by writing concept-to-concept links during ingest, not just concept-to-MOC.

## LLM-vault anti-patterns

| Failure | Detection |
| --- | --- |
| Hallucinated wikilinks to non-existent notes | `unresolvedLinks` Dataview JS scan |
| Duplicate entries (same concept, different title) | alias-based dedup; fuzzy title match on ingest |
| Prose drift (notes balloon to essays) | Dataview by `file.size`; H2-count script |
| Template drift (frontmatter keys diverge) | Linter rules + Dataview audit of required keys |
| Source rigor erosion (claims without `sources:`) | `LIST FROM "" WHERE !sources AND type != "decision"` |
| Tag sprawl (similar tags multiply) | Tag Wrangler audit; weekly tag-count Dataview |
| Stale promoted entries | `WHERE status = "promoted" AND updated < date(today) - dur(180 days)` |

Prevention is the periodic lint — exactly the operation Karpathy specifies.

## Periodic review patterns

Weekly (5 min, mostly automated):
- Run `_health.md` dashboard: orphans, hubs, broken links, draft count.
- Eyeball graph view for new clusters or islands.

Monthly (20 min):
- Promote draft → reviewed → promoted based on usage signals (inlinks).
- Run Linter across vault.
- Tag audit; consolidate near-duplicates.
- Lint pass via Claude: feed it the dashboard output, ask for split/merge proposals.

Quarterly:
- MOC refresh — each of the 9 MOC notes re-curated.
- Source freshness check — any source older than 6 months flagged for re-fetch.

## Export and portability

**Quartz v5** ([quartz.jzhao.xyz](https://quartz.jzhao.xyz/)) is the dominant Obsidian-to-static-site path. It parses Obsidian frontmatter natively and supports `publish: true` selective publishing via the `ExplicitPublish` filter. Patterns that keep the vault export-ready:

- Use **relative wikilinks** only; no absolute Obsidian URIs.
- Add `publish: true` to promoted notes only — draft/reviewed stay private.
- Avoid plugin-specific syntax (Excalidraw embeds, Dataview tables) in note *content* — Dataview blocks don't render in Quartz. Put queries in a single `_health.md` that you don't publish.
- Standard markdown images; no proprietary embed syntax.
- ISO dates everywhere (Quartz's frontmatter parser is strict).

## Privacy

Obsidian itself is local-first and collects no telemetry ([Obsidian privacy policy](https://obsidian.md/privacy)). The developer policy prohibits client-side telemetry in community plugins ([help.obsidian.md/plugin-security](https://help.obsidian.md/plugin-security)) — but server-side usage tracking by AI plugins (Smart Composer, Copilot) is fair game and they do it. Phone-home risks to know:

- Any AI plugin sends note content to its provider on each query — that's the *point*, but it's still egress.
- Sync plugins (Obsidian Sync, Remotely Save, git-sync to a private remote) — verify endpoint.
- Update checks: Obsidian itself pings on launch; disable in settings if needed.
- Local REST API plugin listens on `127.0.0.1` by default — fine, but don't expose externally.

For adze-fusion: if vault may contain confidential research, *don't* install Smart Composer / Smart Connections (they egress notes). MCP server traffic stays local-loopback. Anthropic API egress is unavoidable since Claude is the curator — make that an explicit accepted boundary.

## Decision implications for adze-fusion

1. **Install:** Dataview, Templater, Linter, Local REST API, Tag Wrangler. Skip Smart Composer/Smart Connections.
2. **Adopt cyanheads MCP server** as the canonical Claude-write path; folder-scope `vault/sources/` read-only.
3. **Add 9 thin MOC notes** (`{category}-MOC.md`) layered on existing folders; don't refactor categories.
4. **Create `vault/index.md` and `vault/log.md`** per Karpathy gist; make `log.md` append-only with `## [YYYY-MM-DD] action | title` syntax.
5. **Build `vault/_health.md` dashboard** with the orphan / hub / broken-link / status Dataview queries above; do not publish.
6. **Enforce frontmatter shape via Linter + Templater**; add a `status` lifecycle (draft → reviewed → promoted).
7. **Source rigor query:** `LIST FROM "" WHERE !sources AND type != "decision"` — make zero-sources a release blocker.
8. **Set `publish: true` gate** for any future Quartz export; default is private.
9. **Periodic-lint command in CLAUDE.md** for the agent: weekly orphan/broken-link sweep, monthly promote-or-split pass.
10. **Treat `sources/` as immutable**; never let Claude mutate raw research, only the curated layer.

## Open questions

- Does the user want `vault/log.md` machine-parseable (the Karpathy spec) or human-narrative? They diverge.
- Will the vault eventually publish via Quartz? If yes, lock frontmatter conventions now to avoid retrofit pain.
- Is Tag Wrangler worth the dependency or does folder + MOC cover tagging needs?
- Hub-threshold tuning: 25 inlinks is a guess; calibrate after 100+ notes exist.

## Sources

**Karpathy primary:**
- Karpathy, "LLM Knowledge Bases" tweet, 2026-04-03 — https://x.com/karpathy/status/2039805659525644595
- Karpathy, `llm-wiki` gist, 2026-04-04 — https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- Karpathy, "Software Is Changing (Again)" / Software 3.0 talk, AI Startup School, 2025-06-17 — summary: http://ikyle.me/blog/2025/andrej-karpathy-software-is-changing-again
- Karpathy Claude Code skills / agentic engineering coverage — https://www.augmentcode.com/blog/karpathy-skills-on-openclaw-agents-don-t-write-better-code-but-they-do-it-more-efficiently

**Obsidian core / plugins:**
- Obsidian Help — https://help.obsidian.md/
- Obsidian privacy — https://obsidian.md/privacy
- Plugin security — https://help.obsidian.md/plugin-security
- Dataview — https://github.com/blacksmithgu/obsidian-dataview
- Dataview docs — https://blacksmithgu.github.io/obsidian-dataview/
- Obsidian Linter — https://platers.github.io/obsidian-linter/settings/yaml-rules/
- Smart Composer — https://github.com/glowingjade/obsidian-smart-composer
- AI-templates for Templater — https://forum.obsidian.md/t/ai-templates-for-obsidian-templater/111251

**MCP servers:**
- cyanheads/obsidian-mcp-server — https://github.com/cyanheads/obsidian-mcp-server
- MarkusPfundstein/mcp-obsidian — https://github.com/MarkusPfundstein/mcp-obsidian
- StevenStavrakis/obsidian-mcp — https://github.com/StevenStavrakis/obsidian-mcp
- bitbonsai/mcpvault — https://github.com/bitbonsai/mcpvault

**Structure / atomicity / MOCs:**
- Zettelkasten — Principle of Atomicity — https://zettelkasten.de/posts/principle-of-atomicity-difference-between-principle-and-implementation/
- Zettelkasten — Atomic Note-Taking Guide — https://zettelkasten.de/atomicity/guide/
- Obsidian Rocks — Maps of Content — https://obsidian.rocks/maps-of-content-effortless-organization-for-notes/
- Aidan Helfant — PARA Not Working? Create MOCs — https://medium.com/@aidan.helfant/para-not-working-create-mocs-in-obsidian-3e16c176bf46
- Shuvangkar Das — Folders vs MOCs vs Tags — https://blog.shuvangkardas.com/obsidian-note-organization/

**Orphan / link detection:**
- safjan.com — List Unlinked Notes — https://safjan.com/list-unlinked-orphaned-notes-obsidian/
- Christopher Goodman — Finding Orphans — https://www.cgoodman.com/blog/2025-05-05-obsidian-orphans/
- Forum — Non-existent notes — https://forum.obsidian.md/t/find-all-non-existant-notes-and-the-note-theyre-mentioned-in/50412
- Forum — Dynamic Dataview orphan finder — https://forum.obsidian.md/t/a-dynamic-dataview-orphan-finder/47833

**LLM-wiki community implementations:**
- green-dalii/obsidian-llm-wiki (duplicate detection) — https://deepwiki.com/green-dalii/obsidian-llm-wiki/6.3-duplicate-detection
- ScrapingArt/Karpathy-LLM-Wiki-Stack — https://github.com/ScrapingArt/Karpathy-LLM-Wiki-Stack
- MindStudio — Karpathy LLM Wiki guide — https://www.mindstudio.ai/blog/andrej-karpathy-llm-wiki-knowledge-base-claude-code

**Graph hygiene:**
- Steven Thompson — Obsidian Graph View — https://medium.com/a-voice-in-the-conversation/obsidian-graph-view-searching-nonhierarchical-networks-229630185e17
- InfraNodus — Knowledge Graph metrics — https://infranodus.com/use-case/visualize-knowledge-graphs-pkm

**Export:**
- Quartz v5 — https://quartz.jzhao.xyz/
- Quartz Obsidian compatibility — https://quartz.jzhao.xyz/features/Obsidian-compatibility
