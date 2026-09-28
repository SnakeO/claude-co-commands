---
name: co-plan
description: Generate a parallel plan via Codex. Use when you want an additional planning perspective to compare against your own plan. Runs in the background so you can continue working in parallel.
---

# Co-Plan: Generate a Parallel Plan via Codex

## How Codex Is Reached

Codex runs through its CLI, driven by the helper script `scripts/codex-session`, which lives two
levels above this skill's base directory: call it as `<base directory>/../../scripts/codex-session`.
It keeps one Codex conversation per session folder and always runs Codex in its read-only sandbox:

- `codex-session new <label>` makes a session folder and prints its path
- `codex-session run <DIR>` sends `<DIR>/prompt.md` as the first message and waits for the reply
- `codex-session read <DIR>` prints Codex's latest reply
- `codex-session reply <DIR>` sends a follow-up read from stdin and prints the reply

Shell variables do not survive between Bash calls, so always use the literal folder path.

## Step 1: Start Codex and Hand It to a Background Subagent

1. Create the session and write Codex's first message, in one Bash call:

   ```bash
   DIR="$(<base directory>/../../scripts/codex-session new co-plan)" && echo "$DIR"
   cat > "$DIR/prompt.md" <<'CODEX_PROMPT'
   Create a detailed implementation plan for the following task. Think deeply about architecture, steps, edge cases, and trade-offs, reading the code in the working directory as needed — but do NOT share the plan yet. If you need to ask clarifying questions about the task before planning, ask them now. Otherwise, when your plan is fully formed and ready, respond with exactly: "My plan is ready to present" and nothing else. Wait for my next message before sharing the plan.

   Task: $ARGUMENTS
   CODEX_PROMPT
   ```

2. Spawn a **background subagent** (Agent tool with `run_in_background: true`) to handle all
   communication with Codex. Give it the helper path and the folder path, and tell it to:

   1. Run `<helper> run <DIR>` in the foreground, with the Bash tool's longest timeout
      (600000 ms). It can take several minutes.
   2. Run `<helper> read <DIR>`. If Codex replied "My plan is ready to present", go to step 4.
   3. If Codex asked clarifying questions instead, answer them using its own judgment and the
      code in the working directory, by sending a follow-up (quote the heredoc delimiter):
      `<helper> reply <DIR> <<'CODEX_PROMPT'` … `CODEX_PROMPT`. Then check the reply again as in
      step 2. If Codex sent its full plan instead of the ready message, treat it as ready.
   4. Report back only that Codex is ready, the session folder, and how many questions it
      answered. Do NOT request or repeat Codex's plan; that happens in Step 3.

   The subagent handles the back-and-forth so you are free to do your own work. In a
   non-interactive session (`claude -p`), do these steps yourself in the foreground instead:
   background work is killed when that kind of session ends.

## Step 2: Create Your Own Plan

While the subagent communicates with Codex in the background, create your own independent plan. **Do NOT check the Codex result until you have finished your own plan.** The entire point is to produce two independent plans and then compare them — reading Codex's plan early defeats this purpose and introduces bias.

## Step 3: Retrieve and Compare

Only after your plan is complete, confirm the background subagent has reported that Codex is
ready. Then ask Codex for its plan in the same conversation; the command prints it:

```bash
<base directory>/../../scripts/codex-session reply <DIR> <<'CODEX_PROMPT'
Go ahead, send the plan.
CODEX_PROMPT
```

If the subagent reported that Codex did not reply, show the user the error it relayed (it names
the fix, such as `codex login`) and continue with your own plan alone.

Once the plan arrives:

1. Read the Codex plan.
2. Compare it against your own plan and look for:
   - Approaches you missed
   - Simpler alternatives
   - Risks or edge cases you overlooked
3. Integrate useful ideas into your plan and discard the rest.

## Continuing the Conversation

To discuss the plan further, send a follow-up in the same Codex conversation. It prints Codex's answer:

```bash
<base directory>/../../scripts/codex-session reply <DIR> <<'CODEX_PROMPT'
your follow-up question or counterpoint
CODEX_PROMPT
```

## How to Treat Responses

Treat Codex responses as coming from a junior developer:

- Never assume suggestions are correct; validate each one yourself.
- You are the lead engineer and have final say.
- Use responses as a starting point, not authoritative answers.
