---
name: create-skill
description: >-
  Generates a new Agent Skill (SKILL.md and optional supporting files) from a workflow description, prompt, or codebase pattern. Use when the user asks to create, scaffold, draft, or convert a workflow into a skill.
metadata:
  version: "1.0.0"
  author: ""
---

# create-skill

Scaffolds and writes `SKILL.md` files and on-demand supporting files using `assets/template.md`.

## Setup

- Write each skill to `<skills-dir>/<skill-name>/SKILL.md`.
- Use only `#` and `##` headings, `-` and `1.` lists, code fences, and inline backticks.
- State every rule once in its designated section without repeating instructions across sections.

## Input

- Workflow description: what the skill does, when it triggers, and how it runs (required).
- Skill name: kebab-case identifier (optional, derived from workflow if omitted).
- Reference artifacts: existing prompts, scripts, or docs to convert (optional).
- Destination path: directory where the skill folder will be written (optional).

## Resources

- Read `assets/template.md` before drafting `SKILL.md` to load the required frontmatter constraints, section order, and section guidelines.

## Steps

1. Analyze the user request to identify the target skill's triggers, inputs, steps, and outputs.
   - WHEN the request lacks a clear trigger, input, or output, ask up to 3 clarifying questions, OR proceed to step 2.
2. Read `assets/template.md`.
3. Evaluate reference docs, templates, or scripts for extraction into supporting folders.
   - WHEN supporting content exceeds ~100 lines, write those files to `resources/`, `assets/`, or `scripts/` subdirectories AND document when to load them under `## Resources` in `SKILL.md`.
4. Draft `SKILL.md` by following the inline instructions in `assets/template.md` and removing all YAML/HTML comments and unused optional sections.
5. Output the finished skill.
   - WHEN a destination path is known, write the files to disk, OR output the Markdown directly in chat.

## Gotchas

- WHEN writing `description`, address the agent router by stating inputs, outputs, and triggers (DO NOT write human-facing marketing copy).
- WHEN formatting the Markdown body, use plain text and uppercase keywords like `WHEN`, `AND`, `OR`, `DO NOT` (DO NOT add bold `**`, italics `*`, emojis, or horizontal rules `---`).
- WHEN finalizing `SKILL.md`, strip all `#` YAML comments and `<!-- -->` HTML comments from the template (DO NOT leave template comments or empty headings in the generated file).

## Output

- `<skill-name>/SKILL.md`
- Supporting files in `resources/`, `assets/`, or `scripts/` when applicable.

## Outro

Verify that the generated `SKILL.md` matches the structure in `assets/template.md` with all comments and decorative formatting removed. Report the created file paths (or display the Markdown) and stop without executing the new skill.
