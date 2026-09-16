# agent-skills

Personal Codex and Claude Code agent skills for writing, plan review, browser-based GPT-6 Pro collaboration, and working with Claude Code CLI, OpenAI Codex CLI, Google Antigravity CLI, and Kilo Code CLI.

## Available Skills

| Skill | Description |
|-------|-------------|
| [antigravity-cli](plugins/personal/skills/antigravity-cli/SKILL.md) | General-purpose Antigravity CLI runner (`agy`, Google's successor to Gemini CLI) — code analysis, refactoring, and automated editing with model/autonomy/sandbox selection. |
| [claude-cli](plugins/personal/skills/claude-cli/SKILL.md) | General-purpose Claude Code CLI runner (`claude`, `claude -p`) — code analysis, refactoring, reviews, automated editing, background agents, and structured output. Defaults to Opus; multiple Fable instances require explicit cost confirmation. |
| [chatgpt-pro](plugins/personal/skills/chatgpt-pro/SKILL.md) | Consults GPT-6 Pro when requested or proactively for stalled hard problems and consequential reasoning decisions. Works with Codex, Claude Code, and other agents with browser UI tools; preserves practical outputs and cross-agent handoffs. Prefers Helium, with the host's managed browser as fallback. |
| [codex](plugins/personal/skills/codex/SKILL.md) | General-purpose Codex CLI runner — code analysis, refactoring, and automated editing with model/effort/sandbox selection across the GPT 5.6 lineup (`gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna`). |
| [codex-plan-review](plugins/personal/skills/codex-plan-review/SKILL.md) | Iterative plan review loop — Claude sends the current plan to Codex, revises based on feedback, and re-submits until Codex approves (up to 5 rounds), with GPT 5.6 model options (`gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna`). |
| [kilocode-cli](plugins/personal/skills/kilocode-cli/SKILL.md) | General-purpose Kilo Code CLI runner — code analysis, refactoring, reviews, and automated editing with model/agent/permission selection. |
| [writing](plugins/personal/skills/writing/SKILL.md) | Draft, rewrite, critique, or unblock human-facing prose while preserving facts, voice, and genre-specific intent. |

## Installation

### skills.sh

Install all skills:

```sh
npx skills add makeavish/agent-skills
```

Install individual skills:

```sh
npx skills add makeavish/agent-skills --skill antigravity-cli
npx skills add makeavish/agent-skills --skill claude-cli
npx skills add makeavish/agent-skills --skill chatgpt-pro
npx skills add makeavish/agent-skills --skill codex
npx skills add makeavish/agent-skills --skill codex-plan-review
npx skills add makeavish/agent-skills --skill kilocode-cli
npx skills add makeavish/agent-skills --skill writing
```

### Claude Code

```sh
/plugin marketplace add makeavish/agent-skills
/plugin install personal@makeavish-skills
```

To update after new releases:

```sh
/plugin marketplace update
/plugin update personal@makeavish-skills
```

### Codex

Add this repository as a Codex plugin marketplace, then install the plugin:

```sh
codex plugin marketplace add makeavish/agent-skills
codex plugin add personal@makeavish-skills
```

### ChatGPT Pro requirements and continuity

The `chatgpt-pro` skill requires browser UI tools, an authenticated ChatGPT session with GPT-6 Pro, and an authorized writable artifact location. It does not depend on Codex-specific tools: Claude Code and other hosts use their own available computer-use or browser tools. A host without those capabilities can prepare a blocked handoff but cannot consult Pro. Install or load the skill in each host that should use it; automatic discovery depends on that host's skill support.

For difficult work, the skill calls for a focused Pro consultation after local investigation identifies a concrete reasoning gap. Routine tasks and user requests to avoid external consultation stay local. New sessions normally save results and restart context under `.agents/pro-sessions/<session-id>/HANDOFF.md`. For a read-only repository, use an authorized durable directory outside it; if none is available, ask for one before browser actions or writes. Existing `.codex/pro-sessions/` handoffs remain resumable in place when writes are permitted. A handoff can target Codex, Claude Code, or another agent that has access to the workspace and artifacts.

## IDs

- **Claude Marketplace ID**: `makeavish-skills`
- **Codex Marketplace ID**: `makeavish-skills`
- **Plugin ID**: `personal`
- **Repository**: `makeavish/agent-skills`

## Repository Structure

```text
.
├── .agents/
│   └── plugins/
│       └── marketplace.json
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── personal/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── .codex-plugin/
│       │   └── plugin.json
│       └── skills/
│           ├── antigravity-cli/
│           │   └── SKILL.md
│           ├── claude-cli/
│           │   └── SKILL.md
│           ├── chatgpt-pro/
│           │   ├── assets/
│           │   │   └── session-handoff-template.md
│           │   ├── evals/
│           │   │   ├── files/
│           │   │   │   ├── api-design-HANDOFF.md
│           │   │   │   ├── auth-review-HANDOFF.md
│           │   │   │   ├── auth-review-browser-mock.md
│           │   │   │   ├── incident-context.md
│           │   │   │   └── rollout.md
│           │   │   ├── evals.json
│           │   │   └── trigger-evals.json
│           │   └── SKILL.md
│           ├── codex/
│           │   └── SKILL.md
│           ├── codex-plan-review/
│           │   └── SKILL.md
│           ├── kilocode-cli/
│           │   └── SKILL.md
│           └── writing/
│               ├── evals/
│               │   └── evals.json
│               └── SKILL.md
├── LICENSE
└── README.md
```

## Adding a New Skill

1. Create `plugins/personal/skills/<skill-name>/SKILL.md`
2. Add frontmatter with `name`, `description` (controls auto-trigger), and skill instructions
3. Bump the version in both `plugins/personal/.claude-plugin/plugin.json` and `plugins/personal/.codex-plugin/plugin.json`
4. Update the skill table and any affected installation or structure documentation in this README
5. Run the Claude and Codex plugin validators
6. Reload or reinstall the plugin in the target client

```text
plugins/personal/skills/my-skill/
└── SKILL.md
```

Conventions:
- `name` in frontmatter must exactly match the directory name
- `description` should explain both what the skill does and when it should trigger
- Move reference material into `references/` subdirectory when the SKILL.md gets large

## License

MIT. See [LICENSE](./LICENSE).
