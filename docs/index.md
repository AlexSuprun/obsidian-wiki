---
project_name: 'obsidian-wiki'
generated_by: 'bmad-document-project (quick scan)'
generated_at: '2026-05-09'
repository_type: 'monolith'
parts: 1
optimized_for_llm: true
---

# Project Documentation Index — obsidian-wiki

This is the AI agent's primary entry point for understanding this repo. Read this file first; follow links into the per-area docs for detail.

## Project Overview

- **Type:** monolith — markdown skill framework (no runtime, no build)
- **Primary Language:** Markdown + Bash
- **Architecture:** instruction-based agent framework (Karpathy LLM Wiki pattern)
- **Repository:** [github.com/Ar9av/obsidian-wiki](https://github.com/Ar9av/obsidian-wiki)

## Quick Reference

- **Tech Stack:** Markdown skills, Bash installer, optional macOS launchd cron, optional QMD MCP for semantic search.
- **Entry Point:** `setup.sh` for install; `AGENTS.md` for agent routing.
- **Architecture Pattern:** canonical `.skills/` directory symlinked into per-agent discovery paths; canonical `AGENTS.md` aliased via symlinks to `CLAUDE.md`, `GEMINI.md`, `.hermes.md`.
- **Skill count:** 31 skills (snapshot at scan time).
- **Storage model:** plain markdown in user-supplied Obsidian vault directory; required frontmatter; `[[wikilinks]]` (or markdown links if `OBSIDIAN_LINK_FORMAT=markdown`).

## Generated Documentation

- [Project Overview](./project-overview.md) — purpose, scope, what it is and isn't, skill inventory
- [Architecture](./architecture.md) — three-layer pattern, skill anatomy, multi-agent fan-out, config resolution, vault data model, visibility, cross-project skills, principles
- [Source Tree Analysis](./source-tree-analysis.md) — annotated directory tree, critical folders, entry points, symlink map
- [Development Guide](./development-guide.md) — prerequisites, setup, env config, daily workflow, adding a skill, troubleshooting

## Existing Documentation (in repo)

- [`README.md`](../README.md) — public-facing overview, quick start, full agent compatibility matrix (Claude Code, Cursor, Windsurf, Codex, Gemini CLI, Antigravity, Hermes, OpenClaw, OpenCode, Aider, Factory Droid, Trae / Trae CN, Kiro, GitHub Copilot)
- [`SETUP.md`](../SETUP.md) — install + minimum config walk-through
- [`AGENTS.md`](../AGENTS.md) — **canonical** agent context: skill routing table, vault structure, core principles, cross-project usage. `CLAUDE.md`, `GEMINI.md`, `.hermes.md` are symlinks to this file.
- [`.github/copilot-instructions.md`](../.github/copilot-instructions.md) — GitHub Copilot (VS Code Chat) project rules
- Per-skill docs: `.skills/<name>/SKILL.md` — the instruction body each agent reads when invoking the skill. Start with `.skills/llm-wiki/SKILL.md` for the core pattern.

## Per-Agent Always-On Files (not symlinks — keep in sync manually)

- `.cursor/rules/obsidian-wiki.mdc`
- `.windsurf/rules/obsidian-wiki.md`
- `.kiro/steering/obsidian-wiki.md`
- `.agent/rules/obsidian-wiki.md`, `.agent/workflows/obsidian-wiki.md` (Google Antigravity)
- `.github/copilot-instructions.md`

## Getting Started

1. Run `bash setup.sh` from the repo root. The installer creates `.env`, prompts for `OBSIDIAN_VAULT_PATH`, writes `~/.obsidian-wiki/config`, and symlinks every skill into every supported agent's discovery directory.
2. Open the project in your AI agent and say **"Set up my wiki"** — this triggers the `wiki-setup` skill.
3. From then on, talk to your agent in natural language. Routing lives in [`AGENTS.md`](../AGENTS.md) — match your phrase against the table to see which skill will fire.

## When Editing This Repo

- **Skills**: edit `.skills/<name>/SKILL.md` only. Other `*/skills/` paths are symlinks. Re-run `setup.sh` if you add a new skill so symlinks are created in every agent dir.
- **Agent context**: edit `AGENTS.md` only. `CLAUDE.md` / `GEMINI.md` / `.hermes.md` are symlinks. Keep the per-agent rule files (Cursor, Windsurf, Kiro, Antigravity, Copilot) in sync manually.
- **Adding a skill**: scaffold with the `skill-creator` skill, then update `AGENTS.md` routing table, then re-run `setup.sh`.

## Brownfield PRD Hint

When planning new features or refactors with the BMad PRD workflow, point it at this `index.md` as the project knowledge base.
