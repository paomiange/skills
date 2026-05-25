---
name: project-pause
description: Pause or reload a coding/project session using docs/project-history as the handoff memory. Use only when the user explicitly invokes the project-pause skill, such as with $project-pause or an unambiguous command to use project-pause; do not trigger for general pause, reload, wrap-up, save-progress, history, memory, or handoff requests that do not name project-pause.
---

# Project Pause

## Overview

Manage project session handoff memory in `docs/project-history`. Use `pause` to record the latest session state for a future agent, and use `reload` to resume project progress/development from the recorded history.

## Mode Selection

Before inspecting the workspace, reading history, or writing any files, ask the user exactly:

```text
pause、reload 还是取消不执行？
```

Proceed only after the user chooses one of these modes:

- `pause`: Run the pause workflow and update `docs/project-history`.
- `reload`: Run the reload workflow and continue project progress/development from `docs/project-history`.
- `取消不执行`: Stop immediately. Do not read history, inspect the workspace, or update files.

If the reply is ambiguous or not one of these options, ask the same question again once. If it remains unclear, stop without changing files.

## Pause Workflow

1. Inspect the current workspace state before writing the handoff:
   - Run `git status --short --branch`.
   - Review recent edits, relevant open tasks, test results, and any blockers from the session.
   - If the session involved commands whose output matters, include the command names and outcomes.

2. Ensure the history location exists:
   - Use `docs/project-history/`.
   - If the directory does not exist, create it.

3. Replace stale history with the latest handoff:
   - Write the current record to `docs/project-history/latest-session.md`.
   - Treat previous session records in `docs/project-history` as obsolete.
   - Do not keep a growing archive of detailed session logs. If old session-log files exist in this directory, remove or replace them only when they are clearly generated project-history records; otherwise leave unrelated files untouched and note the ambiguity in `latest-session.md`.

4. Use this structure for `latest-session.md`:

```markdown
# Latest Project Session

Status: current
Updated: YYYY-MM-DD HH:MM TZ

## Goal

Briefly state what the user wanted in this session.

## Completed

List concrete changes made, files touched, commands run, and decisions reached.

## Current State

Summarize repository status, uncommitted files, running services, branch/remotes, and test or validation state.

## Next Steps

List the most likely next actions in priority order.

## Blockers

Record unresolved questions, failed commands, missing approvals, or external dependencies. Write `None` if there are no known blockers.

## Resume Notes

Give the next agent concise context needed to resume without rereading the whole conversation.
```

## Writing Guidelines

- Be factual and specific. Prefer paths, branch names, command outcomes, and concrete remaining work over general narration.
- Keep the record short enough to scan quickly, usually under 100 lines.
- Do not claim tests, builds, commits, pushes, or deployments were completed unless they actually ran successfully.
- If the workspace has uncommitted changes, mention whether they are expected and whether they appear related to the session.
- Do not overwrite unrelated project documentation outside `docs/project-history/`.

## Reload Workflow

Use reload when the user wants to continue project progress or development and the historical work record is in `docs/project-history`.

1. Check for `docs/project-history/`.
   - If it does not exist, tell the user no project history was found and ask for the next instruction.
   - Do not create the directory during reload.

2. Read the latest handoff record:
   - Prefer `docs/project-history/latest-session.md`.
   - If that file does not exist, inspect `docs/project-history/` for the most relevant project-history record and state which file is being used.
   - If no usable record exists, tell the user no usable project history was found and ask for the next instruction.

3. Restore working context:
   - Summarize the recorded goal, completed work, current state, next steps, blockers, and resume notes.
   - Run `git status --short --branch` to compare the current workspace with the recorded state.
   - Identify the most likely next development action from the history.

4. Continue the project:
   - Ask the user for confirmation before making new code or file changes if the next action is not explicit.
   - If the user already provided a concrete follow-up task with the reload request, proceed with that task after restoring context.
