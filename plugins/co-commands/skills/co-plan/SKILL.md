---
name: co-plan
description: Generate a parallel plan via Codex. Use when you want an additional planning perspective to compare against your own plan. Runs in the background so you can continue working in parallel.
---

# Co-Plan: Generate a Parallel Plan via Codex

## Step 1: Launch Codex in the Background

You MUST immediately call the `mcp__validate-plans-and-brainstorm-ideas__codex` tool **in the background** with these parameters:

- `prompt`: `Create a detailed implementation plan for the following task. Think deeply about architecture, steps, edge cases, and trade-offs — but do NOT share the plan yet. When you are done planning, respond with exactly: "I'm ready" and nothing else. Wait for my next message before sharing the plan.\n\nTask: $ARGUMENTS`
- `sandbox`: `read-only`
- `approval-policy`: `never`
- `cwd`: (use the current working directory)

This MUST run in the background so you can work in parallel.

## Step 2: Create Your Own Plan

While Codex works in the background, create your own independent plan. **Do NOT check the Codex result until you have finished your own plan.** The entire point is to produce two independent plans and then compare them — reading Codex's plan early defeats this purpose and introduces bias.

## Step 3: Retrieve and Compare

Only after your own plan is finalized, check that the background Codex call has returned "I'm ready". Then use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the previous response
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
