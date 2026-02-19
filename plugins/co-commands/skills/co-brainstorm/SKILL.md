---
name: co-brainstorm
description: Bounce ideas off Codex. Use when you want fast alternative ideas, critiques, and perspectives on any topic. Triggers an interactive conversation with the Codex MCP server for brainstorming and exploration.
---

# Co-Brainstorm: Bounce Ideas Off Codex

## Step 1: Launch Codex in the Background

You MUST immediately call the `mcp__validate-plans-and-brainstorm-ideas__codex` tool **in the background** with these parameters:

- `prompt`: `Brainstorm on the following topic. Think deeply, explore multiple angles, and prepare your ideas — but do NOT share them yet. When you are done thinking, respond with exactly: "I'm ready" and nothing else. Wait for my next message before sharing your ideas.\n\nTopic: $ARGUMENTS`
- `sandbox`: `read-only`
- `approval-policy`: `never`
- `cwd`: (use the current working directory)

This MUST run in the background so you can work in parallel.

## Step 2: Do Your Own Brainstorming

While Codex works in the background, do your own independent brainstorming on the topic. Think through:

- Multiple approaches and alternatives
- Trade-offs and risks
- Edge cases and constraints
- Creative or unconventional angles

Write down your own ideas and perspectives. **Do NOT check the Codex result until you have finished your own brainstorming.** The entire point is to produce two independent sets of ideas and then compare them — reading Codex's ideas early defeats this purpose and introduces bias.

## Step 3: Retrieve and Compare

Only after your own brainstorming is complete, check that the background Codex call has returned "I'm ready". Then use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the previous response
- `prompt`: `Go ahead, share your ideas.`

Once the ideas arrive:

1. Read the Codex brainstorm output.
2. Compare it against your own ideas and look for:
   - Perspectives you missed
   - Simpler or more creative alternatives
   - Risks or edge cases you overlooked
3. Integrate useful ideas into your thinking and discard the rest.

## Continuing The Conversation

If you want to dig deeper, use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the previous response
- `prompt`: your follow-up to challenge assumptions, explore alternatives, and test edge cases

## How To Treat Responses

Treat Codex responses as coming from a junior developer:

- Never assume suggestions are correct; validate each one yourself.
- You are the lead engineer and have final say.
- Use responses as a starting point, not authoritative answers.
