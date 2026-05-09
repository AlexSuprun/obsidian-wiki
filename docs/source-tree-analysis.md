# Source Tree Analysis — obsidian-wiki

Annotated layout. The repo's only canonical source is `.skills/` — every other `*/skills/` directory is a symlink fan-out for a specific agent's discovery path. The only canonical context file is `AGENTS.md`; `CLAUDE.md`, `GEMINI.md`, and `.hermes.md` are symlinks to it.

```
obsidian-wiki/
│
├── .skills/                          # ★ CANONICAL SOURCE — 31 skills
│   │
│   │  ── Setup / lifecycle ────────────────────────────
│   ├── wiki-setup/                   # initialize vault structure, create index/log, configure Obsidian
│   ├── wiki-rebuild/                 # archive → clear → rebuild, or restore from _archives/
│   ├── wiki-switch/                  # switch between multiple vaults
│   ├── daily-update/                 # morning sync; refresh index; install macOS cron + notification
│   │
│   │  ── Ingest ──────────────────────────────────────
│   ├── wiki-ingest/                  # distill source documents into wiki pages (append or full mode)
│   ├── wiki-history-ingest/          # router: dispatches to claude/codex/hermes/openclaw/copilot ingest
│   ├── claude-history-ingest/        # mine ~/.claude conversations + memories
│   ├── codex-history-ingest/         # mine ~/.codex sessions + rollouts
│   ├── hermes-history-ingest/        # mine ~/.hermes memories
│   ├── openclaw-history-ingest/      # mine ~/.openclaw sessions
│   ├── copilot-history-ingest/       # mine ~/.copilot sessions
│   ├── data-ingest/                  # ingest any raw text — chat exports, logs, transcripts
│   ├── ingest-url/                   # fetch + ingest a URL
│   ├── obsidian-wiki-ingest/         # special-case: ingest the obsidian-wiki repo itself
│   │
│   │  ── Read / query ────────────────────────────────
│   ├── wiki-query/                   # ★ portable — answer questions against compiled wiki with citations
│   ├── wiki-status/                  # delta audit; what's new / pending / stale; insights mode
│   ├── wiki-research/                # deep "find everything about X" sweep
│   │
│   │  ── Maintain ────────────────────────────────────
│   ├── wiki-lint/                    # orphans, broken links, stale content, contradictions
│   ├── cross-linker/                 # add [[wikilinks]] across pages that should connect
│   ├── tag-taxonomy/                 # normalize tags against _meta/taxonomy.md
│   ├── wiki-update/                  # ★ portable — sync current project's knowledge into the vault
│   ├── wiki-synthesize/              # find cross-concept patterns; produce synthesis/ pages
│   │
│   │  ── Capture ─────────────────────────────────────
│   ├── wiki-capture/                 # /wiki-capture — file the current conversation into the vault
│   ├── wiki-dashboard/               # create Obsidian Bases dashboards (_meta/*.base)
│   │
│   │  ── Export / visualize ──────────────────────────
│   ├── wiki-export/                  # export to graphml / neo4j; respects visibility filters
│   ├── graph-colorize/               # color Obsidian graph by tag/category/visibility
│   │
│   │  ── Memory ──────────────────────────────────────
│   ├── memory-bridge/                # cross-tool memory inspection (browse codex memory, compare tools)
│   │
│   │  ── Validation ──────────────────────────────────
│   ├── impl-validator/               # /impl-validator — agent self-review of recent implementation
│   │
│   │  ── Meta ────────────────────────────────────────
│   ├── llm-wiki/                     # ★ CORE — three-layer pattern, page templates, project org, Config Resolution Protocol
│   ├── skill-creator/                # scaffold a new skill (drafting / testing / refining)
│   └── wiki-agent/                   # /wiki-claude /wiki-codex /wiki-hermes /wiki-openclaw /wiki-copilot — per-tool research router
│
│  ── Per-agent symlink fan-out (every */skills/ entry is a symlink → ../.skills/<name>/) ──
├── .claude/skills/                   # Claude Code project-local skills
├── .cursor/skills/                   # Cursor project-local skills
├── .windsurf/skills/                 # Windsurf project-local skills
├── .agents/skills/                   # AGENTS.md-aware agents (Kilocode, generic)
├── .kiro/skills/                     # Kiro IDE/CLI project-local skills
│
│  ── Per-agent always-on context ───────────────────────
├── .agent/                           # Google Antigravity
│   ├── rules/                        # alwaysApply rules
│   └── workflows/                    # slash-command registry
├── .cursor/rules/                    # Cursor always-on rule (.mdc)
├── .windsurf/rules/                  # Windsurf always-on rule
├── .kiro/steering/                   # Kiro inclusion=always steering doc
├── .github/
│   └── copilot-instructions.md       # GitHub Copilot (VS Code Chat) instructions
│
│  ── Canonical agent context (single source of truth) ──
├── AGENTS.md                         # ★ CANONICAL — full skill routing table + core principles
├── CLAUDE.md     → AGENTS.md         # symlink for Claude Code
├── GEMINI.md     → AGENTS.md         # symlink for Gemini CLI / Antigravity
├── .hermes.md    → AGENTS.md         # symlink for Hermes (created by setup.sh)
│
│  ── Operator-facing docs ──────────────────────────────
├── README.md                         # 429 lines — overview, quick start, agent compatibility matrix
├── SETUP.md                          # 159 lines — install + minimum config walk-through
├── LICENSE                           # MIT (per repo)
│
│  ── Installer + config ────────────────────────────────
├── setup.sh                          # ★ ENTRY POINT — symlink fan-out + ~/.obsidian-wiki/config writer
├── .env.example                      # documented config template
├── .env                              # local config (gitignored — created by setup.sh)
├── .gitignore
│
│  ── Optional automation ───────────────────────────────
└── scripts/
    ├── com.obsidian-wiki.daily-update.plist  # macOS launchd plist for `daily-update` skill
    ├── daily-update.sh                       # cron-driven daily-update wrapper
    └── wiki-notify.sh                        # terminal notification helper
```

## Critical Folders Summary

| Folder | Role | Notes |
|---|---|---|
| `.skills/` | **Canonical skill source.** Every agent's `*/skills/` symlinks back here. | Edit skills here, only here. |
| `.claude/skills/`, `.cursor/skills/`, `.windsurf/skills/`, `.agents/skills/`, `.kiro/skills/` | Per-agent project-local discovery dirs. | Generated by `setup.sh`. Treat as build output. |
| `.agent/`, `.cursor/rules/`, `.windsurf/rules/`, `.kiro/steering/`, `.github/copilot-instructions.md` | Agent-specific always-on context bootstraps. | Hand-edited; not symlinks. |
| `scripts/` | Optional shell glue for the `daily-update` skill (cron, notifications). | Only relevant if user enables daily updates. |
| `AGENTS.md` (+ symlinks) | The canonical agent context: routing table, vault layout, core principles, cross-project usage. | Single source of truth. |

## Entry Points

- **For users**: `setup.sh` → `.env` → tell agent "set up my wiki" → reads `AGENTS.md` → routes to `wiki-setup` skill.
- **For agents**: agent always-on file (e.g. `CLAUDE.md`) → routing table → matched skill's `SKILL.md` → execute.
- **For cross-project usage**: `~/.claude/skills/wiki-update` and `~/.claude/skills/wiki-query` (the only two portable skills installed globally for Claude Code) — work from any directory by reading `~/.obsidian-wiki/config`.

## Symlinks Created by `setup.sh` (verbatim)

Project-local skill dirs (each iterates over `.skills/*/`):
- `.claude/skills/<name>` → `.skills/<name>`
- `.cursor/skills/<name>` → `.skills/<name>`
- `.windsurf/skills/<name>` → `.skills/<name>`
- `.agents/skills/<name>` → `.skills/<name>`
- `.kiro/skills/<name>` → `.skills/<name>`

Globals:
- `~/.claude/skills/wiki-update` → `.skills/wiki-update`
- `~/.claude/skills/wiki-query` → `.skills/wiki-query`
- `~/.gemini/skills/<name>`, `~/.gemini/antigravity/skills/<name>`, `~/.codex/skills/<name>`, `~/.hermes/skills/<name>`, `~/.openclaw/skills/<name>`, `~/.copilot/skills/<name>`, `~/.trae/skills/<name>`, `~/.trae-cn/skills/<name>`, `~/.kiro/skills/<name>`, `~/.agents/skills/<name>` → `.skills/<name>` (all skills)

Bootstrap:
- `.hermes.md` → `AGENTS.md`
