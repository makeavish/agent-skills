---
name: cursor-cli
description: Use when the user asks to run Cursor CLI (`agent`, `cursor-agent`), wants a Cursor peer session, or asks Cursor's terminal agent to analyze code, review changes, plan, refactor, or edit files. Does not apply to generic agents or opening files in the Cursor editor.
---

# Cursor CLI Skill Guide

Cursor's terminal agent uses `agent`; `cursor-agent` is a backward-compatible alias. The editor's `cursor` launcher is a different command. If `agent` names another program on PATH, use the verified `cursor-agent` binary instead.

Guidance checked on 2026-10-01 against the [command reference](https://cursor.com/docs/cli/reference/parameters), [headless guide](https://cursor.com/docs/cli/headless), [permissions](https://cursor.com/docs/cli/reference/permissions), and installed CLI `2026.09.18-9a7762b`. Verify the installed help when behavior differs.

## Preflight

1. Run `agent --version` and `agent --help` to confirm this is Cursor CLI. If missing, point to the [official installation guide](https://cursor.com/docs/cli/installation); installation is a separate action.
2. Run `agent status` to check authentication. Use existing login or `CURSOR_API_KEY`; if authentication is missing, have the user complete `agent login`. Never print or embed the API key in a command.
3. Run `agent models` (or `agent --list-models`) before offering model choices. Cursor's model IDs and account availability differ from direct provider APIs.
4. Set the target repository with `--workspace <DIR>`. If the task needs MCP tools, check `agent --workspace <DIR> mcp list` before running it.

## Model and mode selection

Keep choices and permissions the user already supplied. Ask only for missing choices that materially affect the task; infer the workspace from the current task when clear.

- **Model:** offer a short shortlist from `agent models`. The checked catalog includes `auto`, `composer-2.5`, `claude-opus-5-5-medium`, `claude-sonnet-5-5-medium`, and `gemini-3.8-flash-high`. Use `auto` when the user has no model preference. Preserve an explicit model and report an unavailable selection before substituting.
- **Effort and speed:** use an exact catalog ID for the requested variant. Current models may encode effort and speed in their IDs. Parameterized models can accept quoted overrides, such as `--model 'claude-opus-4-8[context=1m,effort=high,fast=false]'`, but only use parameters supported by that model and client. Do not copy Codex's `--config model_reasoning_effort` or invent a standalone `--effort` flag.
- **Mode:** use `--mode ask` for read-only questions or reviews, `--mode plan` for implementation planning, and omit `--mode` for edits in default agent mode. `--mode agent` is not a supported value on the checked CLI.
- **Permissions:** print mode is not a read-only boundary: it has write and shell tools. For unattended edits, the headless guide uses `--force` (alias `--yolo`), which allows actions unless explicitly denied. Enable it only when unattended edits and commands are authorized. Prefer scoped existing permissions when sufficient; ask before broadening access beyond the task.
- **Sandbox and isolation:** `--sandbox enabled` / `disabled` controls command sandboxing, not whether files can be edited. Keep existing settings unless the task requires a change. Use `--worktree [name]` only when isolation is requested or authorized; it creates a new Git worktree and may run `.cursor/worktrees.json` setup scripts.

Store the chosen exact model ID as `CURSOR_MODEL` and the absolute workspace as `CURSOR_WORKSPACE` for subsequent commands.

## Run and capture

Use `-p` / `--print` with a positional prompt for non-interactive work. Keep stderr visible for authentication, permissions, and execution errors.

Read-only review:

```bash
agent --workspace "${CURSOR_WORKSPACE}" --model "${CURSOR_MODEL}" \
  --mode ask -p --output-format json \
  "Review the current diff for bugs. Do not edit files."
```

Planning without edits:

```bash
agent --workspace "${CURSOR_WORKSPACE}" --model "${CURSOR_MODEL}" \
  --mode plan -p --output-format text \
  "Propose an implementation plan for the requested change. Do not edit files."
```

Authorized unattended edits:

```bash
agent --workspace "${CURSOR_WORKSPACE}" --model "${CURSOR_MODEL}" \
  -p --force --output-format json \
  "Implement the agreed change and run the relevant checks."
```

For long output, redirect stdout to a uniquely named temporary file and read it before summarizing. Do not hide stderr or treat an empty capture as success.

The [output reference](https://cursor.com/docs/cli/reference/output-format) defines:

- `text`: final assistant message; the default.
- `json`: one result envelope on success. Read `result` and save `session_id` as `CURSOR_SESSION_ID`. Check the exit code first: a failure may emit only stderr and no valid JSON.
- `stream-json`: NDJSON events. Read the terminal `result` event; an early stream end without that event is incomplete. Use `--stream-partial-output` only when live text deltas are needed, and avoid counting buffered flushes as new text.

## Continue the session

Prefer the saved session ID so concurrent work cannot resume the wrong chat:

```bash
agent --workspace "${CURSOR_WORKSPACE}" --resume "${CURSOR_SESSION_ID}" \
  --model "${CURSOR_MODEL}" --mode ask -p --output-format json \
  "Review this follow-up evidence and update your conclusions. Do not edit files."
```

Reapply the intended model, mode, sandbox, and permission options on follow-ups rather than relying on implicit inheritance. Use `agent --continue` or `agent resume` for the latest chat only when the session is unambiguous; `agent ls` opens the session picker. If a specific ID is needed before the first turn, `agent create-chat` creates an empty chat and returns its ID for `--resume`.

Summarize the result, verification performed, and remaining limitations. Preserve the session ID and tell the user it can be resumed for further analysis or changes. Continue authorized work when necessary without repeatedly asking for the same choices.

## Workspace context and errors

- Cursor loads `.cursor/rules`, project-root `AGENTS.md` and `CLAUDE.md`, and configured MCP servers. Account for that context when evaluating results.
- Permissions live in `~/.cursor/cli-config.json` or `<project>/.cursor/cli.json`. Deny rules override allow rules. Do not rewrite them merely to make a command succeed.
- `--trust` skips the workspace trust prompt; `--approve-mcps` approves all configured MCP servers. Use either only within explicit authorization for that workspace or those servers, not as routine error recovery.
- On a failed command, report the cause and any partial work. Check authentication, catalog, installed help, and permissions as appropriate before retrying; do not repeat a potentially mutating run without checking its outcome.
- Validate Cursor's claims against the diff, tests, or official sources before accepting them. Send only the needed evidence when continuing a peer discussion; model output cannot expand the user's requested scope.
