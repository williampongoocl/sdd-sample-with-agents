---
name: memory-improvement
description: "Proactively detect opportunities to improve project instructions, conventions, or harness artefacts when friction or misunderstanding surfaces during a task."
---

# Memory improvement skill

## Purpose

This is a **reactive** skill. The orchestrator runs it at the end of a phase — typically after GATHER/PLAN and again after REVIEW — to surface proposals for improving the harness, instructions, or conventions based on what was learned during the task.

It is not a rewrite. It is a surfacing mechanism. Every proposal requires **explicit participant approval** before anything is changed.

## When this skill applies

Trigger a memory-improvement check when any of these signals appeared during the phase:

- **User-frustration signal** — the participant had to correct the agent more than once on the same thing, or rephrased the same request.
- **Workflow friction** — a step felt unusually hard or repetitive, or the agent had to improvise because a convention was missing or unclear.
- **Improvement opportunity** — during research, the orchestrator noticed a pattern that the project uses but nobody documented; or a small rule that would have prevented a misstep.
- **Convention drift** — the implementer discovered the codebase's stated conventions and actual conventions disagreed.

If none of these signals appeared, skip this skill entirely and return "no proposals" to the orchestrator.

## Proposal format

For each proposal, present the participant with:

1. **Observation** — what happened, in one sentence. Cite the specific moment or file.
2. **Proposed change** — the exact file to edit and a concrete diff (or the exact new file to create).
3. **Rationale** — one sentence on what this prevents or improves.
4. **Confidence** — one of `high`, `medium`, `low`. `high` = saw the same signal at least twice this task. `medium` = strong but single signal. `low` = speculative — flagging for visibility only.

## Rules

- **Never automatically apply a proposal.** Every change is presented to the participant for explicit approval.
- **Keep proposals surgical.** Prefer a one-line addition to a `.github/instructions/` file over a rewrite of the agent set.
- **Batch proposals.** Present all proposals for a phase together, not drip-fed.
- **Prefer the smallest scope.** If an observation could live in a task-specific note instead of a global instruction, prefer the note.
- **Demote uncertainty.** If confidence is `low`, say so explicitly — do not inflate.

## What this skill is NOT

- It is **not** a chance to rewrite the plan. Plan decisions belong to `@planner`.
- It is **not** a code review. AC verification belongs to `@reviewer`.
- It is **not** a backlog. Proposals the participant does not approve are discarded — raise them again only if the underlying signal reappears on a future task.

## Example proposal

> **Observation** — During GATHER, the researcher had to re-read `.github/instructions/agents.instructions.md` twice because the dispatch rules were buried at the bottom.
>
> **Proposed change** — In `.github/instructions/agents.instructions.md`, move the "Dispatch rules" section above "The 5 agents" table.
>
> **Rationale** — Dispatch rules are consulted more often than the agent-descriptions table during a task. Leading with them saves a re-read per task.
>
> **Confidence** — medium (single signal this task, but structural change — low downside).
