# agent-skills

Personal Claude Code agent skills for working with OpenAI Codex CLI, Google Gemini CLI, and Kilo Code CLI.

## Available Skills

| Skill | Description |
|-------|-------------|
| [codex](plugins/personal/skills/codex/SKILL.md) | General-purpose Codex CLI runner — code analysis, refactoring, and automated editing with model/effort/sandbox selection. |
| [codex-review](plugins/personal/skills/codex-review/SKILL.md) | Iterative plan review loop — Claude sends the current plan to Codex, revises based on feedback, and re-submits until Codex approves (up to 5 rounds). |
| [gemini-cli](plugins/personal/skills/gemini-cli/SKILL.md) | General-purpose Gemini CLI runner — code analysis, refactoring, and automated editing with model/approval-mode/sandbox selection. |
| [kilocode-cli](plugins/personal/skills/kilocode-cli/SKILL.md) | General-purpose Kilo Code CLI runner — code analysis, refactoring, reviews, and automated editing with model/agent/permission selection. |

## Installation

### skills.sh

Install all skills:

```sh
npx skills add makeavish/agent-skills
```

Install individual skills:

```sh
npx skills add makeavish/agent-skills --skill codex
npx skills add makeavish/agent-skills --skill codex-review
npx skills add makeavish/agent-skills --skill gemini-cli
npx skills add makeavish/agent-skills --skill kilocode-cli
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

## IDs

- **Marketplace ID**: `makeavish-skills`
- **Plugin ID**: `personal`
- **Repository**: `makeavish/agent-skills`

## Repository Structure

```text
.
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── personal/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           ├── codex/
│           │   └── SKILL.md
│           ├── codex-review/
│           │   └── SKILL.md
│           ├── gemini-cli/
│           │   └── SKILL.md
│           └── kilocode-cli/
│               └── SKILL.md
├── LICENSE
└── README.md
```

## Adding a New Skill

1. Create `plugins/personal/skills/<skill-name>/SKILL.md`
2. Add frontmatter with `name`, `description` (controls auto-trigger), and skill instructions
3. Bump the version in `plugins/personal/.claude-plugin/plugin.json`
4. Run `/reload-plugins` in Claude Code

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
