---
name: co-plan
description: Generate a parallel plan via Codex. Use when you want an additional planning perspective to compare against your own plan. Runs in the background so you can continue working in parallel.
---

# Co-Plan: Generate a Parallel Plan via Codex

You MUST immediately call the `mcp__validate-plans-and-brainstorm-ideas__codex` tool with these parameters:

- `prompt`: (construct as shown below)
- `sandbox`: `workspace-write`
- `approval-policy`: `never`
- `cwd`: (use the current working directory)

### Prompt Format

```text
$ARGUMENTS

Write your final plan to a file at /tmp/<descriptive-slug>.md where <descriptive-slug> is a short kebab-case name derived from the task (e.g. /tmp/add-oauth2-auth.md). Respond with ONLY the filepath you wrote to. Do not include the plan text in your response. If you cannot write the file for any reason, fall back to responding with the full plan text instead.
```

Run this call in the background so you can continue your own planning work in parallel.

## After The Result Returns

1. The response should be a filepath (e.g. `/tmp/add-oauth2-auth.md`). Read that file to get the Codex plan.
2. If the response is the plan text itself (fallback), use it directly.
3. Compare the Codex plan against your own plan and look for:
   - Approaches you missed
   - Simpler alternatives
   - Risks or edge cases you overlooked
4. Integrate useful ideas into your plan and discard the rest.

## Continuing The Conversation

If you want to discuss the plan further, use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the previous response
- `prompt`: your follow-up question or counterpoint

## How To Treat Responses

Treat Codex responses as coming from a junior developer:

- Never assume suggestions are correct; validate each one yourself.
- You are the lead engineer and have final say.
- Use responses as a starting point, not authoritative answers.
