# SDD Sample With Agents

This repository shows how to run a lightweight Software-Driven Development (SDD) workflow using Copilot customization files.

The workflow starts from a transcript in `docs/transcript/`, turns that into stories and feature/task docs, then executes implementation through an orchestrated multi-agent process.

## SDD Flow In This Repo

1. Capture product discussion in `docs/transcript/transcript.txt`.
2. Generate user stories (vertical slices) from transcript content.
3. Organize work as feature + task docs under `docs/features/`.
4. Research and create an approved implementation plan.
5. Execute plan streams and review acceptance criteria.
6. Repeat for next story/task.

From the current transcript, the agreed direction is:
- Build a healthy routine app
- Start with water tracking first (MVP)
- Default target is 8 glasses/day, customizable
- Optional reminders and timezone/permission considerations

## What The Markdown Files Do

### 1) Agents (`.github/agents/*.agent.md`)

These define specialized roles in a hub-and-spoke workflow.

| File | Role | When Used |
| --- | --- | --- |
| `.github/agents/orchestrator.agent.md` | Entry coordinator | Always; user talks to this agent only |
| `.github/agents/researcher.agent.md` | Context gatherer | GATHER phase |
| `.github/agents/planner.agent.md` | Plan author | PLAN phase |
| `.github/agents/implementer.agent.md` | File editor/executor | EXECUTE phase |
| `.github/agents/reviewer.agent.md` | Quality gate | REVIEW phase |

Key rule: only `orchestrator` is user-invocable. Specialists are dispatched by orchestrator.

### 2) Instructions (`.github/instructions/*.instructions.md`)

These are always-on behavior and quality rules.

| File | Purpose |
| --- | --- |
| `.github/instructions/agents.instructions.md` | Defines the workshop agent roster and dispatch pattern |
| `.github/instructions/copilot-interactions.instructions.md` | Enforces plan-first + explicit approval interaction protocol |
| `.github/instructions/docs.instructions.md` | Rules for structuring and maintaining `docs/` as project knowledge base |
| `.github/instructions/hooks.instructions.md` | Guidance for lifecycle hooks under `.github/hooks/` |
| `.github/instructions/instructions.instructions.md` | Best practices for writing instruction files |
| `.github/instructions/prompts.instructions.md` | Best practices for reusable prompt workflows |
| `.github/instructions/skills.instructions.md` | Best practices for reusable skills |

### 3) Prompts (`.github/prompts/*.prompt.md`)

These are reusable workflows that orchestrator runs.

| File | Purpose | Typical Position In SDD |
| --- | --- | --- |
| `.github/prompts/generate-stories-from-transcript.prompt.md` | Creates INVEST stories from transcript | Discovery -> Backlog |
| `.github/prompts/research-and-plan.prompt.md` | Runs GATHER + PLAN through researcher/planner | Before coding |
| `.github/prompts/implement-and-review.prompt.md` | Runs EXECUTE + REVIEW through implementer/reviewer | During delivery |
| `.github/prompts/generate-implementation-plan.prompt.md` | Produces detailed implementation plan doc | Planning artifact |
| `.github/prompts/execute-implementation-plan.prompt.md` | Executes a specific task/sub-task with status tracking | Focused implementation |

### 4) Skills (`.github/skills/*/SKILL.md`)

Skills provide reusable protocols, usually cross-cutting.

| File | Purpose |
| --- | --- |
| `.github/skills/memory-improvement/SKILL.md` | Proposes instruction/harness improvements when friction appears |

## Step-by-Step: Transcript to Working App

This section is action-oriented. For every step, do the action, check the output, and only then move forward.

### Step 1: Prepare source inputs

User action:
1. Put meeting content in `docs/transcript/transcript.txt`.
2. Confirm templates are available in `docs/features/TEMPLATE/`.

What to check before continuing:
- Transcript includes real product decisions (not placeholders).
- Scope is clear enough to extract user-visible capabilities.

### Step 2: Generate user stories from transcript

User action in Copilot Chat:
1. Run the slash prompt:

```text
/generate-stories-from-transcript
```

2. If Copilot asks for details, provide a direct message like:

```text
Use docs/transcript/transcript.txt as source.
Generate INVEST-compliant vertical slices.
Use consistent roles across all stories.
Exclude deferred items.
```

Expected output:
- Story files generated as separate markdown files (typically under `docs/stories/` per the prompt definition).
- Each story includes acceptance criteria and remains vertically sliced.

Exit criteria:
- You can point to a numbered story list (for example 01, 02, 03) and each story is testable.

### Step 3: Convert stories into feature/task docs

User action:
1. Create a feature folder such as `docs/features/F001-water-tracking/`.
2. Copy files from `docs/features/TEMPLATE/` into that feature folder.
3. Rename `TXXX-task-template.md` into concrete task files (`T001-...`, `T002-...`).
4. Fill each task with technical requirements and acceptance criteria.

Recommended task split for current transcript:
1. `T001-log-water-intake.md`
2. `T002-custom-water-goal.md`
3. `T003-reminder-preferences.md`

Expected output:
- One feature overview plus 2-6 concrete task files.
- Each task is implementable in one stream and maps to user value.

Exit criteria:
- Every story is linked to at least one task file.
- Each task has clear acceptance criteria and file impact.

### Step 4: Run research and create an approved plan

User action in Copilot Chat:
1. Start with orchestrator.
2. Ask it to run research + plan for your target feature/task.

Example:

```text
@orchestrator
Run research-and-plan for F001 water tracking.
Scope first pass to T001-log-water-intake.
Use docs and existing project conventions.
```

What happens:
1. Orchestrator dispatches researcher(s).
2. Planner produces implementation streams and ACs.
3. You may get clarification questions.

Your required action:
- Answer clarifications.
- Explicitly approve the plan when it is correct.

Approval examples:
- "approved"
- "go ahead"
- "proceed"

Expected output:
- A concrete, approved plan with streams, dependencies, acceptance criteria, and review gates.

Exit criteria:
- Plan is explicitly approved and has no unresolved ambiguity.

### Step 5: Execute implementation and run review gates

User action in Copilot Chat:

```text
@orchestrator
Use the approved plan and run implement-and-review.
Complete all streams and keep cycling until all ACs pass.
```

What happens:
1. Implementer executes assigned stream(s).
2. Reviewer validates ACs with evidence.
3. If reviewer fails any AC, orchestrator routes fixes and re-review.

Expected output:
- Implementation complete with reviewer verdict: all ACs satisfied.

Exit criteria:
- All scoped tasks are marked complete and reviewer passes all ACs.

### Step 6: Close, document, and preserve learning

User action:
1. Confirm docs still match the shipped behavior.
2. Run memory-improvement proposals if any friction happened during planning or execution.
3. Accept only useful, surgical instruction/harness improvements.

Expected output:
- Delivery complete.
- Optional process improvements captured for next cycle.

## Change and Enhancement Workflow (After Initial Delivery)

When new requests arrive, do not jump straight to coding. Re-enter the same SDD loop with the right level of planning.

### A) Decide change size first

1. Bug fix (no behavior expansion)
- Example: hydration count resets incorrectly at midnight.
- Action: create or update one focused task file under existing feature.

2. Small enhancement (extends existing capability)
- Example: add 250ml/500ml quick-add buttons.
- Action: add a new task in same feature (for example `T004-quick-add-volume.md`) and re-plan only impacted streams.

3. Major enhancement (new capability area)
- Example: add sleep tracking module.
- Action: create a new feature folder (`F00X-sleep-tracking`) and run full story -> plan -> execute cycle.

### B) Update docs before planning

User action:
1. Update or add transcript notes if change came from stakeholder discussion.
2. Add/adjust story markdown for the new requirement.
3. Add or revise task markdown in `docs/features/`.

Why this matters:
- Planner quality depends on current docs. Stale docs produce stale plans.

### C) Re-plan safely (plan refinement process)

If the previous plan is no longer accurate, ask for plan refinement explicitly.

Example:

```text
@orchestrator
Refine the approved plan for F001 due to new enhancement:
add customizable reminder windows and snooze options.
Keep completed streams unchanged.
Re-plan only impacted tasks and acceptance criteria.
```

What to verify in refined plan:
1. Completed streams are frozen unless regression fix is required.
2. New/changed acceptance criteria are explicit.
3. Dependencies between old and new tasks are clearly stated.

Then explicitly approve refined plan.

### D) Run delta implementation and review

User action in Copilot Chat:

```text
@orchestrator
Execute implement-and-review for the refined plan only.
Validate only impacted and newly added acceptance criteria.
```

Expected output:
- Only changed streams are implemented.
- Reviewer confirms changed ACs pass and no regressions are introduced.

### E) Change log discipline

For every enhancement cycle, keep these updated:
1. Feature `overview.md` scope statement
2. Task statuses and acceptance criteria in `T00X-*.md`
3. Any prompt/instruction updates approved through memory-improvement

## Sample: End-to-End Commands

Use these in sequence for a new feature.

1. Generate stories

```text
/generate-stories-from-transcript
Use docs/transcript/transcript.txt.
Create INVEST user stories and save them as numbered markdown files.
```

2. Plan

```text
@orchestrator
Run research-and-plan for F001 water tracking and T001-log-water-intake.
```

3. Approve

```text
approved
```

4. Execute + review

```text
@orchestrator
Run implement-and-review using the approved plan.
```

5. Enhancement re-plan (later)

```text
@orchestrator
Refine plan for F001 with enhancement: smart reminders by local timezone.
Re-plan only impacted tasks, then execute and review.
```

## Quick Prompting Examples

Use these as command starters in Copilot Chat.

```text
@orchestrator
Run generate stories from docs/transcript/transcript.txt and create INVEST vertical slices.
```

```text
@orchestrator
Run research-and-plan for F001 water tracking, starting with T001 log water intake.
```

```text
@orchestrator
Use the approved plan and run implement-and-review until all acceptance criteria pass.
```

## Notes

- The workshop harness enforces explicit approval before implementation.
- Keep stories vertical and user-visible (avoid purely technical slices).
- Keep docs current; stale docs degrade planning and implementation quality.
