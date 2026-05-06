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

## How Transcript Becomes App Work (Step-by-Step)

This is the practical SDD path for this repository.

### Step 1: Start from transcript

Input source:
- `docs/transcript/transcript.txt`

You extract user-visible capabilities. From the existing transcript, examples are:
- Log daily water intake
- Customize daily water goal
- Optional reminder notifications

### Step 2: Turn transcript into stories

Use the story-generation prompt (`generate-stories-from-transcript`).

Expected output shape:
- Story files under `docs/stories/` (as designed by the prompt)
- One vertical slice per story, each with acceptance criteria

Example story list for this transcript:
1. Log daily water glasses
2. Set custom daily goal
3. Configure reminder notifications

### Step 3: Create feature/task docs under `docs/features/`

Use the existing template folder:
- Copy `docs/features/TEMPLATE/` to a feature folder such as `docs/features/F001-water-tracking/`
- Fill `overview.md`
- Rename `TXXX-task-template.md` to concrete tasks, for example:
  - `T001-render-water-dashboard.md`
  - `T002-track-glass-intake.md`
  - `T003-customize-water-goal.md`
  - `T004-reminder-preferences.md`

Task docs are the implementation units used by the plan/execution workflow.

### Step 4: Research and approve plan

In Copilot Chat:
1. Invoke `@orchestrator`
2. Run the `research-and-plan` workflow for a selected task/story
3. Review plan streams and acceptance criteria
4. Explicitly approve the plan

Output:
- An approved implementation plan with streams, dependencies, and review gates

### Step 5: Implement and review

In Copilot Chat:
1. Invoke `@orchestrator`
2. Run `implement-and-review` with the approved plan
3. `implementer` executes stream(s)
4. `reviewer` validates each acceptance criterion with evidence
5. If review fails, iterate fixes until pass

Output:
- Completed implementation plus AC-based review verdict

### Step 6: Close loop and improve process

After PLAN/REVIEW, run memory improvement check (as defined in skill) to propose small improvements to instructions/harness.

## Sample: End-to-End Run (Transcript -> App)

Use this concrete sequence.

1. Prepare source docs
- Ensure transcript exists in `docs/transcript/transcript.txt`
- Ensure feature template exists in `docs/features/TEMPLATE/`

2. Generate stories
- Ask orchestrator to run story generation from transcript
- Save resulting story files (for example under `docs/stories/01-*.md`, `02-*.md`, ...)

3. Create feature/task docs
- Create `docs/features/F001-water-tracking/`
- Write `overview.md` linking the story IDs
- Create tasks from template (`T001` to `T00N`) with concrete acceptance criteria

4. Plan implementation
- Ask orchestrator: run `research-and-plan` for `F001` and selected `T00N`
- Confirm ambiguity answers if asked
- Approve the final plan explicitly

5. Execute implementation streams
- Ask orchestrator: run `implement-and-review` using approved plan
- Let implementer complete stream 1, then reviewer gate
- Continue stream-by-stream (or in parallel where safe) until all ACs pass

6. Produce shippable result
- Confirm all task statuses are complete
- Ensure docs align with implemented behavior
- Run memory improvement proposals for future workflow refinement

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
