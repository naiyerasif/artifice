---
name: confluence
description: Read, search, create, edit, and manage Confluence pages, blog posts, comments, attachments, spaces, and permissions through the Atlassian MCP server. Use the `/confluence` command with subcommands. Works in Claude, GitHub Copilot, and Gemini CLI, and in any other agent that supports Agent Skills and MCP. Use this skill whenever the user types `/confluence`, mentions Confluence, a wiki page, a space, CQL, or an atlassian.net/wiki URL, or asks to find, summarize, draft, update, comment on, move, archive, export, or share documentation that lives in Confluence, even if the user does not say "Confluence". If the Atlassian MCP is not configured, this skill starts the setup flow first.
compatibility: Requires an agent with Agent Skills support and the Atlassian MCP server (https://mcp.atlassian.com/v2/mcp). The skill guides MCP setup if it is missing.
metadata:
  version: "1.0"
  mcp-server: atlassian-rovo-v2
---

# Confluence

Use the Atlassian MCP server (v2) to work with Confluence.
The primary command is `/confluence`.

## Step 0: First-run gate

Do this before every subcommand, except `setup` and `help`.

1. Look in the available tools for the Atlassian MCP. Match tool names that end with or contain `getAccessibleAtlassianResources` or `getConfluenceContent`. Ignore the prefix. Each client adds its own prefix (for example `mcp__atlassian__`, `mcp_atlassian_`, or `atlassian_`).
2. If no match exists:
   - Say: "The Confluence MCP is not configured."
   - If the client has a connector or extension directory tool, search it for `atlassian`. Offer the result.
   - Otherwise, read `references/setup.md` and run the setup flow.
   - Stop after setup. Tell the user to sign in, then run the original command again.
3. If a match exists, call `getAccessibleAtlassianResources`.
   - Keep the `cloudId`. Every tool call needs it.
   - If more than one site returns, ask which site to use. Reuse the answer for the session.
4. If a call fails with an authentication error, read the "Sign in again" table in `references/setup.md`. Give the user the step for their client. Stop.

## Client compatibility

This skill uses the open Agent Skills format. The same folder works in Claude, GitHub Copilot, and Gemini CLI.

- Tool names: the MCP server adds a client-specific prefix. Always match by the tool name at the end.
- Questions: ask in plain text. If the client has a choice or question tool, use it. If not, give a numbered list and let the user reply with a number.
- Approval: the client may also ask the user to approve each tool call. This does not replace the preview step in the Rules section. Do both.
- Shell: some subcommands return a `curl` command (`attach`). Run it only if a shell with network access exists. If not, give the command to the user.
- Invocation: `/confluence <subcommand>` works in clients that run skills as slash commands. In every client, a plain request such as "find the onboarding page in Confluence" also loads this skill.
- Do not name a client in replies unless the user asks about setup.

## Command syntax

```
/confluence <subcommand> [action] [target] [options]
```

- `target` is a page URL, a numeric ID, or a title.
- Single-action subcommands take the target first: `/confluence read <target>`.
- Grouped subcommands take the action first: `/confluence comment add <target>`.
- `/confluence` with no subcommand: show the table below and ask what the user wants to do.

| Subcommand | Purpose |
|---|---|
| `setup` | Configure the Atlassian MCP |
| `status` | Check connection, user, sites, and tool access |
| `help` | Show subcommands and examples |
| `search` | Find content with CQL |
| `read` | Read or summarize one item |
| `list` | List content of one type in a space |
| `spaces` | List, view, or create spaces; manage space instructions |
| `create` | Create a page, blog, live doc, whiteboard, database, embed, smart link, or folder |
| `update` | Edit content, set status, or convert page mode |
| `comment` | List, add, reply, edit, resolve, reopen, or react |
| `history` | List, view, diff, or restore versions |
| `attach` | List, get, download, or upload attachments |
| `export` | Export content as PDF or Word |
| `move` | Move content to a new parent or position |
| `copy` | Copy content to a new parent |
| `archive` | Archive content |
| `unarchive` | Restore archived content |
| `access` | View or change permissions, restrictions, and public links |
| `tasks` | List, view, complete, or reopen inline tasks |
| `labels` | Add labels to content |
| `follow` | Star or watch pages, spaces, and labels |
| `templates` | List or view templates and blueprints |
| `visual` | Create or edit infographics and interactive visualizations |

For the inputs, follow-up questions, and tools of each subcommand, read `references/commands.md`.

## How the MCP exposes tools

- Primary tools are visible directly: `atlassianUserInfo`, `getAccessibleAtlassianResources`, `discover`, `executeRead`, `executeWrite`, `executeDestructive`, `getConfluenceContent`, `createConfluenceContent`, `updateConfluenceContent`, `searchConfluence`.
- All other Confluence tools are deferred. To use one:
  1. Call `discover` with a short natural-language request (for example "list comments on a page").
  2. Read the returned `toolSchema` and operations.
  3. Run the tool with `executeRead`, `executeWrite`, or `executeDestructive`. Use the type that matches the operation.
- Before you write page or whiteboard content, call `getContentFormatGuide` through `executeRead`. Do this once per session. Write content in the format it gives.

## Rules

### Ask follow-up questions
- If a required input is missing, ask. Ask one question at a time.
- Give short choices when the set is small (for example: page, blog, whiteboard). Use the client's choice tool if it has one.
- Do not ask for information that you can get from the input. A Confluence URL already holds the site, space, and page ID.
- Do not guess a target for any write, move, archive, or access change.

### Resolve a target
1. URL: extract the page ID and use it.
2. Numeric ID: use it.
3. Title: call `searchConfluence` with a CQL title query, scoped by space if the user gave one.
   - One match: use it.
   - More than one match: show up to five (title, space, last modified) and ask which one.
   - No match: say so. Ask for a space, a URL, or other words.

### Confirm before you change anything
- Before `create`, `update`, `move`, `copy`, `comment`, `labels`, and `attach upload`: show a short preview of what will change. Wait for the user to agree.
- Always confirm, with the exact target named, before these actions: `archive`, `history restore`, `access` (all write actions), `spaces create`, `spaces instructions set`.
- State the risk for these:
  - `access replace` removes every grant that is not in the new set.
  - `access public enable` lets anyone with the link view the page without sign-in.
- Do not batch destructive actions. Do one at a time.

### Space instructions
Before you create or update content in a space, call `getConfluenceSpaceInstructions` for that space. If instructions exist, follow them.

### Cost
- `searchConfluence` (CQL) is the default search.
- The Rovo `search` tool is semantic and can use up to 10 Rovo credits per call. Use it only if the user asks for natural-language search, or for search across Jira and other apps.

### Permissions and errors
- The Atlassian admin controls access by permission group: `read_confluence`, `write_confluence`, `search_confluence`. Each group needs its own scope (`read:confluence:agent-interface`, `write:confluence:agent-interface`, `search:confluence:agent-interface`).
- If a call is denied, name the missing group and tell the user to ask their Atlassian admin. Do not retry.
- If a call fails for another reason, show the error in one line. Offer one next step.

### Output
- Keep replies short.
- Always include the page title and URL when you refer to content.
- Do not print raw JSON. Summarize fields in plain text or a short list.
- Do not add code comments in any code you write for the user.
- Never ask the user to paste an API token into the chat.
