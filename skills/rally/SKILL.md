---
name: rally
description: "Read, create, update, link, plan, audit, and report on Broadcom Rally work items (stories, features, defects, tasks, test cases, iterations, releases) through the Rally MCP server, using the /rally command. Subcommands: setup, fetch, find, mine, standup, create (--story, --feature, --defect, --tasks, --testcase), update, comment, link, breakdown, audit, deps, report, handoff, workspace, help. Use whenever the user types /rally, gives a Rally ID (US1234, DE567, TA89, F12, I3, TC45), or asks about Rally backlog, sprint, standup, release, PI, or QA handoff work, even if they do not say Rally or MCP. If the Rally MCP server is not connected, guides the user through setup first."
compatibility: "Requires an agent with MCP support and the Rally MCP server (https://mcp.rallydev.com/mcp or https://mcp-eu.rallydev.com/mcp) with an API key or OAuth client, and network access."
metadata:
  version: "1.1"
  mcp-server: rally
---

# Rally

Subcommand syntax: `/rally <subcommand> [target] [--flags]`. The client decides how the user starts the skill. Read the same syntax from a slash command or from plain words.

This skill drives the Rally MCP server. All Rally MCP tools run as the authenticated user, with the same permissions as the Rally UI. The server has no delete tools. Never try to delete a Rally artifact.

## Step 0 - Preflight (every invocation)

1. Look for the Rally MCP tools. Base names: `rally_search_workflows`, `rally_get_workflow`, `rally_openapi_search_tools`, `rally_openapi_invoke_tool`. Each client adds its own prefix. Match by base name.

   | Client | Typical form |
   |---|---|
   | Claude | `mcp__rally__rally_openapi_invoke_tool` |
   | Copilot CLI | `rally-rally_openapi_invoke_tool` |
   | Gemini CLI | `mcp_rally_rally_openapi_invoke_tool` |

2. If no Rally tools exist, stop. Read [references/setup.md](references/setup.md) and run the setup flow. Do not guess data. Do not use other tools in place of Rally.
3. If Rally tools exist, confirm identity once per session with a read-only call (`get-current-rally-user`, or the nearest user-lookup operation). Keep the user name, default project, and workspace.
4. If a Rally call fails with an auth error (401, 403, "unauthorized", "expired"), read the Troubleshooting section of `references/setup.md`.

## Run any operation

1. Call `rally_search_workflows` with a short task description. If a curated workflow matches, call `rally_get_workflow` and follow its steps.
2. If none matches, call `rally_openapi_search_tools` to find the operation. Then call `rally_openapi_invoke_tool` with `argumentsJson` (`operation` plus `pathParams`, `queryParams`, `body`). Set `includeSchemas` to true when the body shape is unknown. Use `dryRun` on writes when the server supports it.

## Ground rules

- **Missing input: ask.** Ask one short question at a time. Offer 2 to 4 options when the set is small. Use the client ask-user tool if one exists. If not, ask in plain text with numbered options. Never invent IDs, owners, projects, iterations, or estimates.
- **Read before write.** Fetch the item first for any change to an existing item.
- **Draft, confirm, write.** Show the draft. Ask "Create these now?". Write only after a clear yes. For batches, show the full list first.
- **Verify after write.** Fetch the new or changed item. Report formatted ID, name, state, owner, parent, and a link if the response has one.
- **Succinct output.** Lead with the result. Use tables for lists. Do not paste raw JSON unless asked.
- **Ground content in the source.** Base descriptions and acceptance criteria on the item's Description, Notes, Discussion, and attachments. Do not write generic filler.

## Gotchas

- ID prefixes are defaults: `US` story, `DE` defect, `TA` task, `TC` test case, `TS` test set, `F` feature, `I` initiative, `T` theme. A subscription can change them. If a prefix is unknown, search by FormattedID.
- Feature, initiative, and theme are portfolio item types. Find their type paths through API discovery. Do not hard-code them.
- Valid states, severity values, and the "ready for test" flag depend on the workspace. Show the valid values and ask when the user gives a value that does not match.
- The MCP server uses one workspace at a time. It uses the default project from the user profile. Use `/rally workspace` to switch.
- A permission error on a write is final. Tell the user. Do not try another route.
- Reports come from raw artifact data, not from Rally's reporting engine. Label them "custom report from live data". Tell the user to compare the numbers with a built-in Rally report before use.
- Read text-like attachments (md, txt) for context. Name other attachments and skip them.

## Subcommands

| Subcommand | Purpose |
|---|---|
| `/rally setup [--verify]` | Configure or verify the Rally MCP connection |
| `/rally fetch <id>` | Show one item with context |
| `/rally find <text or filters>` | Search items |
| `/rally mine` | Show my assigned work |
| `/rally standup` | Standup summary |
| `/rally create --story\|--feature\|--defect\|--tasks\|--testcase` | Create items |
| `/rally update <id>` | Change fields |
| `/rally comment <id> "<text>"` | Post a discussion comment |
| `/rally link <id> --pr <url>` | Link a pull request |
| `/rally breakdown <id>` | Propose children (Initiative to Features, Feature to Stories) |
| `/rally audit <id>` | Check readiness and acceptance criteria |
| `/rally deps <id>` | Dependency and risk scan |
| `/rally report --iteration\|--release\|--portfolio\|--defects` | Custom status reports |
| `/rally handoff <id> --qa` | Hand a story to QA |
| `/rally workspace [name]` | Show or switch workspace |
| `/rally help [subcommand]` | Show usage |

If the user types `/rally` alone, or an unknown subcommand, show this table and ask what they want. If the request is about Rally but not in `/rally` form, map it to the closest subcommand and run it.

## When to load reference files

| Situation | Read |
|---|---|
| `setup`, no Rally tools, auth error, client install path, MCP config block, `/rally` not recognized | [references/setup.md](references/setup.md) |
| Any subcommand except `help` and `setup` | [references/subcommands.md](references/subcommands.md) (the section for that subcommand only) |
| `create`, `breakdown`, `handoff` with test cases, or any new item draft | [references/templates.md](references/templates.md) |

Do not load a file that the task does not need.
