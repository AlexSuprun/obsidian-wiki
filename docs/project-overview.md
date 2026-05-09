# Project Overview — obsidian-wiki

## Purpose

A skill-based framework for building and maintaining an Obsidian knowledge base with any AI coding agent. Implements Andrej Karpathy's "LLM Wiki" pattern — knowledge is **compiled** into interconnected markdown pages once, then maintained over time, instead of being retrieved fresh from an LLM every query.

**The wiki is the artifact. The agent is the maintainer. Obsidian is the viewer.**

## What it is (and isn't)

- **Is**: a set of markdown instruction files (skills) plus a `setup.sh` that wires those skills into every supported AI agent's discovery path.
- **Isn't**: an application. There is no runtime, no API server, no compiled binary, no package manifest. Everything executes inside the host AI agent (Claude Code, Cursor, Codex, etc.).

## Tech Stack Summary

| Category | Technology | Notes |
|---|---|---|
| Language | Markdown | All skill bodies — `SKILL.md` per skill |
| Language | Bash | `setup.sh`, `scripts/daily-update.sh`, `scripts/wiki-notify.sh` |
| Runtime | None | Skills are read and executed by the host AI agent |
| Build | None | No compile/bundle/package step |
| Distribution | Symlinks | `setup.sh` symlinks `.skills/*` into per-agent skill directories |
| Distribution | `npx skills add Ar9av/obsidian-wiki` | Recommended path via the `skills` CLI |
| Scheduling | macOS launchd | `scripts/com.obsidian-wiki.daily-update.plist` (optional, for `daily-update`) |
| Config | `.env` walk-up + `~/.obsidian-wiki/config` | Resolves `OBSIDIAN_VAULT_PATH` |

## Architecture Type

- **Repository type**: monolith, single part.
- **Architecture pattern**: skill-based / instruction-driven agent framework. Each skill is a self-contained markdown directory under `.skills/<name>/SKILL.md` (some include `references/`, `scripts/`, `assets/`).
- **Multi-agent fan-out**: a single canonical `.skills/` source is symlinked into 10+ agent-specific discovery paths so the same skill set works across Claude Code, Cursor, Windsurf, Codex, Gemini CLI, Antigravity, Hermes, OpenClaw, OpenCode, Aider, Factory Droid, Trae, Kiro, and GitHub Copilot (CLI + VS Code Chat).

## Skill Inventory (canonical source: `.skills/`)

31 skills as of this scan, grouped by intent:

- **Setup / lifecycle**: `wiki-setup`, `wiki-rebuild`, `wiki-switch`, `daily-update`
- **Ingest**: `wiki-ingest`, `wiki-history-ingest`, `claude-history-ingest`, `codex-history-ingest`, `hermes-history-ingest`, `openclaw-history-ingest`, `copilot-history-ingest`, `data-ingest`, `ingest-url`, `obsidian-wiki-ingest`
- **Read / query**: `wiki-query`, `wiki-status`, `wiki-research`
- **Maintain**: `wiki-lint`, `cross-linker`, `tag-taxonomy`, `wiki-update`, `wiki-synthesize`
- **Capture**: `wiki-capture`, `wiki-dashboard`
- **Export / visualize**: `wiki-export`, `graph-colorize`
- **Memory**: `memory-bridge`
- **Validation**: `impl-validator`
- **Meta**: `llm-wiki` (core pattern reference), `skill-creator`, `wiki-agent` (per-tool research router)

## Repository Structure (high-level)

```
obsidian-wiki/
├── .skills/                  # canonical source — 31 skills
├── .claude/skills/           # symlinks → .skills/* (Claude Code)
├── .cursor/skills/           # symlinks → .skills/* (Cursor)
├── .windsurf/skills/         # symlinks → .skills/* (Windsurf)
├── .agents/skills/           # symlinks → .skills/* (AGENTS.md-aware agents)
├── .kiro/skills/             # symlinks → .skills/* (Kiro)
├── .agent/{rules,workflows}/ # Google Antigravity bootstrap
├── .cursor/rules/            # Cursor always-on rules
├── .windsurf/rules/          # Windsurf always-on rules
├── .kiro/steering/           # Kiro always-on steering
├── .github/copilot-instructions.md  # GitHub Copilot context
├── scripts/                  # daily-update.sh, wiki-notify.sh, launchd plist
├── AGENTS.md                 # canonical agent context
├── CLAUDE.md → AGENTS.md     # alias for Claude Code
├── GEMINI.md → AGENTS.md     # alias for Gemini / Antigravity
├── .hermes.md → AGENTS.md    # alias for Hermes
├── README.md
├── SETUP.md
├── setup.sh
├── .env.example
└── LICENSE
```

See `source-tree-analysis.md` for annotated tree.

## Links

- [Architecture](./architecture.md) — three-layer pattern, skill anatomy, config resolution, multi-agent fan-out
- [Source Tree Analysis](./source-tree-analysis.md) — annotated directories
- [Development Guide](./development-guide.md) — setup, workflow, adding a skill
- Project-internal: [`AGENTS.md`](../AGENTS.md), [`README.md`](../README.md), [`SETUP.md`](../SETUP.md)
- Upstream pattern: [Karpathy LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
