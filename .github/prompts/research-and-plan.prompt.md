---
description: "Gather context via parallel @researcher agents then produce a user-approved implementation plan via @planner — run this before implementing any feature or task."
agent: orchestrator
model: "Claude Sonnet 4.6"
tools:
  - read
  - search
  - agent
  - todo
---

# Research & Plan

Run the GATHER and PLAN phases for the task described below.

## Phase 1: GATHER

**Before dispatching researchers, classify the task:**

| Type | Definition |
| ---- | ---------- |
| **Product Feature** | Adds or changes user-facing functionality in the Next.js app under `src/` |
| **Meta-task** | Modifies `.github/` infrastructure (agents, prompts, instructions, skills, hooks) or `docs/` organisation |
| **Both** | Spans product feature work AND `.github/`/docs changes simultaneously |

If ambiguous, default to **Product Feature** (conservative — runs more researchers, not fewer).

Dispatch `@researcher` agents in parallel. Scope depends on task type:

**If Product Feature or Both — dispatch all three:**

```
runSubagent(agentName: "researcher", prompt: "Harness Config Research for [task]. Read all relevant files under .github/ — agents, prompts, instructions, skills, hooks. Identify applicable rules, constraints, and conventions that govern this task. Return a Harness Config Report: what files exist, what rules apply, and any gaps or conflicts.")

runSubagent(agentName: "researcher", prompt: "Docs Research for [task]. Read docs/ — any existing documentation, transcripts, workshop guides, or feature notes. Identify anything relevant to the task. Return a Docs Report: what documentation exists, what is accurate, and what is missing or outdated.")

runSubagent(agentName: "researcher", prompt: "Source Code Research for [task]. Scan the Next.js app under src/. Identify what is already implemented relevant to this task — components, hooks, utilities, API routes, state management. Surface any TODOs, stubs, or mocked implementations. Return a Code Reality Report: what code exists and is complete, what is partial or mocked, and what is entirely absent.")
```

**If Meta-task only — dispatch harness and docs researchers; skip source code:**

```
runSubagent(agentName: "researcher", prompt: "Harness Config Research for [task]. Read all relevant files under .github/. Find applicable rules, constraints, and conventions. Return a Harness Config Report: what files exist, what rules apply, and any gaps or conflicts.")

runSubagent(agentName: "researcher", prompt: "Docs Research for [task]. Read docs/ for anything relevant to the task. Return a Docs Report: what documentation exists, what is accurate, and what is missing or outdated.")
```

State the skip explicitly in your consolidation summary when the source code researcher is omitted.

**After researchers return:**

1. Consolidate findings into a coherent summary
2. Identify gaps or open questions
3. If any requirement is ambiguous or could be satisfied multiple ways, enumerate open questions as a numbered list. For each item state: (a) the question, (b) what decision it unlocks, (c) the default assumption if the participant says "proceed with defaults." Do NOT dispatch `@planner` until all questions are answered or the participant accepts the defaults.

---

## Phase 2: PLAN

Delegate to `@planner` with consolidated context:

```
runSubagent(
  agentName: "planner",
  prompt: "Create a comprehensive implementation plan for [task].

  Context from researchers:
  - Harness config: [from harness researcher]
  - Docs: [from docs researcher]
  - Source code: [from code researcher — omit if meta-task only]

  User requirements:
  - [consolidated requirements]

  Output a self-contained plan document using the required format below. Iterate with the participant until explicit approval.

  ## Required Plan Output Format

  ### Overview
  [1–3 sentence summary]

  ### Stream 1 — [Name]
  - Files: [list]
  - Changes: [description]
  - Dependencies: [none, or other stream]

  ### Stream N — [Name]
  [...]

  ### Acceptance Criteria
  - [ ] [criterion]

  ### Review Gates
  - Tests: [what must pass]
  - Linting: [rules that apply]
  - Docs alignment: [what docs must match after implementation]"
)
```

### Step 2B — Brainstorm (Optional)

After the planner returns a draft plan, assess whether the brainstorm skill should be invoked. Trigger when any of these conditions are true:

- The planner explicitly flags competing approaches or unresolved trade-offs
- The plan contains contested architectural decisions with no clear winner
- Multiple viable strategies exist and the planner's recommendation confidence is low

If triggered, load and follow `.github/skills/brainstorm/SKILL.md`. Pass the planner's draft plan as the problem statement.

Note: No `brainstorm-*` sub-agents exist in this workshop. The brainstorm skill is executed inline by the orchestrator — see the skill file for the exact protocol.

After the brainstorm completes: if the decision surfaces critical concerns or a superior approach, feed it back to `@planner` for one revision pass. If it confirms the plan is sound, proceed to Exit with the brainstorm decision appended as supporting evidence.

If not triggered, skip directly to Exit.

### Step 2C — Memory Improvement Check

After the plan is approved and before closing this phase, check whether the GATHER or PLAN phase surfaced any Proactive Improvement Opportunity signals: useful conventions discovered, undocumented patterns, or gaps that required improvisation.

Load `.github/skills/memory-improvement/SKILL.md` and present any proposals to the participant using the format defined there.

Memory proposals are NEVER automatically applied — they are presented as standalone proposals requiring separate explicit approval.

---

**Exit**: Participant explicitly approves the plan. The approved plan is the deliverable — pass it directly into `implement-and-review.prompt.md`.

Approval phrases: "yes", "approved", "go ahead", "proceed", "do it"
Ambiguous ("sounds good", "okay") → clarify before proceeding

---

## Task

$task
