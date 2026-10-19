---
name: working-workflow
description: Use this skill at the START of any kind of work (feature, bug, refactor, investigation, chore, or any task the user asks you to carry out). It defines how the agent must work. Linear is the single source of truth, the task must exist in Linear before anything else, every step is recorded there, and the issue status must always reflect where the work stands. Triggers include any request to start, take, plan, implement, fix, or continue a task.
---

# Working Workflow

This skill defines **how the agent works**. It applies to every piece of work, before any analysis or code.

## Core rules

1. **Linear is the entry and exit point.** Work is taken from Linear or registered in Linear. The issue must exist **before** anything else happens (before analyzing, planning, or coding). If the user asks for something that has no issue, create it first (status `Backlog` or `Todo`), then continue.
2. **The status always tells the truth.** Move the issue as soon as reality changes, never later "in batch". Someone reading Linear must know exactly where the work stands.
3. **EVERYTHING is recorded in Linear**: analysis, questions, the user's answers, decisions, deliverables, test results, failure reasons. If it was said in chat and it matters, it goes in the issue as a comment or in the description.
4. **Use the exact status names below.** Some contain typos in the Linear workspace (`Redy for testing`, `Tedted`). Always call `list_issue_statuses` for the team (currently `Guido Builds`, key `GUI`) if in doubt, and use the name exactly as Linear returns it.

## Statuses

| Status | Type | Meaning |
|---|---|---|
| `Backlog` | backlog | Registered, no priority. Kept for later. |
| `Todo` | unstarted | Next candidate: the one to take when starting something new. |
| `In specification` | started | Being analyzed and clarified (iterative loop with the user). |
| `Ready for development` | started | Spec approved, deliverables created as sub-issues. Ready to implement. |
| `In Progress` | started | Actively being worked on. |
| `Develop` | started | Already developed (implementation finished). |
| `Under Review` | started | Developed; checks are being run (lint, types, tests, self-review, code review). |
| `Redy for testing` | started | Waiting for **human** validation. Only if human validation is required. |
| `Tedted` | started | Human validation passed. Only if human validation is required. |
| `Failed` | started | Testing failed (human or automated validation). |
| `Done` | completed | Finished and verified. |
| `Rejected` | canceled | Discarded. Requires a comment with the reason. |
| `Duplicate` | duplicate | Already covered by another issue. Link it and say which one. |

## Flow

```
Backlog → Todo → In specification → Ready for development → In Progress → Develop → Under Review ─┬→ Done
                                                                    ▲                              │
                                                                    │                   (human validation required)
                                                                    │                              ▼
                                                                    │                      Redy for testing → Tedted → Done
                                                                    │                              │
                                                                    └──────────── Failed ◄─────────┘

Rejected / Duplicate: allowed from any status.
```

## Phase 1 — Take or register the task

- If the user gives an issue ID/URL: read it (`get_issue`, plus `list_comments`) and respect its current status.
- If there is no issue: search first (`list_issues`) to avoid duplicates, then create it with `save_issue` (team, title, description with context, status `Backlog` or `Todo` as the user indicates).
- Never start work on an issue in `Backlog`. Ask the user to prioritize it, or move it to `Todo` with their consent.
- Parent/child: if the task is a sub-task, link it to its parent.

## Phase 2 — `In specification` (iterative loop)

Move the issue to `In specification`, then loop:

**Analyze → Understand → Ask → Save answers in Linear → repeat**

- Analyze the codebase and the request to understand the real scope.
- Ask the user every open question (batch them, don't drip).
- After each answer, write it to the issue (comment or description update), including decisions and discarded options.
- Keep the description current: goal, scope, out of scope, acceptance criteria, technical approach, risks, and whether **human validation** is needed.
- Stay here until nothing ambiguous remains.

### Exit: create the deliverables

Before leaving this phase, **every deliverable / sub-item must exist as its own issue linked to the parent** (`parentId`), ALWAYS. Each sub-issue gets a title, a description, and acceptance criteria. Work is never done against an unregistered deliverable. If new deliverables appear later, create them in Linear first, then work on them.

Present the final spec and the list of sub-issues to the user. **Ask for explicit approval** before moving to `Ready for development`. Do not move it on your own.

## Phase 3 — `Ready for development` → `In Progress`

- Only issues in `Ready for development` can be started. Verify all sub-issues exist and are linked to the parent.
- Move the parent to `In Progress` when starting. Move each sub-issue to `In Progress` when you start it, and keep the parent's status coherent.
- Comment on meaningful progress, decisions made while coding, and any deviation from the spec. If the spec turns out to be wrong or incomplete, go back to `In specification`, don't improvise silently.

## Phase 4 — `Develop` → `Under Review`

- `Develop`: implementation finished.
- `Under Review`: run the checks (tests, lint, type-check, build, self-review of the diff). Record results in a comment (what was run, outcome). Fix what fails before moving on.

## Phase 5 — Validation

- **No human validation needed** → when checks pass and acceptance criteria are met, move to `Done`.
- **Human validation needed** → move to `Redy for testing`, and comment with clear steps to test and what to expect. Then wait.
  - Passes → `Tedted` → `Done`.
  - Fails → `Failed`. Record why (what was tested, what happened vs expected). Then return to `In Progress` to fix and repeat the flow.
- If a check in `Under Review` fails and cannot be fixed, use `Failed` with the reason, not `Done`.

## Who decides what

| Transition | Who |
|---|---|
| Create issue, `Backlog`/`Todo` ↔ | Agent creates; user prioritizes what goes to `Todo`. |
| `Todo` → `In specification` | Agent, when taking the task. |
| `In specification` → `Ready for development` | **User approves explicitly.** |
| `Ready for development` → `In Progress` → `Develop` → `Under Review` | Agent, automatically as work happens. |
| `Under Review` → `Done` (no human validation) | Agent, only with checks passing and criteria met. |
| `Under Review` → `Redy for testing` | Agent. |
| `Redy for testing` → `Tedted` / `Failed` | **Human result.** The agent only records it. |
| `Rejected` / `Duplicate` | Agent proposes; user confirms. |

## Closing a work session

- Leave the issue in the status that reflects reality, never in a stale one.
- Add a final comment: what was done, what's left, and the next step.
- Do not mark `Done` anything that has pending sub-issues, unverified criteria, or failed checks.

## Don'ts

- Do not start analysis or code before the issue exists in Linear.
- Do not keep decisions only in chat.
- Do not work on a deliverable that has no sub-issue.
- Do not skip statuses to look faster, and do not leave a status stale.
- Do not move to `Ready for development` without the user's explicit approval.
