# Command reference

Contents: setup, status, help, search, read, list, spaces, create, update, comment, history, attach, export, move, copy, archive, unarchive, access, tasks, labels, follow, templates, visual.

Each entry gives: usage, required inputs, follow-up questions, tools, and output.
"Primary" tools are visible directly. All other tools run through `discover` and `executeRead` / `executeWrite` / `executeDestructive`.
Every tool call needs the `cloudId` from the first-run gate.

---

## setup

Usage: `/confluence setup`
Action: read `setup.md` and run the flow. If the MCP is already connected, run `status` and ask: "The MCP is connected. Do you want to reconnect or change the client?"

## status

Usage: `/confluence status`
Tools: `atlassianUserInfo`, `getAccessibleAtlassianResources`, `discover`.
Steps:
1. Call `atlassianUserInfo` and `getAccessibleAtlassianResources`.
2. Call `discover` with "confluence write tools". If write tools return, write access is on. Repeat for "confluence search" and "confluence read".
Output: user, sites, and a short line for each group: `read_confluence`, `write_confluence`, `search_confluence` (available or not available).

## help

Usage: `/confluence help [subcommand]`
Output: the subcommand table from `SKILL.md`. With a subcommand, show its usage and one example from this file.

---

## search

Usage: `/confluence search <text> [--space KEY] [--type page|blogpost] [--label NAME] [--by USER] [--since YYYY-MM-DD]`
Required: search text.
Ask if missing: "What do you want to find?"
Tool: `searchConfluence` (Primary).
Steps:
1. Build a CQL query. Examples:
   - `text ~ "onboarding" AND type = page ORDER BY lastmodified DESC`
   - `space = "ENG" AND label = "runbook" AND lastmodified >= "2026-01-01"`
   - `title ~ "Q3 plan" AND space = "FIN"`
2. Run the search.
3. If there are zero results, remove one filter at a time and try again. Tell the user which filter you removed.
Output: numbered list with title, space, last modified, URL. End with: "Which one do you want to read?"
Note: for natural-language or cross-app search, use the Rovo `search` tool only if the user asks. It can use up to 10 Rovo credits.

## read

Usage: `/confluence read <target> [--summary] [--macros] [--version N]`
Required: target.
Ask if missing: "Which page? Give a URL, an ID, or a title."
Tools: `getConfluenceContent` (Primary). With `--macros`: `resolveConfluenceContentMacros`. With `--version`: `getConfluenceContentVersion`.
Works for pages, blog posts, live docs, comments, whiteboards, embeds, databases, and folders.
Output:
- Default: title, space, last editor and date, then the body.
- `--summary`: five bullets or fewer.
- For a long body, show a short summary and ask: "Do you want the full text?"

## list

Usage: `/confluence list <space> [--type page|blogpost|whiteboard|database|folder|embed|livedoc]`
Required: space key or name. Content type (default: page).
Ask if missing: "Which space?" If the user does not know, run `spaces list` and show the result.
Tool: `listConfluenceContent`.
Output: list with title, last modified, URL. If the list is long, show the first 20 and offer more.

## spaces

Usage: `/confluence spaces <list|get|mine|create|instructions> [key]`

| Action | Tool | Inputs |
|---|---|---|
| `list` | `listConfluenceSpaces` | none |
| `get <key>` | `getConfluenceSpace` | space key or ID |
| `mine` | `getConfluencePersonalSpace` | none |
| `create` | `createConfluenceSpace` | name, key |
| `instructions get <key>` | `getConfluenceSpaceInstructions` | space key |
| `instructions set <key>` | `setConfluenceSpaceInstructions` | space key, instruction text |

Ask if missing: for `create`, ask for the name, then the key (suggest one from the name). For `instructions set`, ask for the text.
Confirm before `create` and `instructions set`.
Output: for `create`, return the space ID, key, and homepage ID.

## create

Usage: `/confluence create [type] [title] [--space KEY] [--parent TARGET] [--template NAME]`
Types: page (default), blog, livedoc, whiteboard, database, embed, smartlink, folder.
Tool: `createConfluenceContent` (Primary).
Ask, in this order, only for what is missing:
1. Type: "What do you want to create?" Choices: page, blog post, live doc, whiteboard, other.
2. Space: "Which space?" If the user does not know, run `spaces list`.
3. Title.
4. Content: "Do you have the content, or do you want me to draft it?" If the user wants a draft, ask for the topic and the audience.
5. Parent: ask only if the user mentions a hierarchy. Default is the space root.
For `embed` and `smartlink`, also ask for the URL.
Steps:
1. Read space instructions (`getConfluenceSpaceInstructions`).
2. If the user names a template, use `listConfluenceTemplates` and `getConfluenceTemplate`.
3. Read the format guide (`getContentFormatGuide` through `executeRead`).
4. Draft the content. Show a short preview with title, space, parent, and the outline.
5. After the user agrees, create the content.
Output: title, ID, URL. Offer: "Do you want to add labels?"

## update

Usage: `/confluence update <target> [instruction] [--status STATE] [--mode live|page]`
Required: target, and the change.
Ask if missing: "What do you want to change?"
Tools: `getConfluenceContent`, `updateConfluenceContent` (Primary). For `--status`: `setConfluenceContentStatus`. For `--mode`: `convertConfluenceContentMode`.
Steps:
1. Read the current content.
2. Read space instructions and the format guide.
3. Use granular edits for small changes. Use full-body replacement only when most of the page changes.
4. Show a before and after of the changed part. Wait for the user to agree.
5. Apply the change.
Output: title, URL, new version number.
Warning: if the page changed after you read it, read it again and show the new difference before you apply.

## comment

Usage: `/confluence comment <list|get|add|reply|edit|resolve|reopen|react> <target> [options]`

| Action | Tool | Inputs |
|---|---|---|
| `list` | `listConfluenceComments` | page or blog target |
| `get` | `getConfluenceComment` | comment ID |
| `add` | `createConfluenceComment` | target, type (footer or inline), text |
| `reply` | `createConfluenceComment` | parent comment ID, text |
| `edit` | `updateConfluenceComment` | comment ID, new text |
| `resolve` / `reopen` | `updateConfluenceCommentResolution` | comment ID |
| `react` | `addConfluenceReaction` (view: `getConfluenceReactions`) | target or comment ID, emoji |

Ask if missing: for `add`, ask "Footer comment or inline comment?" For an inline comment, ask for the exact text on the page to anchor to.
Confirm the comment text before you post it.
Output: comment ID and a link to the page.

## history

Usage: `/confluence history <list|view|diff|restore> <target> [versions]`

| Action | Tool | Inputs |
|---|---|---|
| `list` | `listConfluenceContentVersions` | target |
| `view` | `getConfluenceContentVersion` | target, version number |
| `diff` | `diffConfluenceContentVersions` | target, two version numbers |
| `restore` | `restoreConfluenceContentVersion` | target, version number |

Ask if missing: for `view`, ask for the version number (show the `list` result). For `diff`, default to the latest version against the one before it. Tell the user the default.
Confirm before `restore`. Say: "This creates a new latest version from version N."
Output for `diff`: a short summary of what changed, then the key lines.

## attach

Usage: `/confluence attach <list|get|download|upload> <target> [file]`

| Action | Tool | Inputs |
|---|---|---|
| `list` | `listConfluenceAttachments` | content target |
| `get` | `getConfluenceAttachment` | attachment ID |
| `download` | `downloadConfluenceAttachment` | attachment ID |
| `upload` | `createConfluenceAttachment` | content target, file path |

`download` and `upload` return a `curl` command.
- If a shell with network access is available, run the command.
- If the network is off, give the command to the user and say that they must run it.
Ask if missing: for `upload`, ask for the file path. For `download`, show the `list` result and ask which attachment.

## export

Usage: `/confluence export <target> [pdf|word]`
Required: target, format.
Ask if missing: "Which format: PDF or Word?"
Tool: `exportConfluenceContent`.
Output: the download link.

## move

Usage: `/confluence move <target> --to <parent> [--before|--after <sibling>]`
Required: target, new parent or sibling position.
Ask if missing: "Where do you want to move it? Give the new parent page or space."
Tool: `moveConfluenceContent`.
Confirm with: "Move <title> from <old parent> to <new parent>?"
Output: new location URL.

## copy

Usage: `/confluence copy <target> --to <parent> [--title NEW]`
Required: target, new parent.
Ask if missing: "Where do you want the copy?"
Tool: `copyConfluenceContent`.
Output: new page title and URL.

## archive

Usage: `/confluence archive <target>`
Tool: `archiveConfluenceContent`. It archives one content item only.
Always confirm with the exact title and URL.
Output: confirmation. Say that `unarchive` can restore it.

## unarchive

Usage: `/confluence unarchive <target>`
Tool: `unarchiveConfluenceContent`.
Output: restored title and URL.

## access

Usage: `/confluence access <show|add|remove|replace|restrict|public> <target> [options]`

| Action | Tools | Inputs |
|---|---|---|
| `show` | `getConfluenceContentPermissions`, `getConfluenceContentRestrictionState`, `getConfluencePublicLinkStatus` | target |
| `add` | `addConfluenceContentPermissions` | target, principal, operation |
| `remove` | `removeConfluenceContentPermissions` | target, principal, optional operation |
| `replace` | `replaceConfluenceContentPermissions` | target, full set of grants |
| `restrict` | `setConfluenceContentRestrictionState` | target, state (`OPEN`, `EDIT_RESTRICTED`, or fully restricted) |
| `public enable` | `enableConfluencePublicLink` | target |
| `public disable` | `disableConfluencePublicLink` | target |

Rules:
- Always run `show` first. Show the current state before any change.
- `add` keeps existing grants for other principals.
- `replace` removes all grants that are not in the new set. List what will be removed. Ask for confirmation.
- `public enable` lets anyone with the link view the page without sign-in. State this. Ask for confirmation.
- A principal is a user or group. If you need a user account ID, use `discover` to find a user-lookup tool. If none exists, ask the user for the account ID.
Ask if missing: "Which user or group?" then "Which operation: view or edit?"

## tasks

Usage: `/confluence tasks <list|get|done|reopen> [target|task ID] [--status complete|incomplete]`

| Action | Tool |
|---|---|
| `list` | `listConfluenceTasks` (scope to a page or blog; filter by status) |
| `get` | `getConfluenceTask` |
| `done` | `completeConfluenceTask` |
| `reopen` | `reopenConfluenceTask` |

Ask if missing: for `done` and `reopen`, show the `list` result and ask which task.
Output: task text, status, page URL.

## labels

Usage: `/confluence labels add <target> <label> [label ...]`
Tool: `addLabelsToConfluenceContent`.
Ask if missing: "Which labels?"
To find content by label, use `search --label NAME`.
Output: the labels now on the item.

## follow

Usage: `/confluence follow <star|unstar|watch|unwatch> <page|space|label> <target>`

| Object | Star | Unstar | Watch | Unwatch |
|---|---|---|---|---|
| page | `starConfluenceContent` | `unstarConfluenceContent` | `watchConfluenceContent` | `unwatchConfluenceContent` |
| space | `starConfluenceSpace` | `unstarConfluenceSpace` | `watchConfluenceSpace` | `unwatchConfluenceSpace` |
| label | not supported | not supported | `watchConfluenceLabel` | `unwatchConfluenceLabel` |

Ask if missing: "Do you want to follow a page, a space, or a label?"
Output: one-line confirmation.

## templates

Usage: `/confluence templates <list|get> [name] [--space KEY]`
Tools: `listConfluenceTemplates`, `getConfluenceTemplate`.
Output: template names and types. For `get`, show the template body.
Offer: "Do you want to create a page from this template?" If yes, run `create --template`.

## visual

Usage: `/confluence visual <infographic|app> <create|edit|get> <target>`

| Action | Tool |
|---|---|
| `infographic create` | `createConfluenceInfographicForPage` |
| `infographic edit` | `editConfluenceInfographicForPage` |
| `app create` | `createConfluenceMauiApp` |
| `app edit` | `editConfluenceMauiApp` |
| `app get` | `getConfluenceMauiApp` |

Ask if missing: "Do you want an infographic or an interactive visualization?" Then ask what it must show or what to change.
These tools change the page. Confirm before you run them.
Output: page URL and a short description of the result.
