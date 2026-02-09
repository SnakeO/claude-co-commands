---
name: co-validate
description: Get a staff engineer review of your plan via Codex. Use when you want critical review feedback on a plan before finalizing it. Pass the path to the plan file as the argument.
---

# Co-Validate: Get a Staff Engineer Review of Your Plan

## Arguments

`$ARGUMENTS` should be the path to the plan file. If not provided, check if there is a plan file from the current session (for example in `.claude/projects/` or the working directory).

## Steps

1. Read the plan file at the path provided in `$ARGUMENTS`.
2. Find the original user prompt that triggered the plan by checking the conversation history.
3. Call the review tool with both the original prompt and the full plan content.

You MUST call the `mcp__validate-plans-and-brainstorm-ideas__codex` tool with these parameters:

- `prompt`: construct exactly as shown below
- `sandbox`: `read-only`
- `approval-policy`: `never`
- `cwd`: (use the current working directory)

## Prompt Format

```text
You are a staff engineer reviewing this plan. Respond with only critical issues, big simplifications, or a completely different better approach. Do not repeat the plan back. Be direct and concise.

Original request from the user:
<original_request>
{paste the user's original prompt/request that triggered the plan}
</original_request>

Plan:
{paste the full contents of the plan file}
```

Wait for the result before continuing. You need this feedback to finalize your plan.

## After The Result Returns

1. Review each point of feedback critically.
2. For each issue raised, either:
   - Accept it and update your plan accordingly
   - Override it with an explanation of why your approach is better
3. If points need discussion, continue the conversation.

## Continuing The Conversation

Use `mcp__validate-plans-and-brainstorm-ideas__codex-reply` with:

- `threadId`: the thread ID from the previous response
- `prompt`: your response addressing points, explaining overrides, and asking for clarification when needed

If you override points, explain why so they can push back if needed.

## How To Treat Responses

Treat Codex responses as coming from a junior developer:

- Never assume suggestions are correct; validate each one yourself.
- You are the lead engineer and have final say.
- Use responses as a starting point, not authoritative answers.
