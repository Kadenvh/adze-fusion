# Vault Rules (Karpathy Method)

Operating discipline for adding, editing, and curating vault entries. Read this every session before promoting findings.

---

## Quality bar — every entry must clear this

An entry earns its place in the vault when it meets **all four**:

1. **First-principles.** It describes what the thing actually IS at the level a system designer can reason about. Marketing copy and surface descriptions don't count.
2. **Sourced.** Every factual claim has a `Source:` reference with URL, fetch date, and the relevant quote or paraphrase. No unsourced claims survive.
3. **Replicable.** A future agent or human picks up the entry cold and reconstructs the picture. The entry doesn't rely on context that lives only in someone's head.
4. **Load-bearing.** A future decision (architecture, prioritization, scope) leans on it. If no decision will ever cite it, it's interesting but not vault-worthy. Send it to a public blog post instead.

Entries that don't clear the bar **stay in `inbox/`** until either fixed or deleted. They do NOT live in a category folder in a partial state.

---

## Source rigor — non-negotiable

Every claim of fact has a citation. The format:

```markdown
## Sources

- [Title or short description](URL) — fetched YYYY-MM-DD
  > "Relevant quote or paraphrase that supports the claim above."
```

For multi-source claims, link them inline:

```markdown
Fusion 360 add-ins run in a Python interpreter embedded in the Fusion process [source: [autodesk-fusion-api-reference]], on a single thread that owns both UI and document state [source: [fusion-threading-discussion]].
```

If a claim is from an experiment or smoke test you ran:

```markdown
[empirical] In a smoke test on 2026-05-15 (`scripts/smoke/01-hello-fusion.py`), the add-in loaded successfully and `app.activeDocument` returned the expected `Document` object.
```

If a claim is plausibly true but you couldn't verify in a reasonable amount of time:

```markdown
[unverified] (claim text) — verification deferred, see `inbox/` entry `verify-...`.
```

**Never** silently make a claim without one of these three markers (citation, `[empirical]`, or `[unverified]`).

---

## Entry templates

Templates live in `vault/00-meta/_templates/`. Use the right one for the entry type. Templates are starting points, not straitjackets — but every entry has the same load-bearing sections:

| Section | Required? | Purpose |
|---|---|---|
| Frontmatter (title, category, tags, status) | Yes | Obsidian metadata |
| One-sentence summary | Yes | First line after the title |
| `## First principles` | Yes (except `09-sources/`) | What is this REALLY |
| `## Findings` or `## Surface` | Yes | The substantive content |
| `## Implications for adze` | Yes (except `09-sources/`) | Why this matters to us |
| `## Open questions` | Optional | What we still don't know |
| `## Sources` | Yes | Citation block |
| `## Related` | Yes | `[[wikilinks]]` to 2-6 sibling entries |

---

## Length and atomization

- An entry's body (excluding sources) should be **under 600 words**. If it's longer, atomize.
- One concept per entry. Two concepts that travel together get linked, not merged.
- No prose dumps. Headings + lists + tables over paragraphs.
- Code samples inline only if they are load-bearing for the explanation — otherwise link to a gist or repo.

---

## Inbox discipline

`vault/inbox/` is the staging area. Rules:

1. Inbox is for **unsorted capture during research**. Notes, quotes, half-formed ideas.
2. Inbox is drained **every session** — by the end of session, inbox is empty.
3. Drain options: **promote** to a category (clears the bar), **merge** into an existing entry, **delete** (didn't pan out), or **defer to user** (a judgment call you can't make alone — surface explicitly before close).
4. If something is "interesting but not load-bearing," it goes in your `[idea]` notes in brain.db, not the vault.

---

## Cross-linking discipline

- Every promoted entry gets a `## Related` block with 2–6 `[[wikilinks]]`.
- A new entry that is referenced from an existing entry — back-link the existing entry too. Reciprocal links are the rule.
- Stub `[[wikilinks]]` to entries that don't exist yet are **OK** — they mark intent. Obsidian renders them differently (gray) so they're visible.
- Don't manually maintain index pages. Obsidian's graph view + backlinks panel are the navigation.

---

## Stable category boundaries

The taxonomy in `ONTOLOGY.md` is **stable**. Three rules:

1. New categories are recorded ADRs, not silent folders.
2. Renaming categories is an ADR.
3. Moving an entry between categories is fine if the original was a misfit — just update incoming links.

If you find yourself wanting to invent `vault/11-misc/` or `vault/random-thoughts/`, **stop**. The misfit is signaling either an atomization need (split into 2+ entries) or a real ontology change (ADR).

---

## Forbidden patterns

- ❌ "Various sources confirm…" — name them or drop the claim
- ❌ "I think…" or "It seems likely that…" — first-principles, not vibes
- ❌ "TODO: verify later" without an inbox entry — half-finished work doesn't live in the vault
- ❌ Files in category folders without `## Related` blocks
- ❌ Files longer than 600 words (excluding sources)
- ❌ Promoting findings directly from agent output without triage — agent output goes to `research/findings/`, then atomized into vault entries

---

## When in doubt

Ask the user. The user owns ontology changes, scope decisions, and "should this be in the vault at all" judgment calls when you can't decide. Don't silently bend the rules to fit a finding.
