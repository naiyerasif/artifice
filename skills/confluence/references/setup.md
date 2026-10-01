# Setup: Atlassian MCP (v2)

Use this file for `/confluence setup` and when the first-run gate finds no Atlassian MCP.

## Facts

- Server URL: `https://mcp.atlassian.com/v2/mcp`
- Confluence tools work only with Atlassian MCP v2.
- Default sign-in: OAuth 2.1 in a browser.
- Option: API token sign-in. It works only if the organization admin enabled it.
- An Atlassian cloud site with Confluence is required.
- Gateway option: `https://mcp.atlassian.com/v2/mcp?tools=all` returns a paginated flat list of all tools. Use it only if an MCP gateway needs every tool listed.

## Flow

1. Say: "The Confluence MCP is not configured. I will help you set it up."
2. Ask one question: "Which client do you use?" Choices: Claude Code, Claude Desktop or Claude.ai, Gemini CLI, GitHub Copilot in VS Code, GitHub Copilot CLI, Other.
3. Give only the steps for that client (see below).
4. If the client has a shell and the user agrees, offer to run the command. Run it only after the user agrees.
5. Tell the user to complete the sign-in in the browser.
6. Tell the user to restart the client or reload MCP servers if the tools do not appear.
7. Ask the user to run the original command again. If the tools are now visible, continue without asking.

## Client steps

### Claude Code

```
claude mcp add --transport http atlassian https://mcp.atlassian.com/v2/mcp
```

Open a Claude Code session. Run `/mcp`. Select `atlassian` and sign in.

### Claude Desktop or Claude.ai

1. Open Settings, then Extensions.
2. Select Browse extensions, then Plugins.
3. Search for Atlassian. Install it.

Config file option:

```
{
  "mcpServers": {
    "atlassian": {
      "url": "https://mcp.atlassian.com/v2/mcp"
    }
  }
}
```

### Gemini CLI

```
gemini mcp add --transport http --scope user atlassian https://mcp.atlassian.com/v2/mcp
```

- Always add `--scope user`. Without it, Gemini CLI writes the server to the current project only.
- Config file option. Edit `~/.gemini/settings.json` (user) or `.gemini/settings.json` (project):

```
{
  "mcpServers": {
    "atlassian": {
      "httpUrl": "https://mcp.atlassian.com/v2/mcp"
    }
  }
}
```

- Use the key `httpUrl`. Gemini CLI ignores the keys `url` and `serverUrl`.
- Start `gemini`. Run `/mcp auth atlassian` and sign in.
- Run `/mcp list` to check that the server is connected.
- If tools are missing, check `includeTools` and `excludeTools` for the server in `settings.json`. Remove any filter that hides the Confluence tools.

### GitHub Copilot in VS Code

Quick install: `https://insiders.vscode.dev/redirect/mcp/install?name=atlassian&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.atlassian.com%2Fv2%2Fmcp%22%7D`

Manual option. Create `.vscode/mcp.json` in the project (or use the user-level MCP file):

```
{
  "servers": {
    "atlassian": {
      "type": "http",
      "url": "https://mcp.atlassian.com/v2/mcp"
    }
  }
}
```

- Open Copilot Chat. Select Agent mode. MCP tools appear only in Agent mode.
- Start the `atlassian` server from the MCP server list in the editor. Sign in when prompted.
- Reload the VS Code window if the tools do not appear.

### GitHub Copilot CLI

In the CLI, run `/mcp add`. Select HTTP. Enter the URL `https://mcp.atlassian.com/v2/mcp`.

Config file option. Edit `~/.copilot/mcp-config.json`:

```
{
  "mcpServers": {
    "atlassian": {
      "type": "http",
      "url": "https://mcp.atlassian.com/v2/mcp"
    }
  }
}
```

- Do not set a `tools` filter. A filter can hide the Confluence tools.
- Copilot Business and Enterprise: an organization policy can block MCP servers. If the server does not load, ask the Copilot admin to allow MCP servers.

### Other MCP clients

Add a remote HTTP (streamable HTTP) MCP server with this URL: `https://mcp.atlassian.com/v2/mcp`.

Known commands for other clients:
- Codex: `codex mcp add atlassian --url https://mcp.atlassian.com/v2/mcp`
- Windsurf: add `"serverUrl": "https://mcp.atlassian.com/v2/mcp/"` under `mcpServers.atlassian`.
- Cursor: install the Atlassian plugin from the Cursor Marketplace.

## API token sign-in (optional)

- Use only if OAuth is not possible and the admin enabled token sign-in.
- Create a token at `https://id.atlassian.com/manage-profile/security/api-tokens`.
- Follow the official steps: `https://support.atlassian.com/atlassian-ai-gateway/docs/configure-authentication-via-api-token/`.
- Tell the user to put the token in the client configuration or an environment variable. Never ask for the token in the chat.

## Verify

After sign-in, run these checks:

1. Call `atlassianUserInfo`. Show the user name and email.
2. Call `getAccessibleAtlassianResources`. List the sites. Keep the `cloudId`.
3. Report: "Connected as <name> on <site>."

## Sign in again

| Client | Step |
|---|---|
| Claude Code | Run `/mcp`. Select `atlassian`. Sign in. |
| Claude Desktop or Claude.ai | Open the Atlassian connector or extension. Disconnect, then connect again. |
| Gemini CLI | Run `/mcp auth atlassian`. |
| GitHub Copilot in VS Code | Restart the `atlassian` server from the MCP server list. Sign in when prompted. |
| GitHub Copilot CLI | Remove the server. Add it again with `/mcp add`. Sign in. |
| Other | Remove the server. Add it again. Sign in. |

## Troubleshoot

| Symptom | Action |
|---|---|
| Tools do not appear | Restart the client or reload MCP servers. In Copilot in VS Code, use Agent mode. |
| Sign-in loop or old credentials | Clear cached `clientId` and `.well-known` credentials in the client. Sign in again. |
| Only old Jira or Confluence tools appear | The client points to v1. Change the URL to `https://mcp.atlassian.com/v2/mcp`. |
| API token sign-in is rejected | The admin has not enabled it. Use OAuth 2.1. |
| `Access denied` on a tool | The admin disabled the permission group. Ask the admin to enable `read_confluence`, `write_confluence`, or `search_confluence`. |
| No site in the site list | The account has no Confluence cloud site, or the user did not grant access to the site during sign-in. Sign in again and select the site. |
| Gemini CLI shows no server | Check that the key is `httpUrl`. Check the scope (user or project). |
| Copilot blocks the server | An organization policy can block MCP. Ask the Copilot admin. |

More help: `https://support.atlassian.com/atlassian-ai-gateway/docs/troubleshoot-and-verify-your-setup/`

## Security note

MCP clients act with the user's existing permissions. Tell the user: use least privilege, review high-impact changes before they confirm, and check audit logs for unusual activity.
