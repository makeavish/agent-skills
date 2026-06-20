---
name: antigravity-cli
description: Use when the user asks to run Antigravity CLI (the agy command, agy -p) or references Google Antigravity CLI — the successor to Gemini CLI — for code analysis, refactoring, or automated editing
---

# Antigravity CLI Skill Guide

> **Migration note:** Google retired Gemini CLI for AI Pro / AI Ultra / free-tier users on **June 18, 2026** in favor of **Antigravity CLI**, the Go-based successor that shares a unified architecture with the Antigravity 2.0 desktop app. The command is now `agy` (not `gemini`). Organizations on a Gemini Code Assist Standard/Enterprise license or Google Cloud may keep using the legacy Gemini CLI; everyone else should migrate. If the user still expects the old `gemini` command or `npm install -g @google/gemini-cli`, let them know it's deprecated and point them here.

## Preflight Checks

Before running any Antigravity command, verify the CLI is installed and authenticated:

1. Run `agy --version` to confirm installation. If that flag is unrecognized on the installed build, run `agy --help` to confirm the binary exists and discover the exact version syntax. If `agy` is missing, suggest installing it:
   - **macOS / Linux:** `curl -fsSL https://antigravity.google/cli/install.sh | bash`
   - **Windows (PowerShell):** `irm https://antigravity.google/cli/install.ps1 | iex`

   `agy` is a self-contained Go-native binary that self-updates in the background (if your build supports it, `agy update` forces an update — confirm available subcommands with `agy --help`). There is **no confirmed npm / Homebrew / apt package** — the install script is the only verified distribution channel, so do not suggest `npm install` or `brew install`.
2. Check authentication. Unlike legacy Gemini CLI, Antigravity CLI is **OAuth-first for consumer accounts** — there is no supported `GEMINI_API_KEY` / `GOOGLE_API_KEY` / `ANTIGRAVITY_API_KEY` env var path for AI Pro/Ultra/free users today (it's an open feature request, not a shipped feature). Do not steer users to API-key env vars.
   - **Cached Google login (recommended)** — run `agy` interactively once to complete the OAuth flow (it offers `1. Google OAuth` or `2. Use a Google Cloud project`; OAuth opens a browser and you paste an auth code back). Sign in with the Google account tied to the user's AI Pro / Ultra subscription. There is **no `agy login` / `agy auth` subcommand** — auth triggers automatically when no valid session is cached; sign out with the in-session `/logout` command.
   - **Enterprise / Google Cloud** — choose the "Use a Google Cloud project" option at login; quota then bills against that project. Google's transition announcement says paid Gemini / Gemini Enterprise Agent Platform plans remain supported, but the exact key/credential mechanism isn't clearly documented — have enterprise users confirm the supported flow with their Google Cloud admin rather than assuming a service-account or API-key path works.
   - Credentials are stored in the **OS secure keyring** (macOS Keychain, Windows Credential Manager, Linux freedesktop Secret Service / libsecret), not a plaintext token file, and reused across runs once cached.
3. In headless mode, `agy -p` reuses the cached keyring credentials. If auth fails, suggest the user run `agy` interactively first to complete OAuth.
4. **Headless Linux caveat (WSL2 / bare SSH):** the keyring may fail to auto-unlock without a display manager, forcing repeated re-auth. If the user hits this, suggest updating `agy` (e.g. `agy update`, if supported — a fix reportedly landed in an early patch release) and, as a workaround, installing `dbus-x11` / `libsecret` / `gnome-keyring` and unlocking `gnome-keyring-daemon` at shell startup. Over SSH, `agy` prints an auth URL to open on the local machine.

## Running a Task

1. Ask the user (via `AskUserQuestion`) which **model** to run and how much **autonomy** to grant the agent — in a **single prompt with both questions**.
   - **Model:** default to **`Gemini 3.5 Flash (High)`** (the CLI default — fast and capable). Offer **`Gemini 3.1 Pro (High)`** for deeper reasoning on hard tasks, and note that **`Claude Sonnet 4.6 (Thinking)`**, **`Claude Opus 4.6 (Thinking)`**, and **`GPT-OSS 120B (Medium)`** are also selectable — Antigravity is not Gemini-only. Pass the **friendly label** to `--model` (e.g. `--model "Gemini 3.1 Pro (High)"`).
   - **Autonomy:** for **read-only review/analysis**, run a plain `agy -p` (no skip flag needed). For tasks that **edit files or run commands**, headless mode cannot answer interactive permission prompts, so pass `--dangerously-skip-permissions` (the replacement for the old `--yolo`). Pair it with `--sandbox` whenever possible, especially in untrusted repos.
2. Assemble the command with the appropriate options:
   - `--model "<FRIENDLY LABEL>"` (model selection; a short `-m` alias is **not** confirmed, so use the long form)
   - `-p, --prompt "<your prompt here>"` (also `--print`) — non-interactive / headless mode
   - `--dangerously-skip-permissions` (auto-approve all tool/edit permissions; required for autonomous edits in headless mode — there is **no `--yolo` and no `--approval-mode` launch flag** in `agy`)
   - `--sandbox` (run the session with terminal restrictions; a short `-s` alias is **not** confirmed)
   - `--add-dir <DIR>` (repeatable — extend the workspace beyond cwd; closest equivalent to the old `--include-directories`)
   - `--continue` / `-c` (resume the most recent conversation) or `--conversation <ID>` (resume a specific one) — see [Following Up](#following-up)
3. **Working Directory:** Antigravity CLI has **no `-C` flag**. Run `agy` from the target repository's directory (use `cd <target-dir> &&` before the command), or extend the workspace with `--add-dir <DIR>`.
4. **Output format:** there is **no working JSON output flag.** `--output-format json` currently errors out (`flags provided but not defined`) — headless output is **plain text only**. Do not pass `--output-format` / `--format` / `-o`; capture and parse the plain-text stdout instead. (Google's codelab shows a JSON example, but it does not work on shipped builds — re-check with `agy --help` before assuming otherwise.)
5. **Stderr Handling:** Do NOT append `2>/dev/null` by default. `agy` may emit auth errors, sandbox failures, and tool-execution errors to stderr in headless mode. Only suppress stderr if the user explicitly requests it or after confirming it contains only UI artifacts.
6. Run the command, capture stdout, and summarize the outcome for the user.
7. **After `agy` completes**, inform the user they can resume the conversation (`agy --continue` / `agy --conversation <id>`) or start another task at any time.

### Output Handling

- For short tasks, capture stdout directly.
- For large output, redirect to a temp file:

  ```bash
  cd <target-dir> && agy --model "Gemini 3.5 Flash (High)" --sandbox -p "Analyze this codebase" > /tmp/agy-output-$(date +%s).txt
  ```

  Then read and summarize the file.
- Prefer `-p "prompt"` over piped input — it is more explicit and avoids shell quoting issues.
- **Non-TTY gotcha (reported):** in some CI / pipe / command-substitution contexts, `agy -p` may silently drop the final stdout chunk while still exiting `0`. If output looks truncated, retry through a pseudo-TTY, e.g. `script -qec 'agy -p "..."' /dev/null | tee out.txt` (Linux), then strip ANSI codes. Treat this as a workaround, not guaranteed behavior.

### Quick Reference

| Use case | Key flags |
| --- | --- |
| Read-only review or analysis | `--sandbox -p "..."` |
| Apply edits / run tools autonomously | `--dangerously-skip-permissions -p "..."` |
| Sandboxed execution | Add `--sandbox` |
| Pick a model | `--model "Gemini 3.1 Pro (High)"` |
| Extend workspace to other dirs | `--add-dir <DIR>` (repeatable) |
| Resume most recent conversation | `--continue` / `-c` |
| Resume a specific conversation | `--conversation <ID>` |

> **Flags that do NOT exist in `agy`** (carried over from Gemini CLI muscle memory or hallucinated by third-party blogs — do not use): `--yolo`, `--approval-mode`, `--checkpointing`, `--output-format json`, `-C`, `--include-directories`, an `agy run` subcommand, `--prompt-file`, `--yes`. When in doubt, confirm against `agy --help` on the installed version.

## Workspace Security

- On first run in a project, `agy` shows a per-workspace trust prompt — **"Do you trust the contents of this project?"** — and requires approval before it can read, edit, or execute files. Be deliberate about trusting unknown repos.
- In untrusted repositories, always enable `--sandbox`.
- Permission presets (set interactively via `/permissions` or `/config`, or persisted in `settings.json` under `toolPermission`) reportedly include `request-review` (default), `proceed-in-sandbox`, `always-proceed`, and `strict` (read-only) — verify the exact names in-session via `/permissions`, as they may differ by version. These are configured in-session, not via launch flags.
- `agy` reads `GEMINI.md`, `AGENTS.md`, and `~/.gemini/GEMINI.md` context files; non-secret config lives in `~/.gemini/antigravity-cli/settings.json`. Warn the user about auto-loaded local configuration.
- Avoid enabling telemetry or prompt logging unless the user explicitly requests it.

## Following Up

- After every `agy` command, immediately use `AskUserQuestion` to confirm next steps, collect clarifications, or decide whether to run another task.
- Restate the chosen model, autonomy level, and sandbox setting when proposing follow-up actions.
- **Session resume now works headlessly** (a real improvement over legacy Gemini CLI, where `/chat` was interactive-only): continue the most recent conversation with `agy --continue -p "<follow-up>"`, or resume a specific one with `agy --conversation <ID> -p "<follow-up>"`. Use this to carry context across multiple `agy` invocations instead of re-sending everything each time.

## Migrating From Gemini CLI

If the user is moving an existing Gemini CLI setup over, note what Antigravity handles automatically vs. manually:

- **Auto-migrated on first launch:** Extensions become **Antigravity plugins** (convert Gemini extensions with a command reported as `agy plugin import gemini` — confirm the exact syntax via `agy --help`); session tokens move into the OS keyring; visual settings are mapped. `GEMINI.md` / `AGENTS.md` are read unchanged.
- **Manual step — workspace skills:** move them from `.gemini/skills/` → `.agents/skills/` (global shared skills go in `~/.gemini/antigravity-cli/skills/`).
- **Manual step — MCP config:** MCP servers move out of `~/.gemini/settings.json` into a dedicated `mcp_config.json` (global `~/.gemini/config/mcp_config.json`, workspace `.agents/mcp_config.json`), and the remote-server key is **renamed `url` / `httpUrl` → `serverUrl`**.
- Antigravity retains Agent Skills, Hooks (JSON lifecycle interceptors), and Subagents from Gemini CLI.

## New Capabilities Worth Knowing

- **Asynchronous subagents** are the headline upgrade: the main agent can delegate parallel research, builds, and validation to background subagents without locking the session. This is an **in-session framework**, not a launch flag — managed with slash commands like `/agents` (Agent Manager), `/tasks` (background task logs / terminate), and `/btw <query>` (background side question), plus `Ctrl+K` to fast-approve a pending subagent action. There is **no `--background` / `--async` launch flag.**
- **Conversation rollback:** `/rewind` (alias `/undo`) and `/fork` (alias `/branch`) inside a session.
- **Shell mode** via `!`; an **Artifacts** system for plans / task-tracking / verification; `/usage` shows remaining quota; `/models` lists and switches models.

## Critical Evaluation of Antigravity Output

Antigravity is powered by Google (and selectable Anthropic / open) models, each with their own knowledge cutoffs and limitations. Treat it as a **colleague, not an authority**.

### Guidelines

- **Trust your own knowledge** when confident. If the agent claims something you know is incorrect, push back directly.
- **Research disagreements** using WebSearch or documentation before accepting its claims.
- **Remember knowledge cutoffs** — the underlying models may not know about recent releases, APIs, or changes after their training data.
- **Don't defer blindly** — evaluate suggestions critically, especially regarding:
  - Model names and capabilities
  - Recent library versions or API changes
  - Best practices that may have evolved

### When It's Wrong

1. State your disagreement clearly to the user.
2. Provide evidence (your own knowledge, web search, docs).
3. Optionally run a new `agy` task to discuss the disagreement. **Identify yourself as Claude** so the peer AI knows it's a discussion between assistants:

   ```bash
   agy --model "Gemini 3.5 Flash (High)" --sandbox -p "This is Claude (<your current model name>) following up on a prior analysis. I disagree with [X] because [evidence]. What's your take?"
   ```

4. Frame disagreements as discussions, not corrections — either AI could be wrong.
5. Let the user decide how to proceed if there's genuine ambiguity.

## Error Handling

- Stop and report failures whenever `agy --version` or an `agy -p` command exits non-zero; request direction before retrying.
- Before you use high-impact flags (`--dangerously-skip-permissions`, or disabling `--sandbox`) ask the user for permission using `AskUserQuestion` unless it was already given.
- When output includes warnings or partial results, summarize them and ask how to adjust using `AskUserQuestion`.
- **Capacity vs. quota (429s):** distinguish the two failure modes when the user hits a 429:
  - `MODEL_CAPACITY_EXHAUSTED` ("No capacity available for model …") is **server-side** and affects all accounts (even paid tiers at peak). The fix is to **wait and retry with backoff**, or switch to a lighter model (e.g. `Gemini 3.5 Flash`) — it is not the user's quota.
  - `QUOTA_EXCEEDED` / `RESOURCE_EXHAUSTED` is **account-specific** quota depletion — waiting for the quota window to reset (or using a different account) is what helps.
- This skill operates statelessly across invocations unless you explicitly resume with `--continue` / `--conversation <ID>`. CLI specifics (flags, version syntax, model labels) evolve between releases — when something behaves unexpectedly, confirm against `agy --help` on the user's installed version before concluding.
