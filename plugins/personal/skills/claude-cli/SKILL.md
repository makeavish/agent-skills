---
name: claude-cli
description: Use when the user asks to run Claude Code CLI (`claude`, `claude -p`, `claude --print`), wants a Claude peer agent/session, or asks Claude Code to perform code analysis, refactoring, reviews, automated editing, background-agent work, or structured CLI output from the terminal
---

# Claude CLI Skill Guide

Claude Code uses the `claude` binary. Running `claude` by itself starts an interactive session; use `-p` / `--print` for non-interactive terminal output.

This guide was written against `claude --help` on Claude Code `2.1.204`. CLI flags move quickly, so when something behaves unexpectedly, re-check `claude --help` and the relevant subcommand help before assuming the tool or the user's setup is broken.

## Preflight Checks

Before running any Claude command:

1. Run `claude --version` to confirm installation. If it fails, tell the user Claude Code is not installed or not on `PATH`.
2. Run `claude auth status` to confirm authentication. If it is not authenticated, ask the user whether to run `claude auth login`.
3. If the task depends on MCP servers, run `claude mcp list` and make sure the required servers are configured and approved.
4. If the task depends on plugins or project-local behavior, remember that Claude may load `CLAUDE.md`, skills, plugins, hooks, MCP servers, settings, and other local configuration by default.

## Running a Task

1. Ask the user (via `AskUserQuestion`) which model to run, which effort level to use, which permission mode to use, and whether to run in the current directory or an isolated worktree, in a single prompt with four questions.
   - **Model:** accept aliases like `sonnet`, `opus`, or `fable`, or full model names. Default to `opus` when the user does not choose a model. Fable is the most intelligent model, but it is also the most expensive. Before starting multiple Fable instances—whether parallel, background, or batched—tell the user how many instances you plan to run and get explicit confirmation. A request to use Fable for one instance does not authorize additional Fable instances.
   - **Effort:** choose `low`, `medium`, `high`, `xhigh`, or `max`. Default to `medium` for ordinary work and `high` for deep reviews or complex refactors.
   - **Permission mode:** use the least powerful mode that fits the task. Prefer `manual` or `plan` for read-only review/planning, `acceptEdits` for controlled file edits, `auto` only when the user wants more autonomous execution, and `bypassPermissions` only in a trusted sandbox after explicit permission.
   - **Workspace:** run from the target repository directory. Use `--worktree [name]` when the user wants isolation, or `--add-dir <DIR>` when Claude needs access to extra directories.
2. Assemble the command with the appropriate options:
   - `-p, --print "your prompt here"` for non-interactive output
   - `--model <model>` for model selection
   - `--effort <low|medium|high|xhigh|max>` for reasoning effort
   - `--permission-mode <manual|plan|acceptEdits|auto|dontAsk|bypassPermissions>`
   - `--add-dir <DIR>` to allow access to additional directories
   - `--allowedTools <tools...>` / `--disallowedTools <tools...>` to narrow tool access
   - `--tools <tools...>` to specify the built-in tool set, or `--tools ""` to disable all tools
   - `--mcp-config <file-or-json>` to load MCP servers for this session
   - `--strict-mcp-config` to ignore other MCP configuration
   - `--output-format <text|json|stream-json>` for non-interactive output
   - `--json-schema <schema>` when the user needs validated structured output
   - `--fallback-model <model>` for print-mode fallback when the primary model is unavailable
   - `--max-budget-usd <amount>` for print-mode cost caps
   - `--name <name>` to label the session
   - `--session-id <uuid>` when deterministic session IDs matter
   - `--worktree [name]` to create a new git worktree for the session
   - `--bg` / `--background` to start a background agent without `-p` / `--print`; see [Background Agents](#background-agents)
3. **Working directory:** Claude CLI has no `-C` flag. Run it from the target directory, for example:
   ```bash
   cd /path/to/repo && claude -p "Review this change" --model sonnet --effort high --permission-mode manual
   ```
4. Do not suppress stderr by default. Claude may emit authentication, permission, MCP, validation, and tool errors there.
5. Run the command, capture stdout and stderr, and summarize the result for the user.
6. After Claude completes, tell the user the session can be continued with `claude --continue -p "..."` or resumed with `claude --resume <session-id> -p "..."` if a specific session ID is available.

### Output Handling

- For short tasks, capture stdout directly with default text output.
- For summaries, reviews, or machine-readable output, prefer JSON:
  ```bash
  cd /path/to/repo && claude -p "Summarize the architecture as JSON" --output-format json > /tmp/claude-output-$(date +%s).json
  ```
- For realtime consumers, use `--output-format stream-json`. Add `--include-partial-messages` only when partial chunks are useful.
- Use `--json-schema` when the next step needs a strict shape. Keep the schema small enough to quote safely in the shell, or put it in a temp file and interpolate it carefully.
- For long output, redirect stdout to a temp file and read it back before summarizing.

### Quick Reference

| Use case | Key flags |
| --- | --- |
| Read-only review or analysis | `-p "..." --permission-mode manual` |
| Planning-only pass | `-p "..." --permission-mode plan` |
| Controlled local edits | `-p "..." --permission-mode acceptEdits` |
| More autonomous execution | `-p "..." --permission-mode auto` |
| Skip permissions in a trusted sandbox | `-p "..." --permission-mode bypassPermissions` |
| Structured single result | `-p "..." --output-format json` |
| Realtime structured stream | `-p "..." --output-format stream-json` |
| Continue recent session | `--continue -p "..."` |
| Resume specific session | `--resume <session-id> -p "..."` |
| Fork from resumed session | Add `--fork-session` with `--resume` or `--continue` |
| Add another directory | `--add-dir <DIR>` |
| Run in a new worktree | `--worktree [name]` |
| Start background work | `--bg "..."` |

## Session Handling

- Use `claude --continue -p "new prompt"` to continue the most recent conversation in the current directory when there is only one relevant active workflow.
- Use `claude --resume <session-id> -p "new prompt"` when deterministic resume behavior matters.
- Use `--fork-session` with `--resume` or `--continue` when the user wants to branch from earlier work without mutating the original conversation.
- Use `--no-session-persistence` for one-off print-mode tasks that should not be resumable.
- If stdout JSON includes a session ID, save it and prefer explicit resume over `--continue`.

## Background Agents

- Use `--bg` / `--background` only when the user wants Claude to keep working while the terminal returns immediately.
- Do not combine background mode with `-p` / `--print`; launch background agents as `claude --bg "prompt"`.
- Inspect background work with:
  ```bash
  claude agents --json
  ```
- Scope inspection with `claude agents --cwd <path> --json` when multiple repositories have active background sessions.
- Tell the user when you start background work, what directory it is scoped to, and how you will check on it.

## Workspace Security

- Non-interactive `-p` mode skips the workspace trust dialog. Only use it in directories the user trusts.
- Treat `--permission-mode bypassPermissions`, `--dangerously-skip-permissions`, and broad tool allowlists as high impact. Ask the user before using them unless permission was already given.
- Use `--allowedTools`, `--disallowedTools`, `--tools`, `--mcp-config`, and `--strict-mcp-config` to narrow access when the task is sensitive.
- Use `--safe-mode` to troubleshoot broken local customizations. It disables customizations like `CLAUDE.md`, skills, plugins, hooks, MCP servers, commands, agents, output styles, workflows, themes, and keybindings.
- Use `--bare` for minimal, reproducible runs. In bare mode, Claude skips most local customizations and requires explicit context through flags such as `--system-prompt`, `--add-dir`, `--mcp-config`, `--settings`, `--agents`, or `--plugin-dir`.
- Avoid attaching secrets or sensitive files unless they are required for the task.

## Following Up

- After every `claude` command, immediately use `AskUserQuestion` to confirm next steps, collect clarifications, or decide whether to continue the same session.
- Restate the chosen model, effort, permission mode, working directory, and whether background/worktree mode was used when proposing follow-up actions.
- If the user wants to keep context, resume the existing session instead of starting a fresh one.
- If the user wants a peer review of Claude's output, evaluate it yourself first; do not treat the nested Claude session as the final authority.

## Critical Evaluation of Claude Output

Claude Code is powered by Anthropic models with their own limits, and local configuration can strongly shape behavior. Treat CLI output as a colleague's answer, not an authority.

### Guidelines

- Trust your own knowledge when confident. If the CLI output is wrong, say so.
- Research disagreements with documentation or source code before accepting claims about current APIs, model behavior, or release details.
- Remember that project-local instructions, skills, plugins, MCP servers, and hooks may change the output.
- Validate suggested edits and commands before applying or recommending them.

### When Claude Is Wrong

1. State your disagreement clearly to the user.
2. Provide evidence from code, docs, terminal output, or current web research.
3. Optionally resume the Claude session with a follow-up that frames the disagreement as peer review:
   ```bash
   claude --resume <session-id> -p "This is a peer agent following up. I disagree with [X] because [evidence]. Re-check your reasoning and respond with any correction."
   ```
4. Let the user decide how to proceed if there is genuine ambiguity.

## Error Handling

- Stop and report failures whenever `claude --version`, `claude auth status`, or `claude -p` exits non-zero.
- If authentication fails, run `claude auth status` and suggest `claude auth login` if appropriate.
- If MCP-dependent work fails, run `claude mcp list` and `claude mcp get <name>` before changing prompts.
- If local customizations appear broken, retry with `--safe-mode` only after telling the user what will be disabled.
- If the CLI rejects a flag, re-check `claude --help` on the installed version and adjust. Do not retry by guessing hidden flags.
- When output includes warnings or partial results, summarize them and ask how to adjust using `AskUserQuestion`.
