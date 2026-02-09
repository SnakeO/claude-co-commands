---
name: co-plan
description: Generate a parallel plan via Codex. Use when you want an additional planning perspective to compare against your own plan. Runs in the background so you can continue working in parallel.
---

# Co-Plan: Generate a Parallel Plan via Codex

You MUST immediately call the `mcp__validate-plans-and-brainstorm-ideas__codex` tool with these parameters:

- `prompt`: `/plan $ARGUMENTS`
- `sandbox`: `read-only`
- `approval-policy`: `never`
- `cwd`: (use the current working directory)

Run this call in the background so you can continue your own planning work in parallel.

## After The Result Returns

1. Read the Codex plan output.
2. Compare it against your own plan and look for:
   - Approaches you missed
   - Simpler alternatives
   - Risks or edge cases you overlooked
3. Integrate useful ideas into your plan and discard the rest.

## Continuing The Conversation

If you want to discuss the plan further, use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the previous response
- `prompt`: your follow-up question or counterpoint

## How To Treat Responses

Treat Codex responses as coming from a junior developer:

- Never assume suggestions are correct; validate each one yourself.
- You are the lead engineer and have final say.
- Use responses as a starting point, not authoritative answers.
