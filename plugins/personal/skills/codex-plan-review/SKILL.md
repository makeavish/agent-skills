---
name: codex-plan-review
description: Send the current implementation plan to OpenAI Codex CLI for iterative review. Use when the user wants a Codex second opinion on a plan, architecture, or implementation approach. Do not use this for code diff review or file-change review. Claude and Codex go back-and-forth until Codex approves the plan.
user_invocable: true
---

# Codex Plan Review (Iterative)

Send the current implementation plan to OpenAI Codex for review. Claude revises the plan based on Codex's feedback and re-submits until Codex approves. Max 5 rounds.

---

## When to Invoke

- When the user runs `/codex-plan-review` during or after plan mode
- When the user wants a second opinion on a plan from a different model

## Agent Instructions

When invoked, perform the following iterative review loop:

### Step 1: Ask for Model & Reasoning Effort

Use `AskUserQuestion` to ask **all three questions in a single prompt**:

> Which Codex model should I use for this review (`gpt-5.4`, `gpt-5.3-codex-spark`, or `gpt-5.3-codex`)? What reasoning effort (`xhigh`, `high`, `medium`, or `low`) should I use? And should I enable Codex fast mode for this review?

- If the user already specified a model as an argument, keep it. If the user already explicitly asked for fast mode, set `CODEX_FAST_MODE=true` without asking that part again. If the user specified neither reasoning effort nor fast mode, default to `high` effort and `CODEX_FAST_MODE=false`.
- Store the answers as `CODEX_MODEL`, `CODEX_EFFORT`, and `CODEX_FAST_MODE` for use in all subsequent `codex exec` calls.
- Build `FAST_MODE_FLAG="--enable fast_mode"` when `CODEX_FAST_MODE=true`; otherwise leave it empty.

### Step 2: Generate Session ID

Generate a unique ID to avoid conflicts with other concurrent Claude Code sessions:

```bash
REVIEW_ID=$(uuidgen | tr '[:upper:]' '[:lower:]' | head -c 8)
```

Use this for all temp file paths: `/tmp/claude-plan-${REVIEW_ID}.md` and `/tmp/codex-plan-review-${REVIEW_ID}.md`.

### Step 3: Capture the Plan

Write the current plan to the session-scoped temporary file. The plan is whatever implementation plan exists in the current conversation context (from plan mode, or a plan discussed in chat).

1. Write the full plan content to `/tmp/claude-plan-${REVIEW_ID}.md`
2. If there is no plan in the current context, ask the user what they want reviewed

### Step 4: Initial Review (Round 1)

Run Codex CLI in non-interactive mode to review the plan:

```bash
codex exec \
  --skip-git-repo-check \
  ${FAST_MODE_FLAG} \
  -m ${CODEX_MODEL} \
  --config model_reasoning_effort="${CODEX_EFFORT}" \
  -s read-only \
  -o /tmp/codex-plan-review-${REVIEW_ID}.md \
  "Review the implementation plan in /tmp/claude-plan-${REVIEW_ID}.md. Focus on:
1. Correctness - Will this plan achieve the stated goals?
2. Risks - What could go wrong? Edge cases? Data loss?
3. Missing steps - Is anything forgotten?
4. Alternatives - Is there a simpler or better approach?
5. Security - Any security concerns?

Be specific and actionable. If the plan is solid and ready to implement, end your review with exactly: VERDICT: APPROVED

If changes are needed, end with exactly: VERDICT: REVISE" 2>/dev/null
```

**Capture the Codex session ID** from the output line that says `session id: <uuid>`. Store this as `CODEX_SESSION_ID`. You MUST use this exact ID to resume in subsequent rounds (do NOT use `--last`, which would grab the wrong session if multiple reviews are running concurrently).

**Notes:**
- Always use `--skip-git-repo-check`.
- Always append `2>/dev/null` to suppress thinking tokens from stderr.
- Use `-s read-only` so Codex can read the codebase for context but cannot modify anything.
- Use `-o` to capture the output to a file for reliable reading.

### Step 5: Read Review & Check Verdict

1. Read `/tmp/codex-plan-review-${REVIEW_ID}.md`
2. Present Codex's review to the user:

```
## Codex Review — Round N (model: ${CODEX_MODEL}, effort: ${CODEX_EFFORT})

[Codex's feedback here]
```

If fast mode is enabled, include it in the heading, for example:

```
## Codex Review — Round N (model: ${CODEX_MODEL}, effort: ${CODEX_EFFORT}, fast mode: on)
```

3. Check the verdict:
   - If **VERDICT: APPROVED** → go to Step 8 (Done)
   - If **VERDICT: REVISE** → go to Step 6 (Revise & Re-submit)
   - If no clear verdict but feedback is all positive / no actionable items → treat as approved
   - If max rounds (5) reached → go to Step 8 with a note that max rounds hit

### Step 6: Revise the Plan

Based on Codex's feedback:

1. **Revise the plan** — address each issue Codex raised. Update the plan content in the conversation context and rewrite `/tmp/claude-plan-${REVIEW_ID}.md` with the revised version.
2. **Briefly summarize** what you changed for the user:

```
### Revisions (Round N)
- [What was changed and why, one bullet per Codex issue addressed]
```

3. Inform the user what's happening: "Sending revised plan back to Codex for re-review..."

### Step 7: Re-submit to Codex (Rounds 2-5)

Resume the existing Codex session so it has full context of the prior review:

```bash
echo "I've revised the plan based on your feedback. The updated plan is in /tmp/claude-plan-${REVIEW_ID}.md.

Here's what I changed:
[List the specific changes made]

Please re-review. If the plan is now solid and ready to implement, end with: VERDICT: APPROVED
If more changes are needed, end with: VERDICT: REVISE" \
  | codex exec --skip-git-repo-check ${FAST_MODE_FLAG} resume -o /tmp/codex-plan-review-${REVIEW_ID}.md ${CODEX_SESSION_ID} 2>/dev/null
```

**Note:** `codex exec resume` supports `-o` in current Codex CLI builds. Reuse `/tmp/codex-plan-review-${REVIEW_ID}.md` so every round is captured consistently, then read that file in Step 5.

Then go back to **Step 5** (Read Review & Check Verdict).

**Important:** If `resume ${CODEX_SESSION_ID}` fails (e.g., session expired), fall back to a fresh `codex exec` call (with `--skip-git-repo-check ${FAST_MODE_FLAG} -m ${CODEX_MODEL} --config model_reasoning_effort="${CODEX_EFFORT}"`) including context about the prior rounds in the prompt.

### Step 8: Present Final Result

Once approved (or max rounds reached):

```
## Codex Review — Final (model: ${CODEX_MODEL}, effort: ${CODEX_EFFORT}, fast mode: ${CODEX_FAST_MODE})

**Status:** ✅ Approved after N round(s)

[Final Codex feedback / approval message]

---
**The plan has been reviewed and approved by Codex. Ready for your approval to implement.**
```

If max rounds were reached without approval:

```
## Codex Review — Final (model: ${CODEX_MODEL}, effort: ${CODEX_EFFORT}, fast mode: ${CODEX_FAST_MODE})

**Status:** ⚠️ Max rounds (5) reached — not fully approved

**Remaining concerns:**
[List unresolved issues from last review]

---
**Codex still has concerns. Review the remaining items and decide whether to proceed or continue refining.**
```

### Step 9: Cleanup

Remove the session-scoped temporary files:
```bash
rm -f /tmp/claude-plan-${REVIEW_ID}.md /tmp/codex-plan-review-${REVIEW_ID}.md
```

## Loop Summary

```
Round 1: Claude sends plan → Codex reviews → REVISE?
Round 2: Claude revises → Codex re-reviews (resume session) → REVISE?
Round 3: Claude revises → Codex re-reviews (resume session) → APPROVED ✅
```

Max 5 rounds. Each round preserves Codex's conversation context via session resume.

## Critical Evaluation of Codex Feedback

Codex is a peer, not an authority. Evaluate its review feedback critically before incorporating it into plan revisions.

### Guidelines

- **Trust your own knowledge** when confident. If Codex flags something you know to be correct, don't blindly revise the plan around it — state the disagreement to the user.
- **Watch for stale knowledge** — Codex may be wrong about recent model names, library versions, API changes, or best practices that evolved after its training cutoff. Verify before revising.
- **Don't treat every REVISE as correct** — if Codex's concern is based on a false premise, note it to the user and skip that revision rather than degrading the plan.

### When Codex Feedback Seems Wrong

1. State the disagreement clearly to the user before revising
2. If warranted, push back via session resume — **identify yourself as Claude**:
   ```bash
   echo "This is Claude (<your current model name>) following up. I disagree with [X] because [evidence]. What's your take?" \
     | codex exec --skip-git-repo-check ${FAST_MODE_FLAG} resume -o /tmp/codex-plan-review-${REVIEW_ID}.md ${CODEX_SESSION_ID} 2>/dev/null
   ```
3. Frame it as peer discussion, not a correction — either AI could be wrong
4. Let the user decide how to proceed if there's genuine ambiguity

## Rules

- Claude **actively revises the plan** based on Codex feedback between rounds — this is NOT just passing messages, Claude should make real improvements
- Always ask for model, reasoning effort, and fast mode via `AskUserQuestion` (single prompt, three questions) unless the user already provided some or all of them
- Always use `--skip-git-repo-check` and `2>/dev/null` on all `codex exec` calls
- When fast mode is selected, include `--enable fast_mode` on the initial review, every resume call, and any fallback fresh run
- Always use read-only sandbox mode — Codex should never write files
- Max 5 review rounds to prevent infinite loops
- Show the user each round's feedback and revisions so they can follow along
- If Codex CLI is not installed or fails, inform the user and suggest `npm install -g @openai/codex`
- If fast mode is requested but unsupported by the installed Codex CLI, tell the user and suggest upgrading `@openai/codex` or continuing without fast mode
- If a revision contradicts the user's explicit requirements, skip that revision and note it for the user
