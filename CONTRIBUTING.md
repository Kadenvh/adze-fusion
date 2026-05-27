# Contributing to adze-fusion

Thanks for your interest. adze-fusion is a **research and architecture project** for AI integration with Autodesk Fusion 360 (and eventually the broader multi-CAD adze product line). Most of the work right now is curation of a knowledge vault, not code.

Before contributing, read these three files in order:

1. **`CLAUDE.md`** — what the project is, how the agent operates
2. **`vault/00-meta/VAULT-RULES.md`** — quality bar for vault entries
3. **`vault/00-meta/ONTOLOGY.md`** — taxonomy / where things live

## Project structure

Three-layer architecture per Karpathy's LLM-Wiki pattern:

- **`raw/`** — immutable source documents. Agent reads, never modifies. Add new sources here.
- **`vault/`** — LLM-curated wiki. The artifact. Atomized markdown entries with frontmatter, cross-links, source rigor.
- **`CLAUDE.md`** — master schema (the agent's operating manual).

Plus:
- **`research/`** — research charter + per-stream findings (pre-vault)
- **`plans/`** — staged dispatch plans (Stage 1, future stages)
- **`.github/`** — issue templates, PR template

See `README.md` for the full layout.

## Ways to contribute

### A) Add a research finding or source

If you have a high-signal source (paper, blog, repo, talk) that should inform the project:

1. Drop the source file or a fetched copy into `raw/` with a date-stamped filename (e.g. `raw/<topic>-<YYYY-MM-DD>.md`).
2. Open an issue using the **"Source proposal"** template — describe what the source contributes and where it should map in the vault ontology.
3. The agent (or maintainer) reviews and atomizes the source into the vault during a future ingest operation.

Do NOT directly create `vault/` category entries from raw sources without going through the ingest workflow.

### B) Flag a contradiction

If two vault entries make conflicting claims:

1. Open an issue using the **"Contradiction"** template.
2. Cite both entries by `[[wikilink]]`.
3. Propose a resolution if you can, or leave it open for research dispatch.

### C) Verify an `[unverified]` claim

If you have evidence that supports or refutes a vault claim marked `[unverified]`:

1. Open an issue using the **"Verification"** template.
2. Cite the source you used to verify.
3. The agent updates the entry's `confidence` field and removes the `[unverified]` marker on next ingest.

### D) Propose a research stream

If you think there's a gap in research coverage:

1. Open an issue using the **"Research request"** template.
2. Specify objective, questions, success criteria, source-carve scope.
3. The agent dispatches the stream and atomizes findings into the vault.

## Code contributions

Not yet. adze-fusion is pre-Stage-4. There's no code to contribute to. Once Stage 4 (narrow prototype) lands, this section will be expanded with build / test / coding conventions.

## Quality bar for vault entries

Every promoted vault entry must clear four points (see `vault/00-meta/VAULT-RULES.md`):

1. **First-principles** — describes what the thing actually IS, not marketing
2. **Sourced** — every factual claim cites a source, is marked `[empirical]`, or `[unverified]`
3. **Replicable** — a future agent or human picks it up cold and reconstructs the picture
4. **Load-bearing** — a future decision (architecture, prioritization, scope) leans on it

Entries that don't clear the bar stay in `vault/inbox/` until they do, or get deleted.

## Code of conduct

Be respectful and constructive. The project values rigorous research over fast opinions.

## License

MIT — see `LICENSE`. Contributions are accepted under the same license.
