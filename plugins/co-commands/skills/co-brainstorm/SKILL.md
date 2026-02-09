---
name: co-brainstorm
description: Bounce ideas off Codex. Use when you want fast alternative ideas, critiques, and perspectives on any topic. Triggers an interactive conversation with the Codex MCP server for brainstorming and exploration.
---

# Co-Brainstorm: Bounce Ideas Off Codex

You MUST immediately call the `mcp__validate-plans-and-brainstorm-ideas__codex` tool with these parameters:

- `prompt`: `$ARGUMENTS`
- `sandbox`: `read-only`
- `approval-policy`: `never`
- `cwd`: (use the current working directory)

Wait for the result. This is an interactive conversation.

## After The Result Returns

1. Consider the ideas and perspectives offered.
2. Build on what is useful and discard what is not.
3. Continue the conversation to dig deeper.

## Continuing The Conversation

Use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the previous response
- `prompt`: your follow-up to challenge assumptions, explore alternatives, and test edge cases

Continue until you are comfortable with your understanding.

## How To Treat Responses

Treat Codex responses as coming from a junior developer:

- Never assume suggestions are correct; validate each one yourself.
- You are the lead engineer and have final say.
- Use responses as a starting point, not authoritative answers.
