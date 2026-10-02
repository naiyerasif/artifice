# Rally item templates

Use these as the draft format for `create` and `breakdown`. Fill each field from user input and source context. Remove a field if no real content exists. Do not fill with generic text.

## Contents
- Story
- Feature
- Defect
- Task
- Test case
- Acceptance criteria rules
- Sizing

## Story

```
Name: <short, action-focused>
Description: As a <role>, I want <capability> so that <benefit>.
Acceptance Criteria:
- Given <context>, when <action>, then <result>
- ...
Plan Estimate: <points>
Parent: <Feature ID or none>
Iteration: <name or none>
Owner: <user or none>
```

## Feature

```
Name: <outcome-focused>
Description: <problem, users, intended outcome>
Acceptance Criteria / Success Measure:
- <measurable result>
Parent Initiative: <ID or none>
Release / PI: <name or none>
Owner: <user or none>
Planned Dates: <start and end or none>
```

## Defect

```
Name: <symptom in one line>
Description:
  Steps to reproduce:
  1. ...
  Expected result: ...
  Actual result: ...
  Environment: <build, browser, OS, or service>
Severity: <value>
Priority: <value>
Affected Story / Feature: <ID or none>
Owner: <user or none>
```

## Task

```
Name: <verb + object>
Description: <one or two lines>
Estimate (hours): <number>
Owner: <user or none>
Parent: <Story or Defect ID>
```

## Test case

```
Name: <criterion in short form>
Steps:
1. ...
Expected Result: ...
Linked Story: <ID>
Folder: <name>
```

## Acceptance criteria rules

- Each criterion has one observable result.
- Each criterion can pass or fail in a test.
- Use specific values, not vague words. Avoid "fast", "easy", "properly", "appropriate".
- Cover the main path, the main error path, and any stated limit.
- Keep one criterion per line.

## Sizing

Use the point scale that existing stories in the project use. If unsure, fetch 3 to 5 recent accepted stories to see the scale. If the team has no scale, ask the user. Do not estimate a feature in story points unless the team does so.
