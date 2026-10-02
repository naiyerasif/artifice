# Rally subcommands

Each section gives: syntax, required inputs, follow-up questions, steps, and output.

## Contents
- Shared follow-up pattern
- setup
- fetch
- find
- mine
- standup
- create
- update
- comment
- link
- breakdown
- audit
- deps
- report
- handoff
- workspace
- help

## Shared follow-up pattern

When an input is missing:
1. State what is missing in one line.
2. Ask one question. Give 2 to 4 options when possible.
3. Use the answer. Continue. Do not re-ask for values the user already gave.

Common inputs and how to resolve them:
- **Project:** use the default project. Ask only if the user has none or the task spans projects.
- **Iteration:** "current" means the iteration that contains today. Ask if several match.
- **Release:** ask for the release name if the user does not give one. Offer the current releases found.
- **Owner:** "me" means the authenticated user. Ask if the user does not say.
- **Parent:** ask for the parent ID when a child type needs one.

## setup

`/rally setup [--verify]`

Run the setup flow. With `--verify`, run only the Verify section.

## fetch

`/rally fetch <id> [--children] [--discussion] [--history] [--attachments]`

Required: an ID. If missing, ask: "Which Rally ID?"

Steps:
1. Resolve the type from the prefix. Fetch the item.
2. Always include: ID, name, type, state, owner, project, iteration or release, plan estimate, to-do or actuals if present, blocked flag and reason, parent, description, acceptance criteria, notes.
3. Add children, discussion, revision history, or attachments only if the flag is set. If the user asks for "full context", set all flags.
4. Attachments: read text-like attachments (md, txt) for context. Name other attachments and skip them.

Output: a short header line, then fields in a compact list. Then a 2 to 3 line summary of what the item needs. Do not repeat the raw description.

## find

`/rally find <text or filters>`

Examples: `/rally find login timeout`, `/rally find defects open severity=Critical`, `/rally find stories iteration=current state=Defined`

Required: a search text or at least one filter. If missing, ask what to search for.

Steps:
1. Parse type, state, owner, project, iteration, release, and free text.
2. Query through workflow or API discovery.
3. Limit to 25 rows. If more match, say so and offer to narrow.

Output: table with ID, Name, State, Owner, Iteration or Release, Estimate.

## mine

`/rally mine [--in-progress] [--all]`

Show items owned by the authenticated user. Default: current iteration, not accepted. `--in-progress` shows only In-Progress. `--all` removes the iteration limit.

Output: table grouped by type (Stories, Defects, Tasks). Show blocked items first.

## standup

`/rally standup [--team <project>]`

Without `--team`, build a personal standup. With `--team`, build a team snapshot.

Steps:
1. Fetch in-progress items for the user (or the team iteration), with revision history and discussion for the last 2 days.
2. Find blocked items and their blocked reasons.

Output, personal:
- Done since last standup
- Working on
- Blocked (with reason)

Keep each point to one line, ready to say aloud.

Output, team: blocked items with reasons and recent discussion, recently completed items, stale items (no owner, or no update in 5+ days).

## create

`/rally create --story | --feature | --defect | --tasks | --testcase`

If no type flag is given, ask which type. Use the item templates.

General flow for all types:
1. Collect required fields. Ask follow-ups for anything missing.
2. Show the draft.
3. Ask for confirmation.
4. Create. Verify by fetching the new item.

### create --story

Required: name, project. Ask for: description (as "As a ... I want ... so that ..."), acceptance criteria, plan estimate, parent feature (optional), iteration (optional), owner (optional).

Ask in this order, one at a time, skipping known values:
1. What should the story do? (If the user already gave a sentence, derive the name and description from it.)
2. Acceptance criteria. Offer to draft them from the description.
3. Parent feature ID, or none.
4. Estimate, iteration, owner. Ask together as one question with options "set now" or "leave empty".

### create --feature

Required: name, project. Ask for: description, acceptance criteria or success measure, parent initiative (optional), release or PI (optional), owner (optional), planned dates (optional).

Features are portfolio items. Find the feature type through API discovery. Do not assume a type path.

### create --defect

Required: name, description with steps to reproduce, expected result, actual result. Ask for: severity, priority, environment, affected story or feature (optional), owner (optional), found-in build (optional).

If the user pastes an error or log, extract steps and actual result from it. Ask only for what remains.

### create --tasks

Required: parent story or defect ID. If missing, ask: "Which story or defect gets the tasks?"

Steps:
1. Fetch the parent. Read description, acceptance criteria, and existing child tasks.
2. If the user gave task names, use them. If not, propose 3 to 6 tasks that cover the acceptance criteria. Do not duplicate existing tasks.
3. Show a table: Name, Description (one line), Estimate hours, Owner.
4. Ask: "Create these tasks? Assign to you?" Ask for any missing owner or estimate choice here.
5. Create all tasks under the parent. Verify by fetching the parent with children.

Output: the task checklist with IDs.

### create --testcase

Required: story ID. Optional: `--folder <name>`.

Steps:
1. Fetch the story. Read acceptance criteria.
2. Ask for the test folder name if not given. Create the folder in the same project (find it first; reuse it if it exists).
3. Propose one test case per acceptance criterion with name, steps, and expected result.
4. After confirmation, create each test case, link it to the story, and place it in the folder.
5. If acceptance criteria are missing or vague, say so. Offer `/rally audit <id>` first.

## update

`/rally update <id> [--state X] [--owner X] [--estimate N] [--iteration X] [--blocked "reason"] [--field name=value]`

Required: an ID and at least one change. If the ID is missing, ask for it. If no change is given, ask what to change.

Steps:
1. Fetch the item. Show the current values of the fields to change.
2. Show a before and after table. Ask for confirmation.
3. Update. Fetch again. Show the new values.

Valid states depend on the item type and the workspace config. If the user gives a state that is not valid, show the valid values and ask.

## comment

`/rally comment <id> "<text>"`

Required: ID and text. Ask for either if missing.

Post the text as a discussion post. Show the text to the user before posting if the text was written by you. If the user wrote the exact text, post it without a second confirmation.

## link

`/rally link <id> --pr <url>`

Required: ID and PR URL. If the PR URL is missing, ask. If working in a git repository with the GitHub CLI, offer to use the current branch PR.

Steps:
1. Fetch the item.
2. Add the PR link through the supported field or the discussion, whichever the Rally API supports for the item (find with API discovery). Tell the user which one was used.
3. Post a discussion comment: PR title, URL, and "ready for review". Ask the user to approve the text if the user did not write it.

## breakdown

`/rally breakdown <id>`

Supports Initiative to Features, and Feature to Stories. If the item is another type, say it is not supported and offer `create --tasks` for stories and defects.

Steps:
1. Fetch the item with Description, Notes, Acceptance Criteria, Discussion, attachments, and existing children.
2. List gaps first: missing goal, missing acceptance criteria, unclear scope, no estimate basis.
3. Propose children that do not duplicate existing children. Use the child templates.
4. Show the list with Name, one-line description, acceptance criteria bullets, and estimate. Ask which to create: all, selected, or none. Do not create anything before the user answers.
5. Create the chosen items under the parent. Verify.

Ask a follow-up if gaps block a good breakdown. Do not guess.

## audit

`/rally audit <id> [--children]`

Check readiness of a story, a feature, or (with `--children`) all stories under a feature.

Checks per story:
- Has a plan estimate
- Has acceptance criteria that are testable (specific, observable, no vague words such as "fast", "user-friendly", "works properly")
- Has an owner or a clear team
- Has a parent where the team uses parents
- No unresolved blocked flag

Output: table with ID, Name, Result (Ready or Not ready), and Reason. For each vague acceptance criterion, give a better wording. Offer to update the item. Do not update without confirmation.

## deps

`/rally deps <id>`

Analyze predecessors and successors for the item and its children.

Output: table with Item, Depends on or Blocks, Owning team, Target date, Owner. Flag rows with no target date or no owner. End with a risk summary: High, Medium, or Low, and why.

## report

`/rally report --iteration | --release | --portfolio | --defects [scope]`

Always add the label "Custom report from live data. Not an official Rally report. Compare with a built-in Rally report before you rely on it."

Required scope: project for `--iteration`, release name for `--release`, item ID for `--portfolio`, project for `--defects`. Ask if missing.

- `--iteration [project]`: total stories, points committed vs completed, count by state, blocked items, unestimated items. Table format.
- `--release <name>`: total features, percent complete by story points, open defects by severity, features with unresolved dependencies. End with Green, Yellow, or Red and the reason.
- `--portfolio <id>`: for each child initiative and feature: state, percent done by story points, days since last update. Sort by staleness, oldest first.
- `--defects [project] [--iterations N]`: defects by iteration and severity for the last N iterations (default 3). State if the severity mix is getting worse.

Show how each number was computed in one line under the table.

## handoff

`/rally handoff <id> --qa`

Required: story ID. Ask for deployment notes if the user did not give them.

Steps:
1. Fetch the story and linked test cases.
2. Show the plan: set state or "ready for test" flag, post deployment notes, list test cases. Ask for confirmation.
3. Update the story. Post the notes as a discussion comment.
4. List linked test cases with their state. Flag stories with no test cases and offer `create --testcase`.

Use the state or flag that exists in the workspace for "ready for testing". If unclear, show the valid options and ask.

## workspace

`/rally workspace [name]`

Without a name: show the active workspace and the default project. With a name: switch through the supported operation. If switching is not possible, tell the user to change the Default Project in their Rally profile and restart the MCP connection.

## help

`/rally help [subcommand]`

Without an argument: show the subcommand table from SKILL.md. With an argument: show the syntax, required inputs, and one example for that subcommand.
