---
name: kilocode-cli
description: Use when the user asks to run Kilo Code CLI (`kilo run`, `kilo session`, `kilo agent`) or references Kilo Code / Kilo CLI for code analysis, refactoring, reviews, or automated editing
---

# Kilo Code CLI Skill Guide

Kilo Code CLI uses the `kilo` binary, not `kilocode`.

## Preflight Checks
Before running any Kilo command, verify the CLI is installed and authenticated:
1. Run `kilo --version` to confirm installation. If it fails, inform the user that Kilo Code CLI is not installed or not on `PATH`.
2. Run `kilo auth list` to see which providers are configured.
3. If authentication is missing or the desired provider is unavailable, use `kilo auth login [url]` to log in to the right provider.
4. Discover available models and agents before proposing choices:
   - `kilo models`
   - `kilo agent list`

## Running a Task
1. Ask the user (via `AskUserQuestion`) which model to run, which agent to use, and whether to enable `--auto` for automatic permission approval, in a **single prompt with three questions**. Before asking, run `kilo models` and offer a short validated shortlist of real model IDs instead of a freeform string. Include examples in `provider/model` format such as `kilo/kilo-auto/balanced`, `kilo/google/gemini-3.1-pro-preview`, or `kilo/anthropic/claude-sonnet-4.5`. Default `--auto` to off if the user is unsure.
2. Confirm the working directory. Prefer `--dir <DIR>` over `cd` so the scope is explicit.
3. Assemble the command with the appropriate options:
   - `kilo run "your prompt here"`
   - `--dir <DIR>`
   - `-m, --model <provider/model>` (pick from validated `kilo models` output, not from guesswork)
   - `--agent <agent>`
   - `--variant <name>` (optional — only use when the user explicitly requests a provider-specific reasoning tier or you already know a valid variant from local/provider documentation)
   - `--format <default|json>` (use `json` for structured output that is easier to parse)
   - `-f, --file <PATH>` (optional — attach files to the prompt)
   - `--title <TITLE>` (optional)
   - `--thinking` (optional — only when the user wants to inspect reasoning blocks)
   - `--auto` (optional — high impact)
4. Do NOT suppress stderr by default. Kilo may emit authentication failures, permission prompts, and tool errors there.
5. Run the command, capture stdout, and summarize the result for the user.
6. After Kilo completes, inform the user that you can continue the same session later with `kilo run --continue "..."` or `kilo run --session <id> "..."`.

### Output Handling
- For short tasks, capture stdout directly.
- For large or structured output, prefer `--format json` and redirect to a temp file:
  ```bash
  kilo run "Analyze this repository" --dir /path/to/repo --format json > /tmp/kilo-output-$(date +%s).json
  ```
  Then read and summarize the file.
- Prefer a positional message (`kilo run "prompt"`) over shell piping unless the user explicitly needs piped input.

## Session Handling
- Use `kilo run --continue "new prompt"` to continue the most recent local session when there is only one active workflow.
- Use `kilo session list` to discover a specific session ID, then run `kilo run --session <id> "new prompt"` when deterministic resume behavior matters.
- Use `--fork` with `--continue` or `--session` when the user wants to branch from earlier work without mutating the original conversation.

### Quick Reference
| Use case | Permissions | Key flags |
| --- | --- | --- |
| Read-only review or analysis | Manual approvals | `kilo run "..." --dir <DIR>` |
| Structured output for parsing | Manual approvals | Add `--format json` |
| Attach local files | Manual approvals | Add `-f <PATH>` |
| Auto-approve tool permissions | High impact | Add `--auto` |
| Continue recent session | Inherited from prior run | `kilo run --continue "..."` |
| Continue a specific session | Inherited from prior run | `kilo run --session <id> "..."` |
| Branch from prior work | Inherited from prior run | Add `--fork` with `--continue` or `--session` |

## Workspace Security
- `--auto` automatically approves permissions. Treat it as high impact and ask the user before enabling it.
- Use `--dir` deliberately so Kilo works only in the intended repository or folder.
- Warn the user that Kilo may rely on project-local configuration or provider credentials already present in the environment.
- Avoid attaching sensitive files unless they are necessary for the task.

## Following Up
- After every `kilo` command, immediately use `AskUserQuestion` to confirm next steps, gather clarifications, or decide whether to continue the same session.
- Restate the chosen model, agent, working directory, and whether `--auto` was enabled when proposing follow-up actions.
- If the user wants to keep context, prefer continuing the existing session instead of starting a fresh one.

## Critical Evaluation of Kilo Output

Kilo Code is a CLI client over provider-hosted models. Treat its output as a **colleague, not an authority**.

### Guidelines
- **Trust your own knowledge** when confident. If Kilo claims something you know is wrong, push back directly.
- **Research disagreements** using documentation or web search before accepting Kilo's claims.
- **Remember model/provider limits** — the answer quality and recency depend on the configured provider and model.
- **Don't defer blindly** — validate suggestions critically, especially around:
  - Model capabilities
  - Recent library versions or API changes
  - Best practices that may have evolved

### When Kilo Is Wrong
1. State your disagreement clearly to the user.
2. Provide evidence from your own knowledge, documentation, or web research.
3. Optionally continue the Kilo session with a follow-up prompt that frames the disagreement as a discussion:
   ```bash
   kilo run --session <id> "This is Claude following up. I disagree with [X] because [evidence]. What's your take on this?"
   ```
4. Let the user decide how to proceed if there is genuine ambiguity.

## Error Handling
- Stop and report failures whenever `kilo --version` or `kilo run` exits non-zero; request direction before retrying.
- Before you use `--auto`, ask the user for permission using `AskUserQuestion` unless it was already given.
- When output includes warnings or partial results, summarize them and ask how to adjust using `AskUserQuestion`.
- If the requested model is unavailable, re-check `kilo models`, refresh the cache with `kilo models --refresh` if needed, and then choose a configured provider/model.
