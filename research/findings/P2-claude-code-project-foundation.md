# P2 — Claude Code Project Foundation — Best Practices

**Date:** 2026-05-15 (research conducted 2026-05-26, dated per parent stream protocol)
**Researcher:** general-purpose sub-agent
**Target consumer:** `C:\adze-fusion` project setup

## Summary

A Claude Code "project" is the directory you launch `claude` from, plus the `.claude/` subdirectory and any `CLAUDE.md` files in that directory or its ancestors. Five surfaces drive behavior: **CLAUDE.md** (persistent instructions, ~200 lines max, loaded into every session), **`.claude/settings.json`** (permissions, hooks, env, MCP gating — shareable via git), **`.claude/settings.local.json`** (per-machine permission additions, gitignored), **`.claude/skills/`** (lazy-loaded procedures), and **`.claude/agents/`** (custom sub-agents). Hooks are the only deterministic enforcement layer — CLAUDE.md is context, not policy. For an "agent-as-directory" project like adze-fusion, the optimal setup is: tight CLAUDE.md (the role/mission), aggressive `SessionStart` hook (inject identity from brain.db spoke), narrow permission allowlist (research workflow tools), a `dal` MCP server declared in `.mcp.json`, and 2–4 focused custom skills. Avoid: bloated CLAUDE.md, orphaned hooks, broad `bypassPermissions`.

## First principles

A Claude Code project is **not a framework** — it's a discovery convention. When `claude` launches in a directory, it:

1. Walks up from cwd loading every `CLAUDE.md` / `CLAUDE.local.md` it finds, concatenating them in filesystem-root-to-cwd order so the most local one comes last [Memory docs — fetched 2026-05-26](https://code.claude.com/docs/en/memory). "All discovered files are concatenated into context rather than overriding each other."
2. Merges `settings.json` from four scopes (managed → user `~/.claude/settings.json` → project `.claude/settings.json` → local `.claude/settings.local.json`) with **deny > ask > allow** precedence [Settings docs — fetched 2026-05-26](https://code.claude.com/docs/en/settings).
3. Discovers `.claude/agents/`, `.claude/skills/`, `.claude/commands/`, `.claude/rules/` recursively from cwd up to the repo root.
4. Connects MCP servers declared in `.mcp.json` (project-scoped, git-committed) plus user-scope servers from `~/.claude.json` [MCP docs — fetched 2026-05-26](https://code.claude.com/docs/en/mcp).
5. Fires the `SessionStart` hook before the first user prompt, with the hook free to inject `additionalContext`.

So "the project IS the agent" is achieved by combining: (a) a CLAUDE.md that declares the role, (b) a `SessionStart` hook that injects fresh identity/state, (c) skills that ARE the agent's procedures, and (d) a permission allowlist tuned to the agent's job. Nothing else is required.

## The `.claude/` directory — complete reference

Layout used by mature projects (mirroring `C:\adze-cad\.claude\`):

```
.claude/
├── CLAUDE.md           # optional — alternative to project-root CLAUDE.md
├── settings.json       # git-committed: permissions, hooks, env, MCP gating
├── settings.local.json # gitignored: per-machine permissions
├── agents/             # custom sub-agent definitions (*.md w/ frontmatter)
├── hooks/              # scripts referenced from settings.json hooks
├── skills/             # project-local skills (<name>/SKILL.md)
├── commands/           # legacy slash-command files (now merged into skills)
├── rules/              # path-scoped behavior rules (*.md w/ paths: frontmatter)
└── memory/             # auto-memory store (if autoMemoryDirectory points here)
```

### `settings.json` (project-shared, git-committed)

Top-level keys you will actually use [Settings docs — fetched 2026-05-26](https://code.claude.com/docs/en/settings):

| Key | Purpose |
|---|---|
| `permissions` | `allow` / `ask` / `deny` arrays + `defaultMode` + `additionalDirectories` |
| `hooks` | Event handlers — full list under Hooks below |
| `env` | Environment variables applied to every session and subprocess |
| `model` | Default model (read once at startup; use `/model` to switch mid-session) |
| `enableAllProjectMcpServers` | Auto-accept all servers in `.mcp.json` without per-server approval |
| `enabledMcpjsonServers` / `disabledMcpjsonServers` | Subset of `.mcp.json` servers to load |
| `autoMemoryEnabled` / `autoMemoryDirectory` | Auto-memory toggle and storage path |
| `statusLine` | Command producing a custom status line string |
| `outputStyle` | `"Explanatory"` etc. — read once at startup, modifies system prompt |
| `respectGitignore` | Default true — controls file discovery |
| `claudeMdExcludes` | Glob patterns for ancestor CLAUDE.md files to skip (monorepos) |
| `$schema` | `"https://json.schemastore.org/claude-code-settings.json"` for IDE autocomplete |

Live-reloaded keys: `permissions`, `hooks`, `apiKeyHelper`. Read-once at startup: `model`, `outputStyle`. [Settings docs — fetched 2026-05-26](https://code.claude.com/docs/en/settings).

### `settings.local.json` (per-machine, gitignored)

Same schema as `settings.json` but never committed. Use for: extra `permissions.allow` rules the developer trusts (extends, doesn't override the project allowlist), machine-specific MCP server choices, `skillOverrides` to silence skills, local `env` overrides. The Claude Code `/permissions` UI writes "Yes, don't ask again" decisions here automatically [Settings docs — fetched 2026-05-26](https://code.claude.com/docs/en/settings).

### `agents/` — custom sub-agents

Markdown files with YAML frontmatter [Sub-agents docs — fetched 2026-05-26](https://code.claude.com/docs/en/sub-agents). Required: `name`, `description`. Optional: `tools`, `disallowedTools`, `model` (`sonnet`/`opus`/`haiku`/`inherit` or full ID like `claude-opus-4-7`), `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory` (`user`/`project`/`local`), `effort`, `isolation: worktree`, `color`, `initialPrompt`, `background`. The body is the system prompt. Quoted: "Subagents are loaded at session start. If you add or edit a subagent file directly on disk, restart your session to load it."

Built-in sub-agents to know about: **Explore** (Haiku, read-only, fast codebase search), **Plan** (read-only, used during plan mode), **general-purpose** (all tools — what you spawn via `Agent` for deep research). Explore and Plan skip CLAUDE.md and git status to stay cheap.

### `hooks/` — backing scripts

Convention: `.claude/hooks/<name>.js` (or `.sh`, `.ps1`). Referenced by absolute path or `$CLAUDE_PROJECT_DIR/.claude/hooks/<name>.js` from `settings.json`. Hooks receive JSON via stdin and can emit JSON on stdout for blocking/context injection (see Hooks section).

### `skills/` — project-local skills

Each skill is a directory `<skill-name>/` containing at least `SKILL.md`. The directory name becomes the slash-command (`.claude/skills/triage/SKILL.md` → `/triage`). Supporting files (reference docs, scripts) live next to `SKILL.md` and are referenced from it [Skills docs — fetched 2026-05-26](https://code.claude.com/docs/en/skills).

### `commands/` — legacy

Quoted from skills docs: "**Custom commands have been merged into skills.** A file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way." Prefer `skills/` for new work — only it supports supporting files and the full frontmatter [Skills docs — fetched 2026-05-26](https://code.claude.com/docs/en/skills).

### `rules/` — path-scoped instructions

Markdown files with optional `paths:` frontmatter glob. Rules without `paths` always load (same priority as `.claude/CLAUDE.md`); rules with `paths` only enter context when Claude reads matching files [Memory docs — fetched 2026-05-26](https://code.claude.com/docs/en/memory). Useful for breaking a giant CLAUDE.md into topic files.

## Hooks — complete reference

The doc lists **29 hook events** [Hooks docs — fetched 2026-05-26](https://code.claude.com/docs/en/hooks). Configure them under `settings.json` `hooks` as `{ EventName: [{ matcher: "...", hooks: [{ type: "command", command: "...", timeout: 30 }]}] }`.

**Most-used for a project:**

| Event | Matcher | When it fires | Typical use |
|---|---|---|---|
| `SessionStart` | `startup`/`resume`/`clear`/`compact` | Session begins | Inject identity + open notes from brain.db (most important hook for adze-fusion) |
| `UserPromptSubmit` | none | User submits a prompt | Reject prompts; add context; auto-title sessions. 30s timeout (shorter than others) |
| `PreToolUse` | tool name | Before tool call | Block dangerous Bash, gate writes by file path, validate inputs |
| `PostToolUse` | tool name | After tool call succeeds | Auto-lint/format, refresh gitnexus index, append context |
| `Notification` | `permission_prompt`/`idle_prompt`/... | Claude notifies | Terminal beep, desktop notification |
| `Stop` | none | Claude finishes responding | Auto-export session, closeout reminder, completion check |
| `PreCompact` / `PostCompact` | `manual`/`auto` | Around context compaction | Save state, re-inject critical context |
| `SubagentStart` / `SubagentStop` | agent type | Sub-agent lifecycle | Cost tracking, audit |
| `InstructionsLoaded` | load reason | CLAUDE.md or rule loaded | Debug what context was actually pulled in |

**Exit code semantics** [Hooks docs — fetched 2026-05-26](https://code.claude.com/docs/en/hooks):
- Exit `0`: success. Claude Code parses stdout for JSON.
- Exit `2`: blocking error. stderr surfaces as the reason. For `PreToolUse` this denies the tool call; for `UserPromptSubmit` rejects the prompt.
- Other: non-blocking error, first stderr line shown.

**JSON output schema** (any event):
```json
{
  "continue": false,
  "stopReason": "...",
  "suppressOutput": true,
  "systemMessage": "...",
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Branch: main\nOpen notes: 3",
    "permissionDecision": "allow|deny|ask|defer",
    "modifiedInput": { ... }
  }
}
```

**SessionStart pattern** (the heart of adze-fusion's "agent" identity): match `startup|resume`, run a node/bash script that reads brain.db, emits `additionalContext` with role + identity + open notes + recent decisions. The `C:\adze-cad\.claude\hooks\session-context.js` is a working reference: it reads git state, walks `.ava/brain.db`, and prints context lines [Local inspection — 2026-05-26].

**Common gotcha:** Stdout JSON is **only parsed when exit code is 0**. If your hook script errors and exits non-zero, the JSON is dropped and stderr shows as a warning.

## Permissions

Rules follow `Tool` or `Tool(specifier)`. Evaluation order is **deny → ask → allow**, first match wins [Permissions docs — fetched 2026-05-26](https://code.claude.com/docs/en/permissions). Deny rules from any scope beat allow rules from any scope.

**Specifier syntax cheat sheet:**
- `Bash(npm run *)` — prefix match with space boundary
- `Bash(npm:*)` — equivalent trailing wildcard form (only valid at end)
- `Read(./.env)` — file relative to cwd
- `Read(/src/**/*.ts)` — relative to project root (single `/` = project-relative)
- `Read(//Users/me/secrets/**)` — absolute (double `//`)
- `Read(~/.zshrc)` — home-relative
- `WebFetch(domain:github.com)` — domain match
- `mcp__servername` — all tools on that MCP server
- `mcp__servername__toolname` — specific MCP tool
- `Agent(Explore)` — a sub-agent type
- `Skill(triage)` — a skill (use to pre-approve skills)

**`defaultMode` values:** `default` (prompt on first use), `acceptEdits` (auto-accept edits + common filesystem ops), `plan` (read-only exploration), `auto` (background classifier — research preview), `dontAsk` (auto-deny unless explicitly allowed), `bypassPermissions` (skip everything, dangerous).

**`additionalDirectories`** in `permissions` extends Claude's file access to other paths but **does NOT load CLAUDE.md, hooks, agents, or commands from those paths** (skills are an exception only when added via `--add-dir`/`/add-dir` flag, not the setting). Useful for grant-file-access-only [Permissions docs — fetched 2026-05-26](https://code.claude.com/docs/en/permissions).

**Built-in read-only commands** (never prompt regardless of mode): `ls`, `cat`, `echo`, `pwd`, `head`, `tail`, `grep`, `find`, `wc`, `which`, `diff`, `stat`, `du`, `cd`, and read-only `git` forms.

**Pattern for adze-fusion** (research/curation agent): allow read tooling broadly, allow WebFetch to research domains, allow `Bash(node .ava/dal.mjs:*)` for brain.db CLI, deny destructive shells, leave `Write`/`Edit` on `ask` for documents the agent curates.

## MCP server configuration

Three scopes [MCP docs — fetched 2026-05-26](https://code.claude.com/docs/en/mcp):

| Scope | File | Shared? |
|---|---|---|
| local | `~/.claude.json` (per project entry) | No |
| project | `.mcp.json` at project root | **Yes, via git** |
| user | `~/.claude.json` (user level) | No |

`.mcp.json` is **the** project-required-MCP declaration:

```json
{
  "mcpServers": {
    "dal": {
      "type": "stdio",
      "command": "node",
      "args": ["${CLAUDE_PROJECT_DIR:-.}/.ava/mcp-server.mjs"]
    },
    "gitnexus": { "type": "stdio", "command": "npx", "args": ["-y", "gitnexus"] }
  }
}
```

Env-var expansion uses `${VAR}` and `${VAR:-default}` [MCP docs — fetched 2026-05-26](https://code.claude.com/docs/en/mcp). `${CLAUDE_PROJECT_DIR}` is set by Claude Code in the spawned server's env.

To auto-accept project servers without per-server approval, set `enableAllProjectMcpServers: true` in `settings.json`. To enable a subset, set `enabledMcpjsonServers: ["dal", "gitnexus"]`. Project-scoped servers from `.mcp.json` still prompt on first project load for security; `claude mcp reset-project-choices` resets the trust dialog.

Tool Search is on by default and defers MCP tool descriptions until needed; for tools your agent uses every turn, set `alwaysLoad: true` on that server entry [MCP docs — fetched 2026-05-26](https://code.claude.com/docs/en/mcp).

## Skills

A skill is a directory with `SKILL.md` plus optional supporting files. Frontmatter [Skills docs — fetched 2026-05-26](https://code.claude.com/docs/en/skills):

| Field | Use |
|---|---|
| `description` | Recommended — Claude reads this to decide when to auto-invoke. Truncated at 1,536 chars with `when_to_use`. |
| `when_to_use` | Trigger phrases / examples, appended to description |
| `disable-model-invocation: true` | Only the user can invoke (e.g. `/deploy`) |
| `user-invocable: false` | Only Claude can invoke (background knowledge) |
| `allowed-tools` | Pre-approves these tools while skill is active |
| `argument-hint`, `arguments` | Define args; reference with `$ARGUMENTS`, `$0`, `$1`, or named `$issue` |
| `context: fork` + `agent: Explore` | Run the skill in a fresh sub-agent context |
| `paths` | Glob — only auto-load when working in matching files |
| `model`, `effort` | Override for this skill's turn |

Quoted: "Create a skill when you keep pasting the same instructions, checklist, or multi-step procedure into chat, or when a section of CLAUDE.md has grown into a procedure rather than a fact. Unlike CLAUDE.md content, a skill's body loads only when it's used, so long reference material costs almost nothing until you need it."

**Decision rule** [Skills docs — fetched 2026-05-26](https://code.claude.com/docs/en/skills):
- Use **CLAUDE.md** for: facts always in scope (build commands, conventions, project layout).
- Use a **skill** for: procedures, checklists, on-demand reference (e.g. `/triage`, `/research-domain`).
- Use a **sub-agent** for: a side task that would flood the main context (use Explore or define custom).
- Use a **slash command** = use a skill (commands are deprecated alias).

**Invocation:** `/skill-name [args]` invokes directly. Claude can also auto-invoke based on `description`. The `Skill` tool is how Claude programmatically invokes them. User-level skills (`~/.claude/skills/`) override project skills if names collide.

**Skill content lifecycle:** "When you or Claude invoke a skill, the rendered SKILL.md content enters the conversation as a single message and stays there for the rest of the session." Keep the body short — every line is a recurring token cost.

## Sub-agents

Project-level sub-agents live in `.claude/agents/<name>.md`. User-level in `~/.claude/agents/`. The frontmatter table in the `.claude/` section above is the full surface [Sub-agents docs — fetched 2026-05-26](https://code.claude.com/docs/en/sub-agents).

**When to define a custom one (vs. just spawning `general-purpose` via the `Agent` tool):**
- You spawn the same kind of worker repeatedly with the same instructions
- You want a restricted toolset (e.g. read-only researcher)
- You want a specific model (Haiku for cheap, Opus for hard)
- You want persistent memory across sessions (`memory: project` → `.claude/agent-memory/<name>/`)

**Restrict sub-agent spawning:** When an agent runs as the main thread via `claude --agent`, use `tools: Agent(worker, researcher)` to allowlist which sub-agent types it can spawn.

**Forked-context pattern:** A skill with `context: fork` runs in a sub-agent. A sub-agent with `skills: [...]` preloads skills as reference material. These are complementary [Skills docs — fetched 2026-05-26](https://code.claude.com/docs/en/skills).

## Slash commands

Treat as legacy alias for skills [Commands docs — fetched 2026-05-26](https://code.claude.com/docs/en/commands). The full skill frontmatter set works in `.claude/commands/<name>.md`. New work should use `.claude/skills/<name>/SKILL.md`. Built-in commands you care about: `/init` (scaffold CLAUDE.md), `/memory` (browse and toggle auto-memory), `/agents` (manage sub-agents), `/mcp` (server status + auth), `/permissions` (rule UI), `/plan`, `/model`, `/effort`, `/context`, `/compact`, `/btw`, `/batch`, `/loop`, `/run`, `/verify`.

## brain.db / DAL integration

**[inferred]** Based on local inspection of `C:\adze-cad\.ava\` (the DAL spoke for the existing project) and the user's CLAUDE.md mention of `~/.claude/.ava/dal.mjs` (the hub) [Local inspection — 2026-05-26]:

- The hub-and-spoke pattern: a global hub at `~/.claude/.ava/` (or wherever the user installed `dal.mjs`) and a per-project spoke at `<project>/.ava/` with its own `brain.db`, `mcp-server.mjs`, `dal.mjs`, `handoffs/`, `lib/`, `migrations/`.
- The CLI surface is `node .ava/dal.mjs <command>` — verified commands from the existing project's settings.local.json: `dal_session_start`, `dal_session_close`, `dal_session_export`, `dal_note_add`, `dal_note_list`, `dal_note_complete`, `dal_decision_add`, `dal_decision_list`, `dal_handoff_generate`, `dal_continuity_brief`, `dal_status`, `dal_trace_add`, `dal_identity_get`, `dal_identity_set`.
- Each spoke ships its own MCP server (`mcp-server.mjs`). Declaring it in `.mcp.json` makes the MCP tools `mcp__dal__dal_note_add` etc. available — the existing project does exactly this.

**Spoke init for adze-fusion** (recommended pattern):
1. Run `node ~/.claude/.ava/dal.mjs init` (or equivalent) inside `C:\adze-fusion\` to scaffold `.ava/brain.db`.
2. Run `dal_identity_set` to define the agent's role (e.g. `role: research-curation-agent`).
3. Declare the local MCP server in `.mcp.json` so its tools are available in-session.
4. SessionStart hook calls `dal_continuity_brief` (or reads identity + open notes directly) and emits as `additionalContext`.

The session-end pattern: a `Stop` hook calls `dal_session_export` to persist what happened [Local inspection of existing project hook setup — 2026-05-26].

## Knowledge layer integration

**graphify** (per user CLAUDE.md and the project's CLAUDE.md): produces `graphify-out/GRAPH_REPORT.md` + a wiki + a queryable graph. Rules from the existing CLAUDE.md: "Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md... For cross-module 'how does X relate to Y' questions, prefer `graphify query`... After modifying code files in this session, run `graphify update .`". Integration is via the `graphify` bundled skill and a `PostToolUse` hook (existing project has one).

**gitnexus** (per project CLAUDE.md): provides MCP tools (`gitnexus_context`, `gitnexus_impact`, `gitnexus_detect_changes`, `gitnexus_rename`, `gitnexus_cypher`, `gitnexus_query`). It maintains an AST-derived knowledge graph at `.gitnexus/`. A `PostToolUse(Bash)` hook detects `git commit` and runs `npx gitnexus analyze` to refresh (see `C:\adze-cad\.claude\hooks\gitnexus-post-commit.js`). A `PreToolUse(Edit|Write)` hook runs impact analysis before edits (`gitnexus-impact-check.js`).

For a research/curation agent (adze-fusion) that's not primarily editing code: gitnexus is **less load-bearing**. graphify is more useful if the curation produces an interconnected knowledge artifact. Both are optional.

## Session lifecycle

The user's existing pattern from CLAUDE.md and `C:\Users\Kaden\.claude\skills\`:

- **`/session-init`** — at session start: read rules, synthesize continuity, verify runtime health, orient before work. `--auto-dev` flag for autonomous execution.
- **`/session-closeout`** — at session end: persist continuity, update changed docs, ensure next session can resume.
- **Continuity state** lives in brain.db (notes, decisions, identity, handoffs). The file tree holds longer-form artifacts (plans, findings).
- **Handoff YAML** [inferred from `<project>/.ava/handoffs/`] — structured snapshot generated by `dal_handoff_generate`, read by next session.

For adze-fusion: clone the pattern. Bundle a project-local `session-init` skill if the workflow differs from the user-level one (e.g. research-specific orientation).

## CLAUDE.md best practices

**Doc-stated rules** [Memory docs — fetched 2026-05-26](https://code.claude.com/docs/en/memory):
- Target **under 200 lines**. Longer files consume more context AND **reduce adherence** (Claude pays less attention to bloated files).
- Markdown headers + bullets; specific not aspirational ("Use 2-space indentation" beats "format properly").
- "If two rules contradict each other, Claude may pick one arbitrarily."
- Block-level HTML comments `<!-- ... -->` are stripped before injection — free maintainer notes.
- `/init` scaffolds a starter file; iterate from there.
- Use `@path/to/file` imports for organization; **imports still load fully at launch**, they don't reduce context — only `.claude/rules/` with `paths:` frontmatter actually lazy-loads.

**Hierarchy & load order** (root → cwd, then `CLAUDE.local.md` last per directory):
1. Managed policy `CLAUDE.md` (org-wide, cannot be excluded)
2. User `~/.claude/CLAUDE.md`
3. Project `CLAUDE.md` (or `.claude/CLAUDE.md`)
4. Project `CLAUDE.local.md` (gitignored)

**Community-stated rules** [TECHSY 2026 — fetched 2026-05-26](https://techsy.io/en/blog/claude-md-best-practices), [ClaudeCodeLab — fetched 2026-05-26](https://claudecode-lab.com/en/blog/claude-md-best-practices/):
- 6–10 real rules with **reasons** ("explain the why — it's how Claude decides edge cases").
- 3 commands Claude should know.
- 2 anti-patterns your team has hit.
- Test in a fresh session: ask Claude to summarize the file.

**For adze-fusion specifically** (agent-as-directory): CLAUDE.md should be the **role declaration**. Identity (you are X), mission (you do Y), boundaries (you don't do Z), the 5–10 tools you reach for, the brain.db / DAL integration points. Procedures go in skills, not CLAUDE.md.

## Sub-agent orchestration basics (full deep-dive in P4)

Three modes [Sub-agents docs — fetched 2026-05-26](https://code.claude.com/docs/en/sub-agents):

1. **Built-in delegation** — Claude decides when to use Explore/Plan/general-purpose. Free for the project; tune by writing CLAUDE.md hints like "use Explore for...".
2. **Custom typed delegation** — Define `.claude/agents/research-domain.md` with clear `description`; Claude routes to it.
3. **Skill-fork pattern** — `context: fork` + `agent: Explore` runs a skill in an isolated read-only context.

For research agents: **parallel dispatch** is the killer pattern. Claude can spawn multiple `general-purpose` sub-agents in one message; they execute concurrently and return summaries that don't pollute the main context window. P4 covers this.

## Project hygiene

**`.gitignore` for `.claude/`:**
```
# Per-machine settings + permission additions
.claude/settings.local.json

# Hook logs and ephemeral state
.claude/hooks/hook-log.jsonl
.claude/hooks/*.log

# Local-only memory (if you keep memory in-tree)
.claude/agent-memory-local/

# Worktree / paste cache
.claude/paste-cache/
.claude/file-history/

# Brain.db spoke (handoffs / backups stay local)
.ava/handoffs/
.ava/backups/
.ava/brain.db.bak-*

# DO commit:
# .claude/settings.json
# .claude/agents/*.md
# .claude/hooks/*.js (the scripts themselves, not their logs)
# .claude/skills/**
# .claude/rules/*.md
# .mcp.json
# CLAUDE.md
```

**Never commit:** `.credentials.json`, `.env`, OAuth tokens, anything from `~/.claude/sessions/` or `~/.claude/projects/<project>/`.

## Anti-patterns and footguns

From doc + community sources:

- **Bloated CLAUDE.md** [TECHSY 2026 — fetched 2026-05-26](https://techsy.io/en/blog/claude-md-best-practices): "If your CLAUDE.md is too long, Claude ignores half of it because important rules get lost in the noise. Fix: ruthlessly prune."
- **CLAUDE.md as policy** [Memory docs — fetched 2026-05-26](https://code.claude.com/docs/en/memory): "Claude treats them as context, not enforced configuration. To block an action regardless of what Claude decides, use a PreToolUse hook instead."
- **Conflicting CLAUDE.md rules across the hierarchy** — Claude picks one arbitrarily. Periodically reconcile user + project + parent files.
- **Skill drift** — accumulating dead skills bloats the `/` menu and adds description tokens to every session. Audit with `/agents`-style review.
- **Hooks that exit non-zero accidentally** — drop their JSON output silently. Always exit 0 unless you mean to block.
- **`bypassPermissions` defaultMode** — quoted warning: skips writes to `.git`, `.claude`, `.vscode`, `.idea`, `.husky`. Containers/VMs only.
- **Permissions allowlist sprawl** — `settings.local.json` accretes "yes don't ask again" entries forever. Audit and consolidate to `settings.json` patterns periodically. The existing adze-cad project has 150+ allow entries from this drift.
- **MCP server description bloat** — many connected MCPs each contribute tool definitions to context. Tool Search defers them; verify with `/mcp`.
- **Orphaned hooks** — a hook script referenced from `settings.json` but missing on disk fails silently on each event. Inventory hooks during cleanup.
- **`additionalDirectories` ≠ extra config root** — only `--add-dir` / `/add-dir` loads skills from there; the setting only grants file access.
- **Imports don't save tokens** — `@path/to/file` in CLAUDE.md still loads fully. Use `paths:` rules to actually lazy-load.

## Decision implications for adze-fusion

Concrete actions for `C:\adze-fusion\.claude\`:

1. **Tight CLAUDE.md (target ~150 lines)** — role, mission, boundaries, the 5 core tools, brain.db spoke pointer, "always do X / never do Y", and a Quick Reference. Procedures go in skills.
2. **SessionStart hook** that calls `dal_continuity_brief` (or reads brain.db identity + open notes directly) and emits `additionalContext`. Mirror `C:\adze-cad\.claude\hooks\session-context.js` shape.
3. **`.mcp.json` declares the `dal` MCP server** for the adze-fusion spoke. Add `enableAllProjectMcpServers: true` in `settings.json` so first run doesn't prompt.
4. **Narrow `permissions.allow`** for the research workflow: `WebFetch(domain:*)` for the domains you'll research (start narrow, widen as needed), `Bash(node .ava/dal.mjs:*)`, `Bash(npx gitnexus:*)` if used, `Read` broadly, `Write`/`Edit` left on `ask` so curated artifacts get a confirm.
5. **2–4 custom skills**: `/research-topic`, `/curate-source`, `/synthesize`, `/handoff-prep`. Keep each SKILL.md under 100 lines; use `arguments:` and `$ARGUMENTS` for parameterization.
6. **Stop hook** that calls `dal_session_export` and warns if uncommitted curation artifacts exist.
7. **No custom sub-agents at v0** — start with built-in Explore + general-purpose. Add custom agents only when you've spawned the same shape twice.
8. **gitnexus optional, graphify yes** — adze-fusion is a curation product, so the knowledge graph view of the corpus matters more than the code-impact view. Wire up graphify in PostToolUse if curated files start cross-referencing.
9. **`autoMemoryDirectory: ".claude/memory"`** to keep auto-memory inside the project (gitignore the dir if it shouldn't be shared) — matches what adze-cad does.
10. **No `bypassPermissions`**. Use `defaultMode: "default"` or `"acceptEdits"` only after a few sessions confirm the agent's pattern is safe.

## Open questions

- **DAL hub location** — user's CLAUDE.md says `~/.claude/.ava/dal.mjs` but local inspection found only `<project>/.ava/`. Either the hub doesn't exist on this machine or it's under a different path. The "hub-and-spoke" framing may be a user mental model rather than a literal file split. **[unverified]**
- **`agent` frontmatter / `claude --agent`** — exists in docs but the operational semantics (does adze-fusion launch with `claude --agent <name>` to BE the agent vs. just opening in the directory?) need confirmation. The directory-as-agent pattern works via CLAUDE.md + SessionStart hook even without `--agent`.
- **Skill `paths:` glob behavior in research workflows** — docs say rules use `paths:` for lazy loading; whether skills use the same field equivalently for auto-invocation gating wasn't fully testable from docs. **[unverified]**
- **Persistence across `/compact`** — only project-root CLAUDE.md is guaranteed re-injected. Whether SessionStart hook content re-runs after compact is unclear; likely no (it's `SessionStart`, not `PostCompact`). May need a PostCompact hook to re-inject identity. **[unverified]**

## Sources

Anthropic / Claude official docs (all fetched 2026-05-26):

- [Settings reference](https://code.claude.com/docs/en/settings)
- [Hooks reference](https://code.claude.com/docs/en/hooks)
- [Memory / CLAUDE.md](https://code.claude.com/docs/en/memory)
- [Sub-agents](https://code.claude.com/docs/en/sub-agents)
- [Skills](https://code.claude.com/docs/en/skills)
- [MCP](https://code.claude.com/docs/en/mcp)
- [Permissions](https://code.claude.com/docs/en/permissions)
- [Commands](https://code.claude.com/docs/en/commands)
- [Authentication](https://code.claude.com/docs/en/authentication)

Community (fetched 2026-05-26):

- [Best practices for Claude Code (Anthropic)](https://code.claude.com/docs/en/best-practices)
- [CLAUDE.md Best Practices: 9 Rules for 2026 — TECHSY](https://techsy.io/en/blog/claude-md-best-practices)
- [Claude Code Best Practices: Lessons From Real Projects — RanTheBuilder](https://ranthebuilder.cloud/blog/claude-code-best-practices-lessons-from-real-projects/)
- [The Complete Guide to Writing CLAUDE.md — ClaudeCodeLab](https://claudecode-lab.com/en/blog/claude-md-best-practices/)
- [Claude Code Best Practices — rosmur GitHub Pages](https://rosmur.github.io/claudecode-best-practices/)
- [Claude Code Best Practices: 15 Tips from 6 Projects — aiorg.dev](https://aiorg.dev/blog/claude-code-best-practices)

Local inspection (2026-05-26):

- `C:\adze-cad\.claude\settings.json` — working hook/permission patterns
- `C:\adze-cad\.claude\settings.local.json` — drift example (150+ allow entries)
- `C:\Users\Kaden\.claude\settings.json` — user-level baseline
- `C:\adze-cad\.claude\hooks\session-context.js` — SessionStart reference impl
- `C:\adze-cad\.claude\agents\` — three working custom agents (closeout-worker, doc-validator, security-reviewer)
- `C:\adze-cad\.claude\skills\` — 35+ skill directories
- `C:\adze-cad\.ava\` — DAL spoke layout (brain.db, dal.mjs, mcp-server.mjs, handoffs/, lib/, migrations/)
- `C:\adze-cad\CLAUDE.md` — production CLAUDE.md (~370 lines — note: violates the "<200 lines" guidance, an example of allowable drift in a complex project)
