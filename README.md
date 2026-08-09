# agent-skills

Personal Codex and Claude Code agent skills for writing, plan review, and working with Claude Code CLI, OpenAI Codex CLI, Google Antigravity CLI, and Kilo Code CLI.

## Available Skills

| Skill | Description |
|-------|-------------|
| [antigravity-cli](plugins/personal/skills/antigravity-cli/SKILL.md) | General-purpose Antigravity CLI runner (`agy`, Google's successor to Gemini CLI) — code analysis, refactoring, and automated editing with model/autonomy/sandbox selection. |
| [claude-cli](plugins/personal/skills/claude-cli/SKILL.md) | General-purpose Claude Code CLI runner (`claude`, `claude -p`) — code analysis, refactoring, reviews, automated editing, background agents, and structured output. Defaults to Opus; multiple Fable instances require explicit cost confirmation. |
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
