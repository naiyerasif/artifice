# Install

This folder follows the open Agent Skills standard. It has no client-specific files. Install it once. Every client that reads the standard can use it.

## One copy

```
mkdir -p ~/.agents/skills
cp -r confluence ~/.agents/skills/confluence
```

Clients that read `~/.agents/skills/`: GitHub Copilot, Gemini CLI, and other clients that follow the standard.

## Clients with their own path

Link the same folder. Edit the skill in one place only.

```
ln -s ~/.agents/skills/confluence ~/.claude/skills/confluence
```

| Client | Extra step |
|---|---|
| Claude Code | Run the `ln` command above. |
| Claude.ai and Claude Desktop | Open `confluence.skill` and select Save skill. |
| Copilot, Gemini CLI | None. |

## Use

- Ask in plain words, for example: "Find the onboarding page in Confluence."
- Or use the skill syntax: `/confluence <subcommand>`. Try `/confluence help`.

## First run

Run `/confluence status`. If the Atlassian MCP is not configured, the skill asks which client you use and shows the steps. See `references/setup.md`.
