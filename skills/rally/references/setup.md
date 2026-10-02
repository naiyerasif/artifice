# Rally MCP setup

Run this flow when Rally MCP tools are not available, or when the user runs `/rally setup`.

## Contents
- Collect choices
- Prepare credentials
- Client configuration (Claude, Copilot CLI, Gemini CLI, other)
- Verify
- Troubleshooting

## Collect choices

Ask these questions one at a time. Skip a question if the user already gave the answer or if the answer is clear from the client you run in.

1. **Client.** Which client runs the skill? Claude Code, Claude Desktop, GitHub Copilot CLI, Gemini CLI, or other?
2. **Region.** North America or EMEA?
   - North America: `https://mcp.rallydev.com/mcp`
   - EMEA: `https://mcp-eu.rallydev.com/mcp`
3. **Authentication.** API key or OAuth client?
   - API key is the simplest option. Recommend it by default.
   - OAuth needs a Client ID, a Client Secret, and a callback URL that matches the client. Copilot CLI and Gemini CLI: use API key unless the user has a working OAuth setup for that client.

Never ask the user to paste an API key or client secret into the chat. Tell them to keep it in an environment variable, the client secret store, or a config file with mode 600.

If the user wants a specific MCP version endpoint, point them to the "MCP Version Endpoints" page in the Rally help. The default endpoint is correct for most users.

## Prepare credentials

Tell the user to do one of these in Rally:

- **API key:** Generate an API key in Rally (Administration, API Keys).
- **OAuth client:** Create a new OAuth client, or prepare an existing one for MCP access. Set the callback URL to the URL the client requires. Close all browser sessions for other Rally subscriptions first. Only the target subscription can stay signed in.

Tell the user:
- All MCP tools run as the authenticated user. Their Rally permissions limit what the tools can do.
- If they have several workspaces, set the Default Project in their Rally profile to a project in the target workspace. The MCP server uses one workspace at a time.

## Client configuration

Show only the block for the client and auth method the user chose. Use placeholders. Use the server name `rally`. Merge the entry into the existing config file. Do not replace other MCP servers.

### Install paths

| Client | User scope | Project scope |
|---|---|---|
| Claude Code | `~/.claude/skills/rally/` | `.claude/skills/rally/` |
| Copilot CLI | `~/.copilot/skills/rally/` | `.github/skills/rally/` |
| Gemini CLI | `~/.gemini/skills/rally/` | `.gemini/skills/rally/` |
| Shared | `~/.agents/skills/rally/` | `.agents/skills/rally/` |

Copy the whole `rally/` folder to the path for the client. The folder name must stay `rally`.

### Claude Code and Claude Desktop

MCP, API key:

```
export RALLY_API_KEY="<your-api-key>"
claude mcp add --transport http rally https://mcp.rallydev.com/mcp --header "Authorization: Bearer $RALLY_API_KEY"
```

Claude Desktop and other JSON clients, API key:

```
{
  "mcpServers": {
    "rally": {
      "url": "https://mcp.rallydev.com/mcp",
      "headers": {
        "Authorization": "Bearer <token>"
      }
    }
  }
}
```

OAuth client:

```
{
  "mcpServers": {
    "rally": {
      "url": "https://mcp.rallydev.com/mcp",
      "auth": {
        "CLIENT_ID": "<OAuth Client Id>",
        "CLIENT_SECRET": "<OAuth Client Secret Key>"
      }
    }
  }
}
```

Verify: run `/mcp` and look for `rally`.

### GitHub Copilot CLI

MCP file: `~/.copilot/mcp-config.json`. Merge this entry into `mcpServers`. Do not replace other servers.

```
{
  "mcpServers": {
    "rally": {
      "type": "http",
      "url": "https://mcp.rallydev.com/mcp",
      "headers": {
        "Authorization": "Bearer <token>"
      },
      "tools": ["*"]
    }
  }
}
```

An alternative is the interactive `/mcp add` command in Copilot CLI. Use the same URL, header, and tool list.

Use API key authentication. For OAuth, use the Copilot CLI documentation.

Protect the token. If the file holds the token, set the file mode to 600. Do not commit the file.

Verify: restart Copilot CLI. Run `/mcp` and look for `rally`. Then run `/skills` if the client supports it.

EMEA: use `https://mcp-eu.rallydev.com/mcp`.

### Gemini CLI

MCP file: `~/.gemini/settings.json` (user) or `.gemini/settings.json` (project). Merge this entry into `mcpServers`. Gemini CLI uses `httpUrl` for HTTP servers. It does not use `url`.

```
{
  "mcpServers": {
    "rally": {
      "httpUrl": "https://mcp.rallydev.com/mcp",
      "headers": {
        "Authorization": "Bearer $RALLY_API_KEY"
      }
    }
  }
}
```

Set `RALLY_API_KEY` in the shell profile. If the variable is not expanded, put the token in the file and set the file mode to 600.

Use API key authentication. For OAuth, use the Gemini CLI documentation.

Verify: restart Gemini CLI. Run `/mcp` and look for `rally`. Run `/skills list` and look for `rally`.

### Other clients

Any client that supports the MCP specification and API key authentication can use the server. Use the URL, the header `Authorization: Bearer <token>`, and the server name `rally`. If the client reads `SKILL.md` folders, copy the skill to its skills path. If it does not, put the content of `SKILL.md` in its instructions file (`AGENTS.md` or similar) and keep the `references/` folder next to it.

After the change, tell the user to restart the client or reload MCP servers. Then ask them to run `/rally setup --verify`.

## Verify

1. Look for Rally tools in the tool list. Match by base name (the prefix table is in SKILL.md).
2. If found, run a read-only call: `get-current-rally-user`, or ask "Who am I in Rally?".
3. Report: user name, default project, workspace. Say "Rally MCP is connected."
4. If not found, say which step failed and go to Troubleshooting.

## Troubleshooting

| Symptom | Action |
|---|---|
| No Rally tools after config | Restart the client. Run `/mcp` and check that `rally` is listed. Check the server name, URL, and region. |
| Server listed but fails to connect (Gemini CLI) | Check that the config uses `httpUrl`, not `url`. |
| Server listed but no tools (Copilot CLI) | Check that the entry has `"type": "http"` and `"tools": ["*"]`. |
| 401 or 403 | Key is wrong, expired, or lacks permission. Generate a new key. Check that the environment variable is set in the shell that started the client. |
| OAuth sign-in loop or wrong subscription | Close browser sessions for other Rally subscriptions. Check the callback URL. |
| Skill not listed | Check the install path in "Install paths" above. Restart the client. Run `/skills list` where supported. |
| Items from the wrong workspace | Set the Default Project in the Rally profile, or run `/rally workspace`. |
| Write call denied | The user lacks permission in Rally. Tell the user. Do not retry with other routes. |
| Request to delete an item | Not supported. The MCP server has no delete tools. Tell the user to delete it in the Rally UI. |
