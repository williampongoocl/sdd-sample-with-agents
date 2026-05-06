---
applyTo: "**"
---

# Agent roster — Gen-e2 Workshop (Entry Module)

This workshop uses a deliberately simple **1-tier hub-and-spoke** of 5 agents. Advanced 2-tier patterns (lead agents, nested dispatch, specialist brainstorm sub-agents) exist in production Gen-e2 but are out of scope for the entry module — participants learn the fundamentals here and graduate to the advanced patterns later.

## The 5 agents

| Agent | Role | When to use |
| ----- | ---- | ----------- |
| `orchestrator` | Entry point. Coordinates the workflow; dispatches specialists; talks to the participant. | Always. Participants invoke `@orchestrator` in Copilot Chat to start anything. |
| `researcher` | Gathers context from a single scope (codebase, docs, web). | Dispatched by the orchestrator during Phase 1 (GATHER). Multiple instances can run in parallel, one per scope. |
| `planner` | Converts research findings into a structured plan with phases, tasks, and acceptance criteria. | Dispatched by the orchestrator during Phase 2 (PLAN), after research is consolidated. |
| `implementer` | Executes one stream of an approved plan. Writes and edits files. | Dispatched by the orchestrator during Phase 3 (EXECUTE), after the participant approves the plan. |
| `reviewer` | Quality gate. Verifies implemented work against the plan's acceptance criteria. | Dispatched by the orchestrator during Phase 4 (REVIEW), after implementation completes. |

## Dispatch rules

- **Only the orchestrator is user-invocable.** Participants type `@orchestrator` in Copilot Chat to start. The other four agents (`researcher`, `planner`, `implementer`, `reviewer`) have `user-invocable: false` in their frontmatter and are dispatched only by the orchestrator.
- **Specialists do not dispatch each other.** The orchestrator is the only hub — it dispatches each specialist, collects the result, and returns to the participant between phases.
- **Parallel researchers are allowed.** The orchestrator may dispatch multiple `researcher` instances in parallel, each scoped to a different area (harness config, docs, source code). All other dispatch is single-instance.
- **No 2-tier.** There are no lead agents and no `*-in-teams` prompts in this workshop. If a phase needs more structure than one specialist can handle, the orchestrator breaks it into sequential specialist calls — not nested dispatch.

## Skills, not sub-agents

Cross-cutting expertise in this workshop lives as **skills** under `.github/skills/`, not as sub-agents:

- **`brainstorm`** — three-phase divergent thinking (diverge → vote → synthesize). Executed **inline by the orchestrator** when a decision needs perspective diversity. No `brainstorm-*` sub-agents exist in this workshop.
- **`memory-improvement`** — a reactive check the orchestrator runs at the end of GATHER/PLAN to surface proposals for improving the harness or instructions based on what was learned during the task.

## Model

All five agents are pinned to `Claude Sonnet 4.6` via frontmatter. If a participant's Copilot plan does not expose Sonnet 4.6, the model picker fallback applies — see `docs/setup/module-2-vscode.md` for plan and admin-policy notes.

## Further reading

- `docs/setup/module-4-1-harness.md` — full walkthrough of this harness, including prompts, skills, hooks, and a first-invocation checklist.
