---
name: co-plan
description: Generate a parallel plan via Codex. Use when you want an additional planning perspective to compare against your own plan. Runs in the background so you can continue working in parallel.
---

# Co-Plan: Generate a Parallel Plan via Codex

## Step 1: Spawn a Background Subagent for Codex

You MUST immediately spawn a **background subagent** (using the Task tool with `run_in_background: true`) to handle all communication with Codex. The subagent should:

1. Call `mcp__validate-plans-and-brainstorm-ideas__codex` with:
   - `prompt`: `Create a detailed implementation plan for the following task. Think deeply about architecture, steps, edge cases, and trade-offs — but do NOT share the plan yet. If you need to ask clarifying questions about the task before planning, ask them now. Otherwise, when your plan is fully formed and ready, respond with exactly: "My plan is ready to present" and nothing else. Wait for my next message before sharing the plan.\n\nTask: $ARGUMENTS`
   - `sandbox`: `read-only`
   - `approval-policy`: `never`
   - `cwd`: (use the current working directory)

2. If Codex asks clarifying questions instead of saying it's ready, the subagent should answer them using its own judgment and the codebase context, then wait for Codex to finish and respond with "My plan is ready to present".

3. Once Codex says "My plan is ready to present", the subagent should report back that Codex is ready (but NOT request the plan yet — that happens in Step 3).

The subagent handles the back-and-forth so the main agent is free to do its own work.

## Step 2: Create Your Own Plan

While the subagent communicates with Codex in the background, create your own independent plan. **Do NOT check the Codex result until you have finished your own plan.** The entire point is to produce two independent plans and then compare them — reading Codex's plan early defeats this purpose and introduces bias.

## Step 3: Retrieve and Compare

Only after your own plan is finalized, confirm the background subagent has reported that Codex is ready. Then use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the Codex session
- `prompt`: `Go ahead, send the plan.`

Once the plan arrives:

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
