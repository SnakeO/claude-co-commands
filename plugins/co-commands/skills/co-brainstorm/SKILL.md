---
name: co-brainstorm
description: Bounce ideas off Codex. Use when you want fast alternative ideas, critiques, and perspectives on any topic. Triggers an interactive conversation with the Codex MCP server for brainstorming and exploration.
---

# Co-Brainstorm: Bounce Ideas Off Codex

## Step 1: Spawn a Background Subagent for Codex

You MUST immediately spawn a **background subagent** (using the Task tool with `run_in_background: true`) to handle all communication with Codex. The subagent should:

1. Call `mcp__validate-plans-and-brainstorm-ideas__codex` with:
   - `prompt`: `Brainstorm on the following topic. Think deeply, explore multiple angles, and prepare your ideas — but do NOT share them yet. If you need to ask clarifying questions about the topic before brainstorming, ask them now. Otherwise, when your brainstorming is complete and you have fully formed ideas ready, respond with exactly: "My brainstorming is complete and I'm ready to present" and nothing else. Wait for my next message before sharing your ideas.\n\nTopic: $ARGUMENTS`
   - `sandbox`: `read-only`
   - `approval-policy`: `never`
   - `cwd`: (use the current working directory)

2. If Codex asks clarifying questions instead of saying it's ready, the subagent should answer them using its own judgment and the codebase context, then wait for Codex to finish and respond with "My brainstorming is complete and I'm ready to present".

3. Once Codex says "My brainstorming is complete and I'm ready to present", the subagent should report back that Codex is ready (but NOT request the ideas yet — that happens in Step 3).

The subagent handles the back-and-forth so the main agent is free to do its own work.

## Step 2: Do Your Own Brainstorming

While the subagent communicates with Codex in the background, do your own independent brainstorming on the topic. Think through:

- Multiple approaches and alternatives
- Trade-offs and risks
- Edge cases and constraints
- Creative or unconventional angles

Write down your own ideas and perspectives. **Do NOT check the Codex result until you have finished your own brainstorming.** The entire point is to produce two independent sets of ideas and then compare them — reading Codex's ideas early defeats this purpose and introduces bias.

## Step 3: Retrieve and Compare

Only after your own brainstorming is complete, confirm the background subagent has reported that Codex is ready. Then use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the Codex session
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
