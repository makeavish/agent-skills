---
name: gemini-cli
description: Use when the user asks to run Gemini CLI (gemini prompt, gemini exec) or references Google Gemini CLI for code analysis, refactoring, or automated editing
---

# Gemini CLI Skill Guide

## Preflight Checks
Before running any Gemini command, verify the CLI is installed and authenticated:
1. Run `gemini --version` to confirm installation. If it fails, inform the user and suggest: `npm install -g @google/gemini-cli`
2. Check authentication. Gemini CLI supports multiple auth methods:
   - **Cached Google login** — the default; run `gemini` interactively once to complete the OAuth flow
   - **Gemini API key** — set `GEMINI_API_KEY` or `GOOGLE_API_KEY` environment variable
   - **Vertex AI** — Application Default Credentials (ADC), service-account JSON, or Google Cloud API key
3. If auth fails in headless mode, suggest the user run `gemini` interactively first, set the appropriate env var, or configure Vertex AI credentials.

## Running a Task
1. Ask the user (via `AskUserQuestion`) which model to run (`gemini-3.1-pro-preview`, `gemini-3-flash`, or `gemini-3.1-flash-lite-preview`), which approval mode to use (`default`, `auto_edit`, or `yolo`), and whether to enable sandbox mode — in a **single prompt with three questions**. Default approval mode to `default` if the user is unsure.
2. Select the approval mode required for the task; default to `--approval-mode default` unless auto-approval of edits or full tool access is necessary.
3. Assemble the command with the appropriate options:
   - `-m, --model <MODEL>`
   - `--approval-mode <default|auto_edit|yolo>` (or `--yolo` shorthand for `--approval-mode yolo`; do not combine both)
   - `-s, --sandbox` (boolean — include flag to enable sandboxed execution)
   - `--checkpointing` (required when using `auto_edit` or `yolo` — enables later recovery via interactive `/restore` if needed)
   - `-p, --prompt "<your prompt here>"` (non-interactive/headless mode)
   - `--output-format <text|json>` (optional — use `json` for structured output that's easier to parse programmatically)
   - `--include-directories <DIR>` (optional — extend workspace beyond cwd)
4. **Working Directory**: Gemini CLI does not have a `-C` flag. Run Gemini from the target repository's working directory. Use `cd <target-dir> &&` before the gemini command when operating on a different directory.
5. **Stderr Handling**: Do NOT append `2>/dev/null` by default. Gemini CLI may output auth errors, sandbox failures, and tool execution errors to stderr in headless mode. Only suppress stderr if the user explicitly requests it or after confirming it contains only UI artifacts.
6. Run the command, capture stdout, and summarize the outcome for the user.
7. **After Gemini completes**, inform the user: "You can run another Gemini task at any time. This skill currently operates statelessly — each invocation is independent."

### Output Handling
- For short tasks, capture stdout directly.
- For large output, use shell redirection to a temp file:
  ```bash
  cd <target-dir> && gemini -m gemini-3.1-pro-preview --approval-mode default -s -p "Analyze this codebase" --output-format json > /tmp/gemini-output-$(date +%s).json
  ```
  Then read and summarize the file.
- Prefer `-p "prompt"` over piped input — it is more explicit and avoids shell quoting issues.

### Quick Reference
| Use case | Approval mode | Key flags |
| --- | --- | --- |
| Read-only review or analysis | `default` | `--approval-mode default -s` |
| Apply edits (confirm each) | `default` | `--approval-mode default --checkpointing` |
| Auto-approve file edits only | `auto_edit` | `--approval-mode auto_edit --checkpointing` |
| Full auto (all tools auto-approved) | `yolo` | `--yolo --checkpointing` |
| Sandboxed execution | Any | Add `-s` |
| Structured output for parsing | Any | Add `--output-format json` |
| Extend workspace to other dirs | Any | `--include-directories <DIR>` |

## Workspace Security
- Gemini has trusted-folder behavior and may auto-load local `.env` or `.gemini/settings.json` depending on trust state.
- In untrusted repositories, always enable sandbox mode (`-s`).
- Warn the user about potential auto-loading of local configuration files.
- Avoid enabling telemetry or prompt logging unless the user explicitly requests it.

## Following Up
- After every `gemini` command, immediately use `AskUserQuestion` to confirm next steps, collect clarifications, or decide whether to run another task.
- Restate the chosen model, approval mode, and sandbox setting when proposing follow-up actions.
- For follow-up tasks, run a new `gemini -p` command with additional context. V1 is stateless — each invocation is independent.

### Future: Session Resume (not in v1)
Gemini supports interactive session management via `/chat save <tag>`, `/chat resume <tag>`, and `/chat list`. These are interactive-only commands and don't work in headless `-p` mode. A future version could explore headless resume if it becomes available.

## Critical Evaluation of Gemini Output

Gemini is powered by Google models with their own knowledge cutoffs and limitations. Treat Gemini as a **colleague, not an authority**.

### Guidelines
- **Trust your own knowledge** when confident. If Gemini claims something you know is incorrect, push back directly.
- **Research disagreements** using WebSearch or documentation before accepting Gemini's claims.
- **Remember knowledge cutoffs** — Gemini may not know about recent releases, APIs, or changes that occurred after its training data.
- **Don't defer blindly** — Gemini can be wrong. Evaluate its suggestions critically, especially regarding:
  - Model names and capabilities
  - Recent library versions or API changes
  - Best practices that may have evolved

### When Gemini is Wrong
1. State your disagreement clearly to the user
2. Provide evidence (your own knowledge, web search, docs)
3. Optionally run a new Gemini task to discuss the disagreement. **Identify yourself as Claude** so Gemini knows it's a peer AI discussion:
   ```bash
   gemini -m gemini-3.1-pro-preview --approval-mode default -s -p "This is Claude (<your current model name>) following up on a prior analysis. I disagree with [X] because [evidence]. What's your take on this?"
   ```
4. Frame disagreements as discussions, not corrections — either AI could be wrong
5. Let the user decide how to proceed if there's genuine ambiguity

## Error Handling
- Stop and report failures whenever `gemini --version` or a `gemini -p` command exits non-zero; request direction before retrying.
- Before you use high-impact flags (`--yolo`, disabling sandbox) ask the user for permission using `AskUserQuestion` unless it was already given.
- When output includes warnings or partial results, summarize them and ask how to adjust using `AskUserQuestion`.
- If Gemini reports rate limiting, inform the user. Quotas vary by auth method (e.g., 60 requests/min, 1,000 requests/day on the free Google account tier). Suggest waiting or switching to a paid plan.
