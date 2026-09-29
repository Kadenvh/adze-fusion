# Vault Health Dashboard

Dataview-driven health checks. Open this in Obsidian (with Dataview plugin installed) to render. Queries are static markdown until the plugin runs them.

**Lint operation reads this file.** Every session-close runs through these queries.

---

## Entry count by category

```dataview
TABLE length(rows) AS Count
FROM "vault"
WHERE file.folder != "vault/inbox" AND file.folder != "vault/00-meta" AND file.folder != "vault/00-meta/_templates"
GROUP BY file.folder
SORT file.folder ASC
```

## Source rigor compliance

Entries claiming category status but missing the Sources block.

```dataview
LIST
FROM "vault"
WHERE file.folder != "vault/inbox" AND file.folder != "vault/00-meta" AND file.folder != "vault/00-meta/_templates"
WHERE !contains(file.outlinks, "Sources") AND !contains(file.content, "## Sources")
```

## Orphans (no incoming links)

Entries that nothing else in the vault references. May need promotion to a hub or pruning.

```dataview
LIST
FROM "vault"
WHERE length(file.inlinks) = 0
WHERE file.folder != "vault/00-meta" AND file.folder != "vault/00-meta/_templates" AND file.folder != "vault/inbox"
WHERE file.name != "overview" AND file.name != "index" AND file.name != "hot" AND file.name != "log" AND file.name != "_health" AND file.name != "README"
```

## Hubs (>15 inlinks — split threshold per ScrapingArt rule)

Entries that have grown into mega-hubs. Per the Karpathy-LLM-Wiki pattern, hubs >15 members should be split into sub-hubs.

```dataview
LIST length(file.inlinks) + " inlinks"
FROM "vault"
WHERE length(file.inlinks) > 15
SORT length(file.inlinks) DESC
```

## Under-linked (≤1 outlink — likely silo or stub)

Entries that don't `[[link]]` to enough siblings. Healthy entries have 2-6 `[[wikilinks]]` per VAULT-RULES.

```dataview
LIST length(file.outlinks) + " outlinks"
FROM "vault"
WHERE length(file.outlinks) <= 1
WHERE file.folder != "vault/00-meta" AND file.folder != "vault/00-meta/_templates" AND file.folder != "vault/inbox"
WHERE file.name != "_health"
```

## Low confidence entries

Entries where the agent marked confidence as `low`. These need verification before being treated as load-bearing.

```dataview
LIST confidence
FROM "vault"
WHERE confidence = "low"
SORT file.path ASC
```

## Drafts not yet promoted

```dataview
LIST status
FROM "vault"
WHERE status = "draft"
SORT file.path ASC
```

## Contradictions flagged

Entries containing the `> [!contradiction]` callout — these have outstanding unresolved conflicts.

```dataview
LIST
FROM "vault"
WHERE contains(file.content, "[!contradiction]")
```

## Inbox depth (should be 0 at session end)

```dataview
LIST
FROM "vault/inbox"
WHERE file.name != "README"
```

## Stale entries (no edits in 30+ days)

```dataview
TABLE file.mtime AS "Last edited"
FROM "vault"
WHERE date(today) - file.mtime > dur(30 days)
WHERE file.folder != "vault/00-meta" AND file.folder != "vault/00-meta/_templates"
SORT file.mtime ASC
LIMIT 20
```

## 10-Source Test status

Stage 1 → Stage 2 transition requires **at least 10 promoted entries** spanning **≥6 of the 9 categories**, all clearing the 4-point quality bar (first-principles · sourced · replicable · load-bearing).

Check entry count above. Categories: 01-concepts, 02-platforms, 03-apis, 04-tools, 05-mcp-servers, 06-communities, 07-patterns, 08-decisions, 09-sources.

**Status (manual check at lint time):** [x] PASSED · [ ] PENDING — entries: **21** · categories with ≥1 entry: **8** (01, 02, 03, 04, 05, 06, 07, 09 — only 08-decisions is empty, as designed until Stage 3). Last verified: 2026-05-27 after Stage 0.5 promotion pass.

---

## Related

- [[overview]]
- [[index]]
- [[hot]]
- [[00-meta/VAULT-RULES]]
