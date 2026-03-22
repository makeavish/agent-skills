# agent-skills

Personal Claude Code plugin marketplace with custom agent skills.

## Setup

Add to `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "makeavish-skills": {
      "source": {
        "source": "directory",
        "path": "/path/to/agent-skills"
      }
    }
  },
  "enabledPlugins": {
    "personal@makeavish-skills": true
  }
}
```

Then run `/reload-plugins` in Claude Code.

## Skills

### `/codex-review`

Iterative plan review between Claude and OpenAI Codex CLI. Claude sends the current plan to Codex, revises based on feedback, and re-submits until Codex approves — up to 5 rounds.

**Flow:**
1. Asks for Codex model and reasoning effort (single `AskUserQuestion`)
2. Writes current plan to a temp file
3. Runs `codex exec` in read-only sandbox
4. Presents Codex's feedback; if `VERDICT: REVISE`, Claude revises and re-submits via session resume
5. Continues until `VERDICT: APPROVED` or 5 rounds reached
6. Cleans up temp files

**Models:** `gpt-5.4`, `gpt-5.3-codex-spark`, `gpt-5.3-codex`
**Effort:** `xhigh`, `high`, `medium`, `low`

Requires [Codex CLI](https://github.com/openai/codex): `npm install -g @openai/codex`

## Structure

```
agent-skills/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    └── personal/
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            └── <skill-name>/
                └── SKILL.md
```

To add a new skill: create `plugins/personal/skills/<skill-name>/SKILL.md` with a frontmatter `name`, `description`, and `user_invocable: true`, then run `/reload-plugins`.
