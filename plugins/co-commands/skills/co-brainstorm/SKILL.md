---
name: co-brainstorm
description: Bounce ideas off Codex. Use when you want fast alternative ideas, critiques, and perspectives on any topic. Triggers an interactive conversation with Codex for brainstorming and exploration.
---

# Co-Brainstorm: Bounce Ideas Off Codex

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
   DIR="$(<base directory>/../../scripts/codex-session new co-brainstorm)" && echo "$DIR"
   cat > "$DIR/prompt.md" <<'CODEX_PROMPT'
   Brainstorm on the following topic. Think deeply, explore multiple angles, and prepare your ideas, reading the code in the working directory where it helps — but do NOT share them yet. If you need to ask clarifying questions about the topic before brainstorming, ask them now. Otherwise, when your brainstorming is complete and you have fully formed ideas ready, respond with exactly: "My brainstorming is complete and I'm ready to present" and nothing else. Wait for my next message before sharing your ideas.

   Topic: $ARGUMENTS
   CODEX_PROMPT
   ```

2. Spawn a **background subagent** (Agent tool with `run_in_background: true`) to handle all
   communication with Codex. Give it the helper path and the folder path, and tell it to:

   1. Run `<helper> run <DIR>` in the foreground, with the Bash tool's longest timeout
      (600000 ms). It can take several minutes.
   2. Run `<helper> read <DIR>`. If Codex replied "My brainstorming is complete and I'm ready to present", go to step 4.
   3. If Codex asked clarifying questions instead, answer them using its own judgment and the
      code in the working directory, by sending a follow-up (quote the heredoc delimiter):
      `<helper> reply <DIR> <<'CODEX_PROMPT'` … `CODEX_PROMPT`. Then check the reply again as in
      step 2. If Codex sent its full ideas instead of the ready message, treat it as ready.
   4. Report back only that Codex is ready, the session folder, and how many questions it
      answered. Do NOT request or repeat Codex's ideas; that happens in Step 3.

   The subagent handles the back-and-forth so you are free to do your own work. In a
   non-interactive session (`claude -p`), do these steps yourself in the foreground instead:
   background work is killed when that kind of session ends.

## Step 2: Do Your Own Brainstorming

While the subagent communicates with Codex in the background, do your own independent brainstorming on the topic. Think through:

- Multiple approaches and alternatives
- Trade-offs and risks
- Edge cases and constraints
- Creative or unconventional angles

Write down your own ideas and perspectives. **Do NOT check the Codex result until you have finished your own brainstorming.** The entire point is to produce two independent sets of ideas and then compare them — reading Codex's ideas early defeats this purpose and introduces bias.

## Step 3: Retrieve and Compare

Only after your brainstorming is complete, confirm the background subagent has reported that Codex is
ready. Then ask Codex for its ideas in the same conversation; the command prints it:

```bash
<base directory>/../../scripts/codex-session reply <DIR> <<'CODEX_PROMPT'
Go ahead, share your ideas.
CODEX_PROMPT
```

If the subagent reported that Codex did not reply, show the user the error it relayed (it names
the fix, such as `codex login`) and continue with your own brainstorming alone.

Once the ideas arrive:

1. Read the Codex brainstorm output.
2. Compare it against your own ideas and look for:
   - Perspectives you missed
   - Simpler or more creative alternatives
   - Risks or edge cases you overlooked
3. Integrate useful ideas into your thinking and discard the rest.

## Continuing the Conversation

To dig deeper, send a follow-up in the same Codex conversation. It prints Codex's answer:

```bash
<base directory>/../../scripts/codex-session reply <DIR> <<'CODEX_PROMPT'
your follow-up: challenge assumptions, explore alternatives, test edge cases
CODEX_PROMPT
```

## How to Treat Responses

Treat Codex responses as coming from a junior developer:

- Never assume suggestions are correct; validate each one yourself.
- You are the lead engineer and have final say.
- Use responses as a starting point, not authoritative answers.
