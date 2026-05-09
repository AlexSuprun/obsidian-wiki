# Development Guide — obsidian-wiki

## Prerequisites

- Bash (POSIX). `setup.sh` uses Bash and standard Unix tools (`ln`, `sed`, `mkdir`).
- A writable directory for the Obsidian vault (`OBSIDIAN_VAULT_PATH`).
- At least one supported AI coding agent installed (Claude Code, Cursor, Codex, Gemini CLI, etc.). See `README.md` agent matrix.
- *(Optional)* macOS, if you want the `daily-update` launchd cron.
- *(Optional)* QMD MCP server, for semantic search acceleration.

There is **no** Node, Python, Go, or other language toolchain to install. The framework has no runtime dependencies.

## First-Time Setup

```bash
# 1. Clone (or use `npx skills add Ar9av/obsidian-wiki` instead — see README)
git clone https://github.com/Ar9av/obsidian-wiki.git
cd obsidian-wiki

# 2. Run the installer
bash setup.sh
```

`setup.sh` will:

1. Create `.env` from `.env.example` if missing.
2. Prompt for your vault path if not already set.
3. Write `~/.obsidian-wiki/config` so the portable skills work from any directory.
4. Replace `.hermes.md` with a fresh symlink to `AGENTS.md`.
5. Symlink `.skills/*` into every supported agent's project-local discovery directory.
6. Symlink the same skills into every supported agent's global directory (Claude Code only gets the two portable skills, `wiki-update` and `wiki-query`).
7. Print a summary.

## Environment Configuration

Edit `.env` after setup:

| Variable | Required | Default | Purpose |
|---|---|---|---|
| `OBSIDIAN_VAULT_PATH` | ✅ | — | Absolute path to the Obsidian vault directory |
| `OBSIDIAN_SOURCES_DIR` |  | *(empty)* | Comma-separated source directories for ingest |
| `OBSIDIAN_CATEGORIES` |  | `concepts,entities,skills,references,synthesis,journal` | Folder names created in vault |
| `OBSIDIAN_MAX_PAGES_PER_INGEST` |  | `15` | Cap per ingest run |
| `CLAUDE_HISTORY_PATH` |  | auto-discover from `~/.claude` | Override Claude history root |
| `CODEX_HISTORY_PATH` |  | `~/.codex` | Override Codex history root |
| `LINT_SCHEDULE` |  | `weekly` | `daily` / `weekly` / `manual` |
| `OBSIDIAN_LINK_FORMAT` |  | `wikilink` | `wikilink` or `markdown`. Affects future writes only. |
| `OBSIDIAN_RAW_DIR` |  | `_raw` | Staging directory inside the vault |
| `QMD_WIKI_COLLECTION` |  | — | QMD collection name for compiled wiki |
| `QMD_PAPERS_COLLECTION` |  | — | QMD collection name for raw sources |

## Daily Workflow

The intended user loop:

```
"Set up my wiki"             → wiki-setup (one-time)
"What's the status?"         → wiki-status reports the delta
"Ingest the new stuff"       → wiki-ingest processes the delta
"What do I know about X?"    → wiki-query answers from compiled pages
"Audit my wiki"              → wiki-lint flags orphans / broken links / staleness
```

When drift accumulates:

```
"Archive and rebuild"        → wiki-rebuild snapshots to _archives/, clears, fresh ingest
"Restore the old one"        → wiki-rebuild restores from a previous archive
```

From any other project directory:

```
/wiki-update                 → sync this project's knowledge into the vault
/wiki-query <question>       → ask against the compiled wiki
```

## Adding or Modifying a Skill

1. **Create the skill directory** under `.skills/` (the canonical source). Optionally use the `skill-creator` skill — it scaffolds a `SKILL.md` and walks you through testing.
2. **Add a routing entry** to `AGENTS.md` so the agent knows when to invoke the new skill. Because `CLAUDE.md`, `GEMINI.md`, and `.hermes.md` are symlinks to `AGENTS.md`, this updates context for all symlinked agents in one edit.
3. **Update agent-specific rule files** if the skill needs always-on context (`.cursor/rules/obsidian-wiki.mdc`, `.windsurf/rules/obsidian-wiki.md`, `.kiro/steering/obsidian-wiki.md`, `.agent/rules/obsidian-wiki.md`, `.agent/workflows/obsidian-wiki.md`, `.github/copilot-instructions.md`). These are not symlinked — keep them in sync.
4. **Re-run `bash setup.sh`** to fan out new symlinks. The installer handles stale-symlink replacement automatically.

### Skill anatomy

```
.skills/<skill-name>/
├── SKILL.md            # required — the instruction body the agent executes
├── references/         # optional — supporting context loaded on demand
├── scripts/            # optional — helper scripts the skill may invoke
├── assets/             # optional — templates, examples, fixtures
└── agents/             # optional — sub-skill definitions
```

Convention: name skills with a prefix that signals scope:
- `wiki-*` — core wiki operations
- `*-history-ingest` — per-tool conversation/session ingest
- `*-ingest` — generic ingest variants

## Modifying Agent Context

The canonical context lives in **`AGENTS.md` only**. `CLAUDE.md`, `GEMINI.md`, and `.hermes.md` are symlinks to it (created by `setup.sh`). Edit `AGENTS.md` directly — never the symlinks.

The agent-specific always-on rule files (`.cursor/rules/…`, `.windsurf/rules/…`, `.kiro/steering/…`, `.agent/rules/…`, `.agent/workflows/…`, `.github/copilot-instructions.md`) are independent files because each agent has its own front-matter / directive format. Keep them in sync manually when conventions change.

## Optional: Daily Update Cron (macOS)

```bash
# Tell your agent:
"/daily-update set up the daily cron"
```

Behind the scenes, the `daily-update` skill installs `scripts/com.obsidian-wiki.daily-update.plist` via `launchctl` and wires `scripts/daily-update.sh` to run on schedule. The optional `scripts/wiki-notify.sh` produces a Terminal notification when an update completes.

## Test / Validation Approach

There is no test runner. Validation is operational:

- `wiki-lint` — finds orphans, broken `[[wikilinks]]`, stale pages, contradictions.
- `wiki-status` — audits delta between sources and compiled pages; reports drift.
- `impl-validator` — meta-skill: ask the agent to validate its own recent implementation work.
- `wiki-rebuild` — full archive + restore safety net. Snapshots live in `$VAULT/_archives/`. Nothing is ever lost.

When iterating on a skill, the recommended manual loop is:

1. Edit `.skills/<name>/SKILL.md`.
2. Run the skill against a small test vault (or your real vault if you trust the change).
3. Inspect the resulting pages, `index.md`, `log.md`, and `.manifest.json` deltas.
4. Run `wiki-lint` to confirm no regressions in the broader vault.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Skill not discovered by agent | Symlink missing or stale | Re-run `bash setup.sh`. It detects and replaces stale symlinks. |
| `setup.sh` warns "is a real directory, skipping symlink" | A previous manual install left a real dir | Inspect the directory; remove or merge contents, then re-run `setup.sh`. |
| On Windows, skill files appear as text containing a path | Git wrote symlinks as regular files (no `core.symlinks=true`) | `setup.sh` detects this and replaces with real symlinks; re-run after enabling `git config --global core.symlinks true`. |
| Skills can't find vault | `OBSIDIAN_VAULT_PATH` missing | Check `.env`, then `~/.obsidian-wiki/config`. Re-run `setup.sh` to re-prompt. |
| `.hermes.md` content drifted from `AGENTS.md` | Edited the symlink as if a regular file | `setup.sh` rewrites it as a symlink on every run. Edit `AGENTS.md` instead. |
