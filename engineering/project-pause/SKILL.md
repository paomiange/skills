---
name: project-pause
description: Save or reload project handoff context in docs/project-history/latest-session.md. Run only when explicitly asked to use project-pause; discussing or editing this skill does not invoke its workflow.
---

# Project Pause

Maintain one handoff file: `docs/project-history/latest-session.md`. This is a context summary, not a backup of code or a way to restart services.

## Select the Action

- If the user specifies `pause` or `reload`, proceed directly.
- Otherwise ask whether to pause, reload, or cancel before inspecting the workspace or history. Accept `1` / `pause`, `2` / `reload`, or `3` / cancel; clarify an ambiguous answer without starting the workflow.
- If the user cancels, stop without inspecting or changing the workspace.

## Pause

1. Read the existing handoff if present. Carry forward still-relevant constraints, decisions, unfinished work, and blockers; reconcile them with the current session instead of replacing them with only this session's activity.
2. Check current state with `git status --short --branch` and `git rev-parse HEAD`. Review relevant session changes and actual validation outcomes. If Git is unavailable or this is not a Git repository, record that limitation and use available file/session evidence; do not invent repository state.
3. Create the history directory if needed and update only `latest-session.md` using the structure below. Leave other history files untouched.

```markdown
# Latest Project Session

Updated: YYYY-MM-DD HH:MM TZ

## Goal and Constraints

User goal, active constraints, and decisions that affect remaining work.

## Progress and Validation

Completed work, relevant files, branch and HEAD, uncommitted changes, and actual validation commands and outcomes. Distinguish known session changes from unrelated or uncertain changes.

## Open Work and Blockers

Unfinished tasks, unresolved questions, and dependencies; use None if empty.

## Next Steps and Resume Context

Prioritized next actions and the context needed to perform them. Include relevant service state or setup details when needed. Distinguish user-authorized work from suggested follow-ups.
```

Keep the handoff concise, usually under 100 lines. Record only supported facts; label unknown or unverified state. Do not claim successful tests, commits, pushes, or deployments without evidence.

## Reload

1. Read `docs/project-history/latest-session.md`. If it is missing or unusable, report that and ask what to work on; do not create a file or search for substitute history.
2. Check the current branch, HEAD, and working tree with the same Git commands used for pause. If Git is unavailable, report the limitation. Compare available evidence with the handoff, report differences, and inspect only the files or validation results needed for the next action. A clean working tree alone does not establish that the recorded state is current.
3. Briefly summarize the goal, progress, outstanding work, and material state differences. Treat the handoff as historical context, not fresh authorization or proof that services and tests remain current.
4. Follow the user's intent:
   - If they requested only context loading, report the restored context and stop.
   - If they requested continued work or a concrete follow-up and the next action is clear and authorized, proceed.
   - If intent or the next action is unclear, ask a focused question before making changes. Suggested next steps in the handoff do not independently authorize execution.
