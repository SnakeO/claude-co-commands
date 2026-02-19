# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin (`snakeo-co-commands`) providing three collaboration skills that use OpenAI's Codex MCP server as a second opinion. No compiled code, no build steps, no tests — purely Markdown skill definitions and JSON config.

## Architecture

```
.claude-plugin/marketplace.json   ← marketplace registration (name, version, plugin refs)
plugins/co-commands/
  plugin.json                     ← plugin definition (auto-discovers skills via glob)
  skills/
    co-brainstorm/SKILL.md        ← interactive brainstorming
    co-plan/SKILL.md              ← parallel plan generation
    co-validate/SKILL.md          ← staff engineer plan review
```

**Skill discovery:** `plugin.json` uses `"skills": ["./skills/*"]` — any subdirectory with a `SKILL.md` is auto-registered.

**SKILL.md format:** YAML frontmatter (`name`, `description`) followed by Markdown instructions that Claude Code follows when the skill is invoked.

## Core Design Pattern: Prevent Bias

All three skills follow the same anti-bias pattern:

1. Launch Codex in background via `mcp__validate-plans-and-brainstorm-ideas__codex` — prompt tells Codex to work but only reply `"I'm ready"`
2. Agent does its own independent work (brainstorming/planning/reviewing)
3. Only after finishing its own work, agent calls `codex-reply` to retrieve Codex's output
4. Compare both independent results

This prevents the agent from being influenced by Codex's response before forming its own perspective.

## MCP Integration

All skills use two MCP tools from the `validate-plans-and-brainstorm-ideas` server (wraps `@openai/codex mcp-server`):

- `mcp__validate-plans-and-brainstorm-ideas__codex` — start a new session (`sandbox: read-only`, `approval-policy: never`)
- `mcp__validate-plans-and-brainstorm-ideas__codex-reply` — continue via `threadId`

## Versioning

**Always bump the version when making changes.** Version must be updated in three places and kept in sync:

1. `.claude-plugin/marketplace.json` → `metadata.version`
2. `.claude-plugin/marketplace.json` → `plugins[0].version`
3. `plugins/co-commands/plugin.json` → `version`
