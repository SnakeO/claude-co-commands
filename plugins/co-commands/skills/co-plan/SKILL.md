---
name: co-plan
description: Generate a parallel plan via Codex. Use when you want an additional planning perspective to compare against your own plan. Runs in the background so you can continue working in parallel.
---

# Co-Plan: Generate a Parallel Plan via Codex

You MUST immediately call the `mcp__validate-plans-and-brainstorm-ideas__codex` tool with these parameters:

- `prompt`: `/plan $ARGUMENTS — When you have finished planning, do NOT send the plan yet. Instead respond with exactly: "I'm ready to tell you the plan. Just let me know when." Then wait for my reply before sending the plan.`
- `sandbox`: `read-only`
- `approval-policy`: `never`
- `cwd`: (use the current working directory)

Run this call in the background so you can continue your own planning work in parallel.

## After The Result Returns

The first response should be Codex saying it is ready. **You MUST complete your own plan BEFORE requesting the Codex plan.** Do not read, request, or look at the Codex plan until your own plan is fully written. The entire point is to produce two independent plans and then compare them — reading Codex's plan early defeats this purpose.

Only after your own plan is finalized, use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the previous response
- `prompt`: `Go ahead, send the plan.`

Then once the plan arrives:

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
