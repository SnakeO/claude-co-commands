---
name: co-validate
description: Get a staff engineer review of your plan via Codex. Use when you want critical review feedback on a plan before finalizing it. Pass the path to the plan file as the argument.
---

# Co-Validate: Get a Staff Engineer Review of Your Plan

## Arguments

`$ARGUMENTS` should be the path to the plan file. If not provided, check if there is a plan file from the current session (for example in `~/.claude/plans/` or the working directory).

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

Read the plan file, and find the original user prompt that led to the plan in the conversation
history. Paste the original request between the tags below; the plan file itself is appended
after the prompt with `cat`, not pasted.

1. Create the session and write Codex's first message, in one Bash call:

   ```bash
   DIR="$(<base directory>/../../scripts/codex-session new co-validate)" && echo "$DIR"
   cat > "$DIR/prompt.md" <<'CODEX_PROMPT'
   You are a staff engineer reviewing this plan. Analyze it for critical issues, big simplifications, or a completely different better approach, reading the code in the working directory where it helps — but do NOT share your review yet. If you need to ask clarifying questions about the plan or original request before reviewing, ask them now. Otherwise, when your review is complete and fully formed, respond with exactly: "My review is complete and I'm ready to present" and nothing else. Wait for my next message before sharing your review.

   Original request from the user:
   <original_request>
   {paste the user's original prompt/request that triggered the plan}
   </original_request>

   Plan:
   CODEX_PROMPT
   cat "<path to the plan file>" >> "$DIR/prompt.md"
   ```

2. Spawn a **background subagent** (Agent tool with `run_in_background: true`) to handle all
   communication with Codex. Give it the helper path and the folder path, and tell it to:

   1. Run `<helper> run <DIR>` in the foreground, with the Bash tool's longest timeout
      (600000 ms). It can take several minutes.
   2. Run `<helper> read <DIR>`. If Codex replied "My review is complete and I'm ready to present", go to step 4.
   3. If Codex asked clarifying questions instead, answer them using its own judgment and the
      code in the working directory, by sending a follow-up (quote the heredoc delimiter):
      `<helper> reply <DIR> <<'CODEX_PROMPT'` … `CODEX_PROMPT`. Then check the reply again as in
      step 2. If Codex sent its full review instead of the ready message, treat it as ready.
   4. Report back only that Codex is ready, the session folder, and how many questions it
      answered. Do NOT request or repeat Codex's review; that happens in Step 3.

   The subagent handles the back-and-forth so you are free to do your own work. In a
   non-interactive session (`claude -p`), do these steps yourself in the foreground instead:
   background work is killed when that kind of session ends.

## Step 2: Do Your Own Review

While the subagent communicates with Codex in the background, do your own independent review of the plan. Look for:

- Critical issues or flaws in the approach
- Opportunities for simplification
- Missing edge cases or risks
- Whether a completely different approach would be better

Write down your own assessment. **Do NOT check the Codex result until you have finished your own review.** The entire point is to produce two independent reviews and then compare them — reading Codex's review early defeats this purpose and introduces bias.

## Step 3: Retrieve and Compare

Only after your review is complete, confirm the background subagent has reported that Codex is
ready. Then ask Codex for its review in the same conversation; the command prints it:

```bash
<base directory>/../../scripts/codex-session reply <DIR> <<'CODEX_PROMPT'
Go ahead, share your review. Be direct and concise. Do not repeat the plan back. Focus only on critical issues, big simplifications, or a completely different better approach.
CODEX_PROMPT
```

If the subagent reported that Codex did not reply, show the user the error it relayed (it names
the fix, such as `codex login`) and continue with your own review alone.

Once the review arrives:

1. Read the Codex review output.
2. Compare it against your own review.
3. For each issue raised (by either review), either:
   - Accept it and update the plan accordingly
   - Override it with an explanation of why the current approach is better

## Continuing the Conversation

To address its points, explain overrides and ask for clarification, send a follow-up in the same Codex conversation. It prints Codex's answer:

```bash
<base directory>/../../scripts/codex-session reply <DIR> <<'CODEX_PROMPT'
your response: points addressed, overrides explained, questions
CODEX_PROMPT
```

If you override points, explain why so Codex can push back if needed.

## How to Treat Responses

Treat Codex responses as coming from a junior developer:

- Never assume suggestions are correct; validate each one yourself.
- You are the lead engineer and have final say.
- Use responses as a starting point, not authoritative answers.
