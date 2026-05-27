# Summary

<!-- One-paragraph description of what this PR changes and why. -->

## Type of change

- [ ] Vault entry — new or updated under `vault/`
- [ ] Research finding — new file in `research/findings/`
- [ ] Raw source — new file in `raw/`
- [ ] Schema / ontology change — affects `vault/00-meta/`
- [ ] Architecture decision (ADR) — new file in `vault/08-decisions/`
- [ ] Agent / operating manual — changes to `CLAUDE.md`
- [ ] Plan — changes to `plans/`
- [ ] GitHub config / templates — changes to `.github/`
- [ ] Docs / README

## Quality bar (for vault entries)

If this PR adds or updates entries under `vault/01-*` through `vault/09-*`:

- [ ] Entry has correct frontmatter (`category`, `status`, `confidence`)
- [ ] First-principles section is present and substantive
- [ ] All factual claims have citations, or are marked `[empirical]` / `[unverified]`
- [ ] `## Related` block has 2-6 `[[wikilinks]]` to sibling entries
- [ ] No prose dumps — body is under 600 words (excluding sources)
- [ ] No new top-level category folders (any taxonomy change must go through an ADR)

## Provenance

<!-- If this PR derives from a research stream or source: link the finding file and the source entry. -->

- Research stream: `research/findings/...`
- Source: [[...]]

## Checklist

- [ ] `vault/index.md` updated if a new entry was promoted
- [ ] `vault/log.md` updated with this operation
- [ ] `vault/hot.md` updated if context-relevant
- [ ] No `[unverified]` claims that are load-bearing for the change
- [ ] If contradictions were introduced or resolved, `[!contradiction]` callouts are correct
