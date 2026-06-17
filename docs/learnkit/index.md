# Understanding obsidian-wiki — Learning Documentation

> A progressive guide to the obsidian-wiki framework: how a runtime-free, instruction-based agent system compiles knowledge into an Obsidian vault. For developers who want to understand, use, and extend the project.

---

## Phase 1: Foundations & Mental Model
*Understand what the project is, the idea behind it, and the layers it operates on.*

| Module | Topic | Description |
|--------|-------|-------------|
| [1.1](1-foundations/1.1-what-is-obsidian-wiki.md) | What is obsidian-wiki | The one-sentence definition, what it is and isn't (no runtime, no API, no build), and the "wiki is the artifact, agent is the maintainer" framing. |
| [1.2](1-foundations/1.2-the-llm-wiki-pattern.md) | The LLM Wiki Pattern | Karpathy's "compile, don't retrieve" idea — why knowledge is compiled once into pages instead of regenerated per query, and what that buys you. |
| [1.3](1-foundations/1.3-three-layer-architecture.md) | Three-Layer Architecture | Source → Wiki → Maintenance loop. How raw inputs become compiled pages and how the loop keeps them current. |

---

## Phase 2: The Skill System
*Learn the core unit of the framework — the skill — and how agents find and run them.*

| Module | Topic | Description |
|--------|-------|-------------|
| [2.1](2-skill-system/2.1-skill-anatomy.md) | Skill Anatomy | The structure of a skill: `SKILL.md` plus optional `references/`, `scripts/`, `assets/`, `agents/`. What each part is for. |
| [2.2](2-skill-system/2.2-skill-routing.md) | Skill Routing & Discovery | How a user utterance maps to a skill via the `AGENTS.md` routing table, and how each agent discovers skills. |
| [2.3](2-skill-system/2.3-canonical-source-and-fanout.md) | Canonical Source & Multi-Agent Fan-Out | One canonical `.skills/` source symlinked into 10+ agent discovery paths (Claude Code, Cursor, Codex, Gemini, **Hermes**, OpenClaw, Kiro, Copilot…). The single-source-of-truth design, the `AGENTS.md` symlinks, and the `.hermes.md` symlink case. |

---

## Phase 3: Setup & Configuration
*Learn how the framework installs itself and resolves where the vault lives.*

| Module | Topic | Description |
|--------|-------|-------------|
| [3.1](3-setup-config/3.1-the-installer.md) | The Installer (setup.sh) | What `setup.sh` does step by step — symlink fan-out, global config, the `.hermes.md` workaround, and how it avoids clobbering data. |
| [3.2](3-setup-config/3.2-config-resolution-protocol.md) | Config Resolution Protocol | The `.env` walk-up from CWD, fallback to `~/.obsidian-wiki/config`, and the vault-side `AGENTS.md` override. |
| [3.3](3-setup-config/3.3-environment-variables.md) | Environment Variables | The full `.env` reference — required vs optional vars, defaults, and what each one controls. |

---

## Phase 4: The Knowledge Vault
*Learn the data model — the compiled output the whole system maintains.*

| Module | Topic | Description |
|--------|-------|-------------|
| [4.1](4-knowledge-vault/4.1-vault-structure.md) | Vault Structure & Categories | The folder layout (`concepts/`, `entities/`, `skills/`, etc.) and how pages are sorted into them. |
| [4.2](4-knowledge-vault/4.2-page-frontmatter.md) | Page Frontmatter & Templates | The required frontmatter (`title`, `category`, `tags`, `sources`, `created`, `updated`) and the page template shape. |
| [4.3](4-knowledge-vault/4.3-links-and-special-files.md) | Wikilinks & Special Files | `[[wikilinks]]` as the graph edges, plus the four state files: `index.md`, `log.md`, `hot.md`, `.manifest.json`. |
| [4.4](4-knowledge-vault/4.4-visibility-model.md) | Visibility Model | The optional `visibility/` tags (`public`/`internal`/`pii`), filtered mode, and single-source-of-truth design. |

---

## Phase 5: Core Workflows
*Learn the day-to-day operations: getting knowledge in, getting answers out, keeping it healthy.*

| Module | Topic | Description |
|--------|-------|-------------|
| [5.1](5-workflows/5.1-ingest-family.md) | The Ingest Family | `wiki-ingest`, per-agent history ingest (`claude-`, `codex-`, **`hermes-`**, `openclaw-`, `copilot-history-ingest`), `data-ingest`, `ingest-url` — what each handles and how delta tracking works. |
| [5.2](5-workflows/5.2-query-and-status.md) | Query & Status | `wiki-query` (cheap index pass → page bodies → cited answer) and `wiki-status` (delta audit + insights mode). |
| [5.3](5-workflows/5.3-maintenance-loop.md) | The Maintenance Loop | `wiki-lint`, `cross-linker`, `tag-taxonomy`, `wiki-synthesize` — how the vault is kept connected and drift-free. |
| [5.4](5-workflows/5.4-cross-project-skills.md) | Cross-Project Skills | `wiki-update` and `wiki-query` working from any directory — the two portable skills and how they read global config. |

---

## Phase 6: Extending & Operating
*Learn how to add to the system, automate it, and validate your changes.*

| Module | Topic | Description |
|--------|-------|-------------|
| [6.1](6-extending/6.1-adding-a-skill.md) | Adding or Modifying a Skill | The four-step loop: create under `.skills/`, add routing to `AGENTS.md`, sync agent rule files, re-run `setup.sh`. |
| [6.2](6-extending/6.2-automation.md) | Automation (daily-update) | The macOS launchd cron, `daily-update.sh`, and the terminal notification helper. |
| [6.3](6-extending/6.3-validation-and-troubleshooting.md) | Validation & Troubleshooting | No test suite — operational validation via `wiki-lint`, `wiki-status`, `impl-validator`, `wiki-rebuild`. Common symlink/config problems. |

---

## Reference Materials

| Resource | Description |
|----------|-------------|
| [Cheat Sheet](cheat-sheet.md) | Skill map, vault layout, config resolution flow, command quick reference |
| [Glossary](glossary.md) | Terms: skill, fan-out, Config Resolution Protocol, manifest, hot cache, visibility, and more |

---

## Quick Reference

### Essential Commands
```bash
bash setup.sh                  # install: symlink skills into every agent's discovery path
"set up my wiki"               # → wiki-setup (one-time vault init)
"what's the status?"           # → wiki-status (delta between sources and vault)
"ingest the new stuff"         # → wiki-ingest (compile sources into pages)
"what do I know about X?"      # → wiki-query (cited answer from compiled pages)
"audit my wiki"                # → wiki-lint (orphans, broken links, staleness)
/wiki-update                   # sync the current project into the vault (any directory)
```

### Key Files
| File | Purpose |
|------|---------|
| `.skills/<name>/SKILL.md` | Canonical source for each skill — the instructions the agent executes |
| `AGENTS.md` | Canonical agent context: routing table, vault layout, core principles |
| `setup.sh` | Entry point — symlink fan-out + writes `~/.obsidian-wiki/config` |
| `.env` / `~/.obsidian-wiki/config` | Resolves `OBSIDIAN_VAULT_PATH` and other settings |
| `$VAULT/index.md`, `log.md`, `hot.md`, `.manifest.json` | The four state files every write keeps current |

---

*20 modules + cheat sheet + glossary*
