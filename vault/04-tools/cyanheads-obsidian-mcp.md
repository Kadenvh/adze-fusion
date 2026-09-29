---
title: cyanheads/obsidian-mcp-server
category: 04-tools
tags: [tool, mcp, obsidian, vault, curation]
status: promoted
confidence: high
license: Apache-2.0
language: TypeScript
maturity: production
---

# cyanheads/obsidian-mcp-server

MCP server that exposes Obsidian vault operations as JSON-RPC tools for Claude to call. The canonical write path between Claude Code and the adze-fusion vault — when wired in, Claude mutates `vault/` through surgical tool calls rather than raw filesystem writes.

## First principles

The vault is a folder of plain markdown files; Claude can already read/write via filesystem tools. So what does an Obsidian MCP server add? **Surgical, frontmatter-aware writes** — `obsidian_patch_note` edits at a specific heading or block, `obsidian_manage_frontmatter` does atomic key-level get/set/delete on YAML, `obsidian_manage_tags` keeps the tag taxonomy clean. These prevent the failure mode where filesystem-level rewrites silently break wikilinks, reorder frontmatter keys, or smash adjacent content. The server also enforces folder-scoped permissions via env vars, so `vault/09-sources/` can be made read-only at the protocol layer — making "raw is immutable" structurally true, not just a convention.

## Surface

| Aspect | Detail |
|---|---|
| License | Apache-2.0 |
| Version | v3.2.2 (May 2026) |
| Stars | 561 (P3 finding, 2026-05-15) |
| Language | TypeScript |
| Dependency | Obsidian Local REST API plugin v4.0.0+ (must be installed and API key set in Obsidian) |
| Transport | stdio (project-scope via `.mcp.json`) |
| Read tools | `obsidian_get_note`, `obsidian_list_notes`, `obsidian_list_tags`, `obsidian_search_notes` (text / JSONLogic / BM25 via Omnisearch), `obsidian_list_commands` |
| Write tools | `obsidian_write_note`, `obsidian_append_to_note`, `obsidian_patch_note`, `obsidian_replace_in_note` (regex), `obsidian_manage_frontmatter`, `obsidian_manage_tags`, `obsidian_delete_note`, `obsidian_execute_command` |
| Permissions | Folder-scoped via env vars + global read-only kill switch |

## Implications for adze

- **Optional for now, valuable once vault traffic grows.** While the vault has <10 entries, raw filesystem writes are fine. Past ~30 entries with cross-links, the surgical-write story matters and the server starts paying for itself.
- **Folder-scoped permissions** map cleanly onto the three-layer architecture: `raw/` (read-only via env), `vault/09-sources/` (read-only for the agent — Karpathy's anchor sources should be immutable), `vault/` rest writable.
- **Requires Obsidian to be running** — REST API plugin is an Obsidian-side dependency. For headless / CI usage, fall back to filesystem writes.
- **Alternative: filesystem-direct.** `bitbonsai/mcpvault` doesn't require Obsidian running; viable if the user runs the curator from a server.

## Open questions

- Maintenance signal beyond stars — release cadence, issue triage SLA. Not surfaced in P3.
- Whether the surgical `patch_note` is robust to wikilinks vs heading-name drift (renaming a heading the patch targets is a known footgun class in patch-based editors). `[unverified]`
- Mobile Obsidian compatibility — relevant if the user wants to read the vault on phone but irrelevant to Claude's write path.

## Sources

- [cyanheads/obsidian-mcp-server — GitHub](https://github.com/cyanheads/obsidian-mcp-server) — fetched 2026-05-15
  > v3.2.2; Apache-2.0; 561 stars; full tool list as enumerated above.
- [research/findings/P3-obsidian-llm-vault-curation.md](../../research/findings/P3-obsidian-llm-vault-curation.md) — internal finding (P3)

## Related

- [[../01-concepts/mcp-protocol]]
- [[../07-patterns/karpathy-llm-wiki-pattern]]
- [[../09-sources/karpathy-llm-wiki-gist]]
- [[../09-sources/scrapingart-llm-wiki-stack]]
