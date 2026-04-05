---
name: codex
description: Use when the user asks to run Codex CLI (codex exec, codex resume), asks to use OpenAI Codex for code analysis, refactoring, or automated editing, or explicitly asks for Codex fast mode
---

# Codex Skill Guide

## Running a Task
1. Ask the user (via `AskUserQuestion`) which model to run (`gpt-5.4`, `gpt-5.3-codex-spark`, or `gpt-5.3-codex`), which reasoning effort to use (`xhigh`, `high`, `medium`, or `low`), which sandbox mode (`read-only`, `workspace-write`, or `danger-full-access`) to use, and whether to enable fast mode in a **single prompt with four questions**. Default sandbox to `read-only` if the user is unsure. If the user already asked for fast mode, set `CODEX_FAST_MODE=true` without asking again.
2. Select the sandbox mode required for the task; default to `--sandbox read-only` unless edits or network access are necessary.
3. Store the user's fast-mode choice as `CODEX_FAST_MODE`. When it is enabled, add `--enable fast_mode` to every `codex exec` or `codex exec resume` command for this workflow.
4. Assemble the command with the appropriate options:
   - `-m, --model <MODEL>`
   - `--config model_reasoning_effort="<xhigh|high|medium|low>"`
   - `--sandbox <read-only|workspace-write|danger-full-access>`
   - `--enable fast_mode` (optional, when `CODEX_FAST_MODE=true`)
   - `--full-auto`
   - `-C, --cd <DIR>`
   - `--skip-git-repo-check`
   - `-o <file>` (optional — use for long-running tasks where stdout may be large; see note below)
   - `"your prompt here"` (as final positional argument)
5. Always use `--skip-git-repo-check`.
6. **IMPORTANT**: By default, append `2>/dev/null` to all `codex exec` commands to suppress thinking tokens (stderr). Only show stderr if the user explicitly requests to see thinking tokens or if debugging is needed.
7. Run the command, capture stdout/stderr (filtered as appropriate), and summarize the outcome for the user.
8. **After Codex completes**, capture the session ID from the output line `session id: <uuid>` and store it as `CODEX_SESSION_ID`. Inform the user: "You can resume this Codex session at any time by saying 'codex resume' or asking me to continue with additional analysis or changes."

### Output Capture Note
- Use `-o <file>` with `codex exec` when you expect long output (large analysis, multi-file refactors). Reading from a file is more reliable than stdout for large results.
- `-o <file>` is also supported by `codex exec resume` in current Codex CLI builds. Use it for follow-up turns when you want reliable capture of a long reply.

### Fast Mode
- Fast mode is a Codex feature flag. Enable it per command with `--enable fast_mode`.
- For users who want it on by default across sessions, use `codex features enable fast_mode`. To turn it back off, use `codex features disable fast_mode`.
- Fast mode is separate from model choice and reasoning effort. Treat it as an independent latency preference, not as a substitute for picking a model.

### Quick Reference
| Use case | Sandbox mode | Key flags |
| --- | --- | --- |
| Read-only review or analysis | `read-only` | `--sandbox read-only 2>/dev/null` |
| Apply local edits | `workspace-write` | `--sandbox workspace-write --full-auto 2>/dev/null` |
| Permit network or broad access | `danger-full-access` | `--sandbox danger-full-access --full-auto 2>/dev/null` |
| Enable fast mode for a one-off run | Match task needs | `--enable fast_mode` plus the usual model/sandbox flags |
| Resume recent session (single-session) | Inherited from original | `echo "prompt" \| codex exec --skip-git-repo-check resume --last 2>/dev/null` |
| Resume specific session (concurrent-safe) | Inherited from original | `echo "prompt" \| codex exec --skip-git-repo-check resume ${CODEX_SESSION_ID} 2>/dev/null` |
| Resume and explicitly re-enable fast mode | Inherited from original | `echo "prompt" \| codex exec --skip-git-repo-check --enable fast_mode resume ${CODEX_SESSION_ID} 2>/dev/null` |
| Resume and capture a long reply to a file | Inherited from original | `echo "prompt" \| codex exec --skip-git-repo-check resume -o /tmp/codex-resume.txt ${CODEX_SESSION_ID} 2>/dev/null` |
| Run from another directory | Match task needs | `-C <DIR>` plus other flags `2>/dev/null` |

## Following Up
- After every `codex` command, immediately use `AskUserQuestion` to confirm next steps, collect clarifications, or decide whether to resume.
- **Preferred resume** (concurrent-safe): use the captured `CODEX_SESSION_ID`:
  ```bash
  echo "new prompt" | codex exec --skip-git-repo-check resume ${CODEX_SESSION_ID} 2>/dev/null
  ```
- If the user wants fast mode on a resumed session and it is not already enabled, use:
  ```bash
  echo "new prompt" | codex exec --skip-git-repo-check --enable fast_mode resume ${CODEX_SESSION_ID} 2>/dev/null
  ```
- **Fallback resume** (single-session only): use `--last` if no session ID was captured:
  ```bash
  echo "new prompt" | codex exec --skip-git-repo-check resume --last 2>/dev/null
  ```
- If the resumed answer may be long, prefer capturing it to a file:
  ```bash
  echo "new prompt" | codex exec --skip-git-repo-check resume -o /tmp/codex-resume.txt ${CODEX_SESSION_ID} 2>/dev/null
  ```
- When resuming, don't use configuration flags (model, reasoning effort) unless the user explicitly requests them — the resumed session inherits them from the original.
- Restate the chosen model, reasoning effort, sandbox mode, and fast-mode setting when proposing follow-up actions.

### Concurrent Session Safety
`--last` grabs the most recently created Codex session globally. If multiple Claude Code sessions are running simultaneously, it may resume the wrong one. Always prefer the explicit `CODEX_SESSION_ID` captured from the initial run's `session id: <uuid>` output line. `--last` is fine for simple single-session workflows.

## Critical Evaluation of Codex Output

Codex is powered by OpenAI models with their own knowledge cutoffs and limitations. Treat Codex as a **colleague, not an authority**.

### Guidelines
- **Trust your own knowledge** when confident. If Codex claims something you know is incorrect, push back directly.
- **Research disagreements** using WebSearch or documentation before accepting Codex's claims. Share findings with Codex via resume if needed.
- **Remember knowledge cutoffs** - Codex may not know about recent releases, APIs, or changes that occurred after its training data.
- **Don't defer blindly** - Codex can be wrong. Evaluate its suggestions critically, especially regarding:
  - Model names and capabilities
  - Recent library versions or API changes
  - Best practices that may have evolved

### When Codex is Wrong
1. State your disagreement clearly to the user
2. Provide evidence (your own knowledge, web search, docs)
3. Optionally resume the Codex session to discuss the disagreement. **Identify yourself as Claude** so Codex knows it's a peer AI discussion. Use your actual model name (e.g., the model you are currently running as) instead of a hardcoded name:
   ```bash
   echo "This is Claude (<your current model name>) following up. I disagree with [X] because [evidence]. What's your take on this?" | codex exec --skip-git-repo-check resume ${CODEX_SESSION_ID} 2>/dev/null
   ```
4. Frame disagreements as discussions, not corrections - either AI could be wrong
5. Let the user decide how to proceed if there's genuine ambiguity

## Error Handling
- Stop and report failures whenever `codex --version` or a `codex exec` command exits non-zero; request direction before retrying.
- Before you use high-impact flags (`--full-auto`, `--sandbox danger-full-access`) ask the user for permission using `AskUserQuestion` unless it was already given.
- If `--enable fast_mode` fails because the installed CLI is too old or the feature is unavailable, tell the user and suggest upgrading `@openai/codex` before retrying without fast mode.
- When output includes warnings or partial results, summarize them and ask how to adjust using `AskUserQuestion`.
