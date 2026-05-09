# Architecture — obsidian-wiki

## Executive Summary

obsidian-wiki is an **instruction-based agent framework**. It has no runtime of its own — it's a curated bundle of markdown skills plus a single Bash installer (`setup.sh`) that makes those skills discoverable to any AI coding agent. The agent is the executor; the skills are the program; the user's Obsidian vault is the persistent store.

The framework operates on three layers:

1. **Source** — raw inputs the user wants to remember (chat histories, documents, URLs, project repos, transcripts).
2. **Wiki** — compiled, interconnected markdown pages with frontmatter and `[[wikilinks]]`, organized into category folders inside the user's Obsidian vault.
3. **Maintenance loop** — skills that ingest new sources, query the compiled wiki, lint for drift, synthesize cross-cutting analyses, and keep `index.md`, `log.md`, `hot.md`, and `.manifest.json` current.

## Technology Stack

| Category | Technology | Version | Justification |
|---|---|---|---|
| Skill format | Markdown | — | Agent-readable, version-controllable, no runtime needed |
| Installer | Bash | POSIX | Symlink fan-out across agent dirs; no Node/Python dependency |
| Optional cron | macOS launchd | — | `scripts/com.obsidian-wiki.daily-update.plist` for `daily-update` skill |
| Optional semantic search | QMD (MCP server) | external | Speeds up `wiki-ingest` and `wiki-query`; framework falls back to Grep/Glob if absent |
| Storage | Plain markdown in user-supplied directory | — | Read by Obsidian directly; nothing proprietary |
| Distribution | `npx skills add Ar9av/obsidian-wiki` (preferred) or `git clone` + `bash setup.sh` | — | Two install paths; both end up symlinking the same `.skills/` |

There is no `package.json`, `go.mod`, `requirements.txt`, `Cargo.toml`, or equivalent. The repo declares no runtime dependencies because it has no runtime.

## Architecture Pattern: Skill-Based Agent Framework

### Skill anatomy

Every skill lives at `.skills/<name>/SKILL.md`. A skill is a self-contained markdown document that the agent reads and follows step-by-step. Some skills include subdirectories:

- `references/` — supporting context the skill loads on demand
- `scripts/` — optional helper scripts (Bash/JS) the agent may invoke
- `assets/` — templates, examples, or fixtures
- `agents/` — sub-skill definitions
- `eval-viewer/` — evaluation artifacts (only `skill-creator`)

Skills declare their trigger phrases in the routing table inside `AGENTS.md`. The host agent matches a user utterance against that table and reads the matched `SKILL.md`.

### Routing layer (always-on bootstrap)

Each agent has a different mechanism for "always-on" project context. obsidian-wiki ships them all, all pointing at the same content:

| Agent | Always-on file | Skill discovery |
|---|---|---|
| Claude Code | `CLAUDE.md` (symlink → `AGENTS.md`) | `.claude/skills/` + `~/.claude/skills/` |
| Cursor | `.cursor/rules/obsidian-wiki.mdc` | `.cursor/skills/` |
| Windsurf | `.windsurf/rules/obsidian-wiki.md` | `.windsurf/skills/` |
| Codex | `AGENTS.md` | `~/.codex/skills/` |
| Gemini CLI | `GEMINI.md` (symlink → `AGENTS.md`) | `~/.gemini/skills/` |
| Antigravity | `.agent/rules/` + `.agent/workflows/` | `.agents/skills/` |
| Kiro | `.kiro/steering/obsidian-wiki.md` | `.kiro/skills/` + `~/.kiro/skills/` |
| Hermes | `.hermes.md` (symlink → `AGENTS.md`) | `~/.hermes/skills/` |
| OpenClaw | `AGENTS.md` | `~/.openclaw/skills/` + `~/.agents/skills/` |
| OpenCode / Aider / Factory Droid | `AGENTS.md` | `~/.agents/skills/` |
| Trae / Trae CN | `AGENTS.md` | `~/.trae/skills/` / `~/.trae-cn/skills/` |
| GitHub Copilot (VS Code) | `.github/copilot-instructions.md` | — (describe intent in chat) |
| GitHub Copilot (CLI) | — | `~/.copilot/skills/` |

The canonical content is `AGENTS.md`. `CLAUDE.md`, `GEMINI.md`, and `.hermes.md` are symlinks to it — single source of truth for context, multiple discovery paths for breadth.

### Multi-agent fan-out

`setup.sh` is the deployment mechanism:

1. Creates `.env` from `.env.example` if missing; prompts for `OBSIDIAN_VAULT_PATH`.
2. Writes `~/.obsidian-wiki/config` so global skills (`wiki-update`, `wiki-query`) work from any directory.
3. Replaces `.hermes.md` with a fresh symlink to `AGENTS.md` (Windows git workaround included — replaces regular files written by git without `core.symlinks=true`).
4. Symlinks every directory in `.skills/*` into each agent's project-local discovery dir.
5. Symlinks the same skills (or, for `~/.claude/skills/`, just the two portable skills `wiki-update` + `wiki-query`) into each agent's global skill directory.

The function `install_skills()` defends against accidental data loss: it removes stale symlinks, replaces git-on-Windows symlink-as-regular-file artifacts, and skips real directories (so user customizations are not clobbered).

## Configuration Resolution

Skills resolve their config via the **Config Resolution Protocol** documented in `.skills/llm-wiki/SKILL.md`:

1. **Walk up from CWD** — look for `.env` in the current directory, then each parent up to `$HOME`. Stop at the first `.env` containing `OBSIDIAN_VAULT_PATH`.
2. **Global config** — if no local `.env` is found, read `~/.obsidian-wiki/config`.
3. **Prompt setup** — if neither exists, instruct the user to run `wiki-setup`.

The resolved config sets `OBSIDIAN_VAULT_PATH` (required) and may include `OBSIDIAN_WIKI_REPO`, `OBSIDIAN_SOURCES_DIR`, `OBSIDIAN_CATEGORIES`, `OBSIDIAN_MAX_PAGES_PER_INGEST`, `CLAUDE_HISTORY_PATH`, `CODEX_HISTORY_PATH`, `LINT_SCHEDULE`, `OBSIDIAN_LINK_FORMAT` (`wikilink` | `markdown`), `OBSIDIAN_RAW_DIR`, and optional QMD collection names (`QMD_WIKI_COLLECTION`, `QMD_PAPERS_COLLECTION`).

After config, the skill **always** reads `$OBSIDIAN_VAULT_PATH/AGENTS.md` if present — this is the user's owner-specific overrides (domain vocabulary, ingest preferences, writing style). Vault-side `AGENTS.md` overrides framework defaults for the duration of the session.

## Vault Data Model (compiled output)

The vault is the system's single source of truth for compiled knowledge:

```
$OBSIDIAN_VAULT_PATH/
├── index.md              # master catalog — every page, always current
├── log.md                # chronological op log (ingests, updates, lints)
├── hot.md                # ~500-word semantic snapshot of recent activity
├── .manifest.json        # ingest ledger (paths, timestamps, pages produced, last_commit_synced)
├── _meta/
│   ├── taxonomy.md       # controlled tag vocabulary
│   └── *.base            # Obsidian Bases dashboard definitions
├── _insights.md          # graph analysis (hubs, bridges, dead ends)
├── _raw/                 # staging area for unprocessed drafts
├── _archives/            # full snapshots from wiki-rebuild
├── concepts/             # abstract ideas, patterns, mental models
├── entities/             # people, tools, libraries, companies
├── skills/               # how-to knowledge, techniques, procedures
├── references/           # factual lookups — specs, APIs, configs
├── synthesis/            # cross-cutting analysis
├── journal/              # time-bound entries
└── projects/<name>/      # per-project knowledge (optional sub-tree)
```

Every page has required frontmatter: `title`, `category`, `tags`, `sources`, `created`, `updated`. Pages connect via internal links — `[[wikilinks]]` by default, or standard Markdown links when `OBSIDIAN_LINK_FORMAT=markdown`.

## Visibility Model

Pages may carry a `visibility/` tag (`public`, `internal`, `pii`). Untagged pages behave as public. Filtered mode is opt-in (triggered by phrases like "public only", "user-facing answer", "no internal content"). The vault stays single-source-of-truth — visibility shapes surfacing, not storage. Visibility tags don't count toward the 5-tag limit and are listed separately in the taxonomy.

## Cross-Project Skills (work from any directory)

Two skills are designed to be invoked from outside the obsidian-wiki repo:

- **`wiki-update`** — scans the *current* project (README, source structure, git log, package metadata), distills architecture decisions / patterns / trade-offs, writes to `$VAULT/projects/<project-name>.md`, and updates `.manifest.json`, `index.md`, `log.md`. On repeat runs uses `last_commit_synced` from the manifest to process only the delta.
- **`wiki-query`** — answers questions against the compiled wiki using a cheap pass over titles, tags, and `summary:` frontmatter first; opens page bodies only when needed; returns a synthesized answer with `[[wikilink]]` citations.

These two are the only skills installed into `~/.claude/skills/` so they're available globally without polluting the Claude Code skill list with the full 31-skill set.

## Core Operating Principles

- **Compile, don't retrieve.** Update existing pages — don't append or duplicate.
- **Track everything.** Update `.manifest.json` after ingesting, and `index.md` / `log.md` / `hot.md` after any write.
- **Connect with `[[wikilinks]]`.** Every page links to related pages — this is what makes it a graph, not a folder.
- **Frontmatter is required.** Every page: `title`, `category`, `tags`, `sources`, `created`, `updated`.
- **Single source of truth.** Visibility shapes surfacing, never duplicates content.
- **Keep context warm.** `hot.md` is updated on every write so the next session can pick up without re-crawling the vault.

## Testing Strategy

There is no test suite. Validation is operational:

- `wiki-lint` — finds orphans, broken links, stale content, contradictions.
- `wiki-status` — audits delta between sources and compiled pages.
- `impl-validator` — meta-skill that lets the user ask the agent to validate its own recent work.
- `wiki-rebuild` — archives current state to `_archives/` and rebuilds; full restore is supported.

## Deployment Architecture

There is no deployment. Distribution is install-time:

1. `npx skills add Ar9av/obsidian-wiki` (recommended) — the `skills` CLI handles per-agent install.
2. `git clone … && bash setup.sh` — manual symlink fan-out described above.

The optional `scripts/com.obsidian-wiki.daily-update.plist` is a macOS launchd plist that schedules the `daily-update` skill on a recurring basis.
