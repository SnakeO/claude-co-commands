# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin (`snakeo-co-commands`) providing three collaboration skills that use OpenAI's Codex as a second opinion, through the Codex CLI. No compiled code and no build steps: Markdown skill definitions, JSON config, and one small shell helper.

## Architecture

```
.claude-plugin/marketplace.json   ← marketplace registration (name, version, plugin refs)
plugins/co-commands/
  plugin.json                     ← plugin definition (auto-discovers skills via glob)
  scripts/codex-session           ← drives `codex exec` (new / run / read / reply)
  skills/
    co-brainstorm/SKILL.md        ← interactive brainstorming
    co-plan/SKILL.md              ← parallel plan generation
    co-validate/SKILL.md          ← staff engineer plan review
```

**Skill discovery:** `plugin.json` uses `"skills": ["./skills/*"]` — any subdirectory with a `SKILL.md` is auto-registered.

**SKILL.md format:** YAML frontmatter (`name`, `description`) followed by Markdown instructions that Claude Code follows when the skill is invoked.

## Core Design Pattern: Prevent Bias

All three skills follow the same anti-bias pattern:

1. The agent writes Codex's first message into a session folder (`codex-session new`), then spawns a
   **background subagent** that runs it (`codex-session run`). The prompt tells Codex to ask any
   clarifying questions first, and otherwise reply only with a skill-specific ready phrase:
   - co-brainstorm: "My brainstorming is complete and I'm ready to present"
   - co-plan: "My plan is ready to present"
   - co-validate: "My review is complete and I'm ready to present"
2. The subagent answers Codex's clarifying questions (`codex-session reply`) until Codex says it is
   ready, then reports back without requesting Codex's work.
3. Meanwhile the agent does its own independent work (brainstorming/planning/reviewing).
4. Only after finishing, the agent asks Codex to present (`codex-session reply`) and compares.

This prevents the agent from being influenced by Codex's response before forming its own perspective.

## Codex Integration (CLI, since 2.0.0)

The skills used to call the Codex MCP server (`codex mcp-server`, wrapped as the
`validate-plans-and-brainstorm-ideas` server). OpenAI removed that command in Codex 0.154.0
(openai/codex#42993), so 2.0.0 drives the CLI instead through `scripts/codex-session`:

- `new <label>` makes a temp session folder; `run <dir>` sends `<dir>/prompt.md` with
  `codex exec --json --sandbox read-only` and saves the reply and the thread id
- `reply <dir>` continues the thread with `codex exec resume <thread id>`, kept read-only with
  `-c 'sandbox_mode="read-only"'` (`exec resume` has no `--sandbox` flag)
- `read <dir>` prints the latest reply

Prompts always go on stdin: nothing needs escaping, and `codex exec` otherwise waits on stdin when
it is not a terminal. Skills reference the helper as `<base directory>/../../scripts/codex-session`.

Test with a real session (skills load from a folder with `claude --plugin-dir plugins/co-commands`).
`claude -p` kills background tasks when it exits, so a one-shot headless run never sees the
subagent finish; test the full flow in an interactive session.

## Versioning

**Always bump the version when making changes.** Version must be updated in three places and kept in sync:

1. `.claude-plugin/marketplace.json` → `metadata.version`
2. `.claude-plugin/marketplace.json` → `plugins[0].version`
3. `plugins/co-commands/plugin.json` → `version`
