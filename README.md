# Claude Co-Commands Plugin

3 collaboration commands for Claude Code that use the [Codex CLI](https://github.com/openai/codex) to generate parallel plans, validate plans, and brainstorm ideas.

## Commands

| Command | Description | When to Use |
|---------|-------------|-------------|
| `/co-brainstorm` | Bounce ideas off Codex | Want fast alternative ideas, critiques, and perspectives |
| `/co-plan` | Generate a parallel plan via Codex | Want a second opinion on your planning approach |
| `/co-validate` | Get a staff engineer review of your plan | Want critical feedback before finalizing a plan |

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- The [Codex CLI](https://github.com/openai/codex), signed in:

  ```bash
  npm install -g @openai/codex    # or: brew install codex
  codex login
  ```

No MCP server is needed. The skills run Codex through a small helper, `scripts/codex-session`,
which drives `codex exec` in Codex's read-only sandbox and keeps each conversation in a temporary
folder so follow-ups continue the same Codex session. A background subagent starts Codex and
answers any clarifying questions it asks, while Claude does its own independent work.

## Installation

### Option 1: Plugin Marketplace (Recommended)

```bash
# Add the marketplace
/plugin marketplace add SnakeO/claude-co-commands

# Install the plugin
/plugin install co-commands@snakeo-co-commands
```

### Option 2: Git Clone

```bash
git clone https://github.com/SnakeO/claude-co-commands.git
# Copy skill folders to ~/.claude/skills/
cp -r claude-co-commands/plugins/co-commands/skills/* ~/.claude/skills/
```

### Option 3: Manual Copy

Copy `plugins/co-commands/skills/` contents to `~/.claude/skills/`.

## Upgrading from 1.x

Versions 1.x used the Codex MCP server (`codex mcp-server`). OpenAI removed that command in
Codex 0.154.0, so the server now exits at startup and Claude Code reports
`validate-plans-and-brainstorm-ideas ... Connection closed`. Version 2.0 does not use it.
Remove the old server entry from wherever you added it:

```bash
claude mcp remove validate-plans-and-brainstorm-ideas -s user      # ~/.claude.json
claude mcp remove validate-plans-and-brainstorm-ideas -s project   # a project's .mcp.json
```

### Verify

1. Run `codex login status`; it should say you are logged in.
2. Restart Claude Code after updating the plugin.
3. Test with `/co-brainstorm test idea`. It should spawn a background subagent that runs `codex-session run`.

## Command Details

### `/co-brainstorm`

Starts an interactive brainstorming session with Codex. Pass your topic or question as the argument.

```
/co-brainstorm how should we structure the authentication system
```

Supports follow-up conversation to dig deeper into ideas.

### `/co-plan`

Generates an alternative plan in the background while you continue your own planning. Pass your task description as the argument.

```
/co-plan add user authentication with OAuth2 support
```

Compare the Codex plan against yours to catch missed approaches, simpler alternatives, or overlooked edge cases.

### `/co-validate`

Sends your plan to Codex for a staff-engineer-style review. Pass the path to your plan file.

```
/co-validate .claude/plans/my-plan.md
```

Returns critical issues, simplification opportunities, and alternative approaches. Supports back-and-forth discussion.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `the Codex CLI is not installed` | `npm install -g @openai/codex` (or `brew install codex`), then `codex login` |
| `Codex did not reply` | Read the lines it prints from `stderr.log`. For auth errors run `codex login` in a terminal |
| `validate-plans-and-brainstorm-ideas ... Connection closed` at startup | A leftover 1.x MCP server entry. Remove it (see Upgrading from 1.x) |
| Commands not appearing after install | Restart Claude Code and verify the skill folders exist |

## License

MIT
