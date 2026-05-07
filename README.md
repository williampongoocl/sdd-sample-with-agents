# SDD Sample with Agents

This repo is a **Story-Driven Development (SDD)** harness built on GitHub Copilot's agent and customisation system. It shows how to take a raw meeting transcript and — step by step — turn it into working code using a coordinated team of AI agents.

---

## What is SDD?

Story-Driven Development is a workflow where:

1. **A meeting transcript** is the source of truth for what to build.
2. **User stories** (INVEST-compliant) are extracted from the transcript and become the backlog.
3. **Implementation plans** are generated from those stories before any code is written.
4. **Agents** research, plan, implement, and review in a structured, human-approved loop.

Every decision has an audit trail — from transcript to story to plan to code.

---

## Harness Map

The harness lives entirely under `.github/`. Here is what every file does.

### `.github/agents/` — The Agent Roster

Agents are specialised AI personas. Only `@orchestrator` is user-invocable. The others are dispatched internally.

| File | Agent | Role | User-invocable |
|------|-------|------|----------------|
| `orchestrator.agent.md` | `@orchestrator` | Entry point. Coordinates the full workflow (GATHER → PLAN → EXECUTE → REVIEW). Never writes code itself — delegates everything. | ✅ Yes — type `@orchestrator` in Copilot Chat |
| `researcher.agent.md` | `@researcher` | Gathers context from one assigned scope (codebase, docs, or web). Evidence-based: every finding cites a file path or URL. | ❌ Internal only |
| `planner.agent.md` | `@planner` | Converts research findings into a structured plan with phases, tasks, and acceptance criteria. Iterates with the user until explicit approval. | ❌ Internal only |
| `implementer.agent.md` | `@implementer` | Executes one stream of an approved plan. Writes and edits files exactly as the plan specifies. Multiple instances can run in parallel. | ❌ Internal only |
| `reviewer.agent.md` | `@reviewer` | Quality gate. Verifies each acceptance criterion after implementation with specific evidence (file path + line). Returns pass/fail verdict. | ❌ Internal only |

**Hub-and-spoke rule:** The orchestrator is always the hub. Specialists never talk to each other — all results flow back to the orchestrator before the next phase starts.

---

### `.github/instructions/` — Persistent Rules

Instruction files define rules that Copilot applies automatically to every interaction. They are not task workflows — they are standing conventions.

| File | `applyTo` | Purpose |
|------|-----------|---------|
| `agents.instructions.md` | `**` | Defines the 5-agent roster, their roles, dispatch rules, and the hub-and-spoke constraint. Reference for how the agent system is structured. |
| `copilot-interactions.instructions.md` | `**` | The four interaction protocols: Plan-First, Clarification-First, Constructive Challenge, and Implementation Discipline. Copilot must present a plan and wait for explicit approval before editing any file. |
| `docs.instructions.md` | `**` | Rules for the `docs/` folder: what belongs there, how to structure files, naming conventions, and how agents use docs for context. |
| `hooks.instructions.md` | `**` | Best practices for lifecycle hooks (session-start, pre-tool-use, post-tool-use): valid use cases, format, and constraints. |
| `instructions.instructions.md` | `**` | Meta-rules for writing instruction files themselves: frontmatter format, `applyTo` glob patterns, scope priority, and length limits. |
| `prompts.instructions.md` | `**` | Best practices for writing prompt files: frontmatter fields, mode reference (`ask`/`edit`/`agent`), and when to use prompts vs. skills. |
| `skills.instructions.md` | `**` | Best practices for writing agent skills: file location, `SKILL.md` structure, statelessness, and when to use skills vs. prompts. |

---

### `.github/prompts/` — Repeatable Task Workflows

Some prompts are invoked directly by the user as slash commands. Others are used **internally by the orchestrator** — the orchestrator runs them on your behalf when you speak to it in Copilot Chat. You never need to invoke the internal ones yourself.

| File | How to invoke | When it runs | What it does |
|------|---------------|--------------|--------------|
| `generate-stories-from-transcript.prompt.md` | `/generate-stories-from-transcript` (user-invoked) | After you place a transcript in `docs/transcript/transcript.txt` | Reads the transcript, extracts INVEST-compliant user stories, and writes each as a separate `.md` file under `docs/stories/` |
| `research-and-plan.prompt.md` | **Orchestrator-internal** — triggered when you tell `@orchestrator` what you want to implement | GATHER + PLAN phases, before any implementation begins | Dispatches parallel `@researcher` agents (codebase, docs, source code), consolidates findings, then dispatches `@planner` to produce a user-approved implementation plan |
| `generate-implementation-plan.prompt.md` | `/generate-implementation-plan` (user-invoked) | After approving the high-level plan from `@orchestrator` | Generates a detailed implementation plan document covering design, component structure, state management, testing, and task breakdown. Saves to `docs/implementation-plans/` |
| `implement-and-review.prompt.md` | **Orchestrator-internal** — triggered when you approve the plan in `@orchestrator` chat | EXECUTE + REVIEW phases, after plan approval | Dispatches `@implementer` streams (parallel where safe), then dispatches `@reviewer` to gate each batch before the next begins |
| `execute-implementation-plan.prompt.md` | `/execute-implementation-plan` (user-invoked) | When implementing one task at a time with granular control | Executes a single task from a story, validates against acceptance criteria, and updates task status checkboxes |

---

### `.github/skills/` — Cross-cutting Capabilities

Skills package complex workflows that are reused across agents. They are invoked inline by the orchestrator, not by the user directly.

| Folder | Skill | When it activates | What it does |
|--------|-------|-------------------|--------------|
| `memory-improvement/SKILL.md` | `memory-improvement` | At the end of GATHER/PLAN and after REVIEW — only when friction signals appear | Surfaces concrete proposals to improve instructions, conventions, or harness files based on what was learned during the task. Every proposal requires explicit user approval before anything changes. |

**Friction signals that trigger this skill:** the user had to correct the agent more than once on the same thing; a step felt unusually hard or repetitive; a convention was missing; the codebase's stated and actual conventions disagreed.

---

## End-to-End Walkthrough: Transcript → Running App

This walkthrough follows the full SDD lifecycle using the sample transcript in `docs/transcript/transcript.txt` (a product planning meeting about a health habit tracker app).

---

### Step 1 — Place your transcript

**What to do:** Save your meeting recording or notes as plain text in `docs/transcript/transcript.txt`.

The sample transcript already in this repo looks like:

```
Product Owner: So, our main goal is to help users build healthy daily routines — think hydration, sleep, steps, meditation.
Designer: I was thinking a dashboard showing streaks could motivate users, maybe with badges for milestones.
Developer: We'll need to track user activity daily...
PO: Let's start with water tracking — keep it simple. 8 glasses per day is the default, but users should be able to customize.
```

**Tips:**
- Include speaker labels (e.g., `PO:`, `Designer:`, `Dev:`) so the agent can attribute context.
- Explicitly note anything the team decided to **defer** — the agent will exclude deferred items from the story set.
- If you have a WebVTT file (from Teams/Zoom), it is supported directly. Place it as-is in `docs/transcript/transcript.txt`.

---

### Step 2 — Generate user stories from the transcript

**What to do:** Open Copilot Chat and run the prompt:

```
/generate-stories-from-transcript
```

**What the agent does:**
1. Reads `docs/transcript/transcript.txt` in full.
2. Identifies all distinct user-visible capabilities the team agreed to build.
3. Picks a consistent role set (e.g., `health-conscious user`, `first-time user`) used across every story.
4. Writes one INVEST-compliant, vertically-sliced user story per capability as a `.md` file under `docs/stories/`.

**Example stories generated from the sample transcript:**

- `docs/stories/01-track-daily-water-intake.md`
- `docs/stories/02-customize-water-goal.md`
- `docs/stories/03-enable-hydration-reminders.md`
- `docs/stories/04-view-habit-streak-dashboard.md`

**Example story file (`docs/stories/01-track-daily-water-intake.md`):**

```markdown
# Track daily water intake

**As a** health-conscious user, **I want** to log each glass of water I drink throughout the day, **so that** I can see my progress toward my daily hydration goal.

## Acceptance Criteria

- [ ] I can tap a button to add one glass (250 ml) to my daily count.
- [ ] The current count and goal (default 8 glasses) are visible on the home screen.
- [ ] My count resets to zero at midnight in my local timezone.
- [ ] If I reach my daily goal, I see a congratulatory indicator.

## Notes

- Default goal is 8 glasses; customisation is a separate story.
- Reminders are a separate story (03).
```

**What to check before moving on:**
- Open each generated story. Verify it describes an end-to-end user capability (not a technical task).
- If a story is too large (feels like more than a few days of work), ask the orchestrator to split it.
- If a story is missing, add it manually following the same format.

---

### Step 3 — Research and plan a story

**What to do:** Pick a story to implement. Open Copilot Chat and start a conversation with the orchestrator, describing what you want to build:

```
@orchestrator I want to implement story 01 — Track daily water intake.
The story is in docs/stories/01-track-daily-water-intake.md.
Please research the codebase and produce an implementation plan.
```

> **Note:** You do not need to type `/research-and-plan` manually. The orchestrator runs that workflow internally as soon as you describe your task.

**What the agent does (Phase 1 — GATHER):**

The orchestrator dispatches three `@researcher` agents in parallel, each scoped to a different area:

- **Harness researcher** — reads `.github/` to find applicable rules, agent configurations, and constraints.
- **Docs researcher** — reads `docs/` for any existing documentation relevant to this story.
- **Source code researcher** — scans `src/` to find what is already implemented, what is partial, and what is absent.

After all three return, the orchestrator consolidates findings and surfaces any open questions. For example:

> **Open question:** Should the water count persist across app restarts (local storage), or is in-memory state acceptable for MVP?
> — Default assumption: local storage persistence (matches the transcript's emphasis on daily tracking).

**What to do when open questions appear:**

Reply in chat with your decision, or type:

```
Proceed with defaults.
```

**What the agent does (Phase 2 — PLAN):**

The orchestrator dispatches `@planner`, which produces a structured plan:

```markdown
### Overview
Implement the water intake logging screen with a tap-to-add button, daily count display, midnight reset, and goal-reached indicator.

### Phase 1 — Data layer
- Files: src/store/waterStore.ts, src/types/water.ts
- Changes: Create Zustand store with count, goal, lastResetDate; add midnight-reset logic

### Phase 2 — UI layer
- Files: src/app/water/page.tsx, src/app/water/_components/WaterCounter.tsx
- Changes: Render count/goal, tap-to-add button, goal-reached banner

### Acceptance Criteria
- [ ] Tapping the button increments the count and persists to localStorage
- [ ] Count resets at midnight local time
- [ ] Goal-reached indicator appears when count >= goal
```

**What to do when the plan appears:**

Read the plan carefully. You can:
- **Approve it:** Type `Approved. Proceed.`
- **Request changes:** Type what you want changed, e.g. `Change Phase 1 to use React Context instead of Zustand.` The planner will revise and re-present.
- **Reject it:** Type `Rejected.` and explain why. The planner will start over.

**Do not proceed to Step 4 until you have explicitly approved the plan.**

---

### Step 4 — Generate a detailed implementation plan

**What to do:** After approving the high-level plan from Step 3, generate the detailed implementation plan document. Run:

```
/generate-implementation-plan
```

Then paste the story details and approved plan summary:

```
/generate-implementation-plan

Story: docs/stories/01-track-daily-water-intake.md

User Story:
As a health-conscious user, I want to log each glass of water I drink throughout the day,
so that I can see my progress toward my daily hydration goal.

Approved plan summary:
- Phase 1: Data layer — Zustand store in src/store/waterStore.ts
- Phase 2: UI layer — page.tsx + WaterCounter.tsx under src/app/water/
```

**What the agent does:**

Produces a comprehensive plan document saved to `docs/implementation-plans/01-track-daily-water-intake.md` covering:

- **Visual layout** — component arrangement, responsive breakpoints
- **Color & typography** — exact Tailwind classes for light/dark mode
- **Component structure** — file tree with each component and its responsibility
- **State management** — TypeScript interfaces for store and component state
- **Acceptance criteria** — grouped by layout, functionality, navigation, and error handling
- **Task breakdown** — checkboxed task list grouped into Setup, Layout, Feature, Testing
- **Testing requirements** — integration, performance, accessibility, and edge cases

**Example output structure in `docs/implementation-plans/01-track-daily-water-intake.md`:**

```markdown
## Status
⬜ NOT STARTED

1. Setup & Configuration
   - [ ] Install/configure Zustand
   - [ ] Create src/types/water.ts with WaterState interface

2. Layout Implementation
   - [ ] Scaffold src/app/water/page.tsx
   - [ ] Build WaterCounter.tsx with count + goal display

3. Feature Implementation
   - [ ] Wire tap-to-add button to store increment action
   - [ ] Implement midnight reset logic with timezone support
   - [ ] Show goal-reached banner when count >= goal

4. Testing
   - [ ] Unit test store actions
   - [ ] Integration test: tap increments count and persists
   - [ ] Edge case: count resets correctly at midnight
```

**What to check before moving on:**
- Confirm all acceptance criteria from the story appear in the plan.
- Check that the component structure matches your existing project conventions.
- Update task status to `[~] In Progress` when you start working.

---

### Step 5 — Implement and review

#### Option A — Full plan execution (recommended for multi-task stories)

Once you have approved the plan (in Step 3 or Step 4), continue the conversation in the same `@orchestrator` chat:

```
@orchestrator Approved. Please proceed with implementation.
Approved plan: docs/implementation-plans/01-track-daily-water-intake.md

Please execute all phases. Parallelise Phase 1 and Phase 2 if their file scopes do not overlap.
```

> **Note:** You do not need to type `/implement-and-review` manually. The orchestrator runs that workflow internally once you approve the plan.

**What the agent does:**

1. Analyses the plan for independent streams (can run in parallel) vs. dependent streams (must run in sequence).
2. Dispatches one `@implementer` per parallel-safe stream — each implementer touches only its assigned files.
3. After each batch completes, dispatches `@reviewer` with the relevant acceptance criteria and the list of files touched.
4. If the reviewer returns `Revisions required`, routes a fix cycle back to the implementer before proceeding.
5. Returns a final summary of all files changed.

**Example reviewer verdict:**

```markdown
## Verdict: Revisions required

| AC | Status | Evidence |
|----|--------|---------|
| Tapping increments count | pass | src/store/waterStore.ts:14 — increment() updates count |
| Count resets at midnight | fail | src/store/waterStore.ts — no reset logic found |
| Goal-reached indicator | pass | WaterCounter.tsx:42 — conditional banner renders |

### Issues found
1. Midnight reset is missing. The store has no lastResetDate field and no reset action.
   Add a `checkAndReset()` action called on app mount and on each increment.
```

**What to do with the verdict:**
- `All phase ACs satisfied — phase complete` → review the changes in your editor, then move to the next story.
- `Revisions required` → the orchestrator automatically routes fixes back to the implementer. You only need to confirm once the fix cycle is done.
- `Blocked — need orchestrator decision` → the agent will surface the blocker to you and wait for your input.

#### Option B — One task at a time (for granular control)

If you prefer to implement one task at a time:

```
/execute-implementation-plan
```

Specify exactly which task to execute:

```
/execute-implementation-plan

Story: docs/stories/01-track-daily-water-intake.md
Plan: docs/implementation-plans/01-track-daily-water-intake.md

Execute task: "Create src/types/water.ts with WaterState interface"

Acceptance criteria for this task:
- [ ] WaterState interface includes count (number), goal (number), lastResetDate (string | null)
- [ ] File is importable from the store without type errors
```

The agent will validate its input, execute only the specified task, and update the checkbox in your plan document from `[ ]` to `[x]`.

---

### Step 6 — Repeat for remaining stories

Go back to `docs/stories/` and pick the next story. Repeat Steps 3–5.

**Suggested order:** Prioritise stories with no dependencies on other stories first (the `Independent` criterion in INVEST). The orchestrator's research phase will surface any dependencies it finds.

---

## Handling Enhancements and Changes

When a new feature request, change request, or scope refinement comes in, follow this process:

---

### Scenario A — New requirement from a new meeting

1. **Update the transcript.** Add the new meeting output to `docs/transcript/transcript.txt`, or create a new file (e.g., `docs/transcript/transcript-sprint-2.txt`) and note the date.

2. **Re-run story generation** scoped to the new content:

   ```
   /generate-stories-from-transcript

   Please generate stories only from the new section of the transcript (Sprint 2 planning meeting, added below the existing content).
   The existing stories in docs/stories/ should not be modified.
   ```

3. **Number new stories sequentially** — if you already have `01` through `04`, new stories start at `05`.

4. **Proceed with Steps 3–5** for each new story.

---

### Scenario B — Change to an existing story (scope change)

1. **Edit the story file directly** in `docs/stories/NN-story-name.md`. Update the acceptance criteria to reflect the change.

2. **Re-run research and plan** by telling the orchestrator about the change:

   ```
   @orchestrator Story 02 (docs/stories/02-customize-water-goal.md) has been updated.
   The new requirement is: users can set a goal between 4 and 20 glasses (previously only 8 was supported).
   Please re-research and produce a revised plan. The existing implementation in src/app/water/ may need changes.
   ```

3. **Review the revised plan carefully.** The planner will diff the new plan against what was already built and highlight what needs to change.

4. **Approve the revised plan** before any files are touched.

---

### Scenario C — Refining an existing plan (before implementation starts)

If implementation has **not yet started** and you want to change the plan:

1. Open Copilot Chat and tell the orchestrator what to change:

   ```
   @orchestrator The current plan for story 01 uses Zustand for state management.
   We've decided to use React Context + useReducer instead.
   Please revise the plan accordingly — specifically Phase 1 (data layer) and the State Management section of the implementation plan.
   ```

2. The orchestrator will dispatch `@planner` with the revised requirement. The planner will produce an updated plan and present it for your approval.

3. Once approved, the orchestrator updates `docs/implementation-plans/01-track-daily-water-intake.md`.

---

### Scenario D — Mid-implementation change (implementation already started)

If implementation is **in progress** (some tasks are `[x]` completed, others are `[ ]` not started):

1. **Do not re-run the full plan from scratch.** Only revise the incomplete tasks.

2. Open Copilot Chat:

   ```
   @orchestrator Story 01 implementation is in progress.
   Tasks 1 and 2 are complete. Task 3 (midnight reset logic) needs to change —
   the product owner now wants the reset to happen at 3 AM instead of midnight.
   Please revise only the incomplete tasks in the plan to reflect this change.
   ```

3. The orchestrator will show you the revised tasks. Approve them before the implementer continues.

4. **Completed tasks (`[x]`) should not be re-implemented** unless the change explicitly requires it. Flag this if the orchestrator suggests touching completed work.

---

### Scenario E — Post-implementation bug or regression

If a review returns `Revisions required` or you discover a bug after implementation:

1. **Do not edit files manually.** Keep the agent audit trail intact.

2. Open Copilot Chat:

   ```
   @orchestrator The reviewer found that the midnight reset is missing in src/store/waterStore.ts.
   The acceptance criterion is: count resets to zero at midnight in the user's local timezone.
   Please dispatch @implementer to add a checkAndReset() action and call it on app mount and on each increment.
   Then dispatch @reviewer to verify the fix.
   ```

3. The orchestrator will run a targeted fix-and-verify cycle without re-implementing the whole story.

---

## Folder Reference

```
.github/
├── agents/           # Agent personas (orchestrator, researcher, planner, implementer, reviewer)
├── instructions/     # Persistent rules applied to every Copilot interaction
├── prompts/          # Task workflows — some user-invoked (/generate-stories-from-transcript, etc.), some orchestrator-internal
└── skills/
    └── memory-improvement/   # Reactive harness improvement proposals

docs/
├── transcript/       # Raw meeting transcripts (source of truth for what to build)
├── stories/          # Generated user stories (one .md per story)
├── features/         # Feature folders (F001-, F002-...) with overview.md and task files
│   └── TEMPLATE/     # Blank templates for new features and tasks
└── implementation-plans/   # Detailed implementation plan documents (generated by /generate-implementation-plan)
```

---

## Quick Reference

### User entry points

| How to invoke | When | Example |
|---------------|------|---------|
| `@orchestrator` + describe the story | Start GATHER + PLAN for any story (Steps 3–4) | `@orchestrator I want to implement story 01 — docs/stories/01-track-daily-water-intake.md. Please research and plan.` |
| `@orchestrator` + approve the plan | Start EXECUTE + REVIEW after plan approval (Step 5A) | `@orchestrator Approved. Please proceed with implementation.` |
| `/generate-stories-from-transcript` | Extract stories from the transcript (Step 2) | *(no extra input — reads `docs/transcript/transcript.txt` automatically)* |
| `/generate-implementation-plan` | Generate a detailed plan document after `@orchestrator` produces the high-level plan (Step 4) | Paste the story + approved plan summary |
| `/execute-implementation-plan` | Implement one task at a time for granular control (Step 5B) | Paste the specific task + its acceptance criteria |

### Orchestrator-internal prompts (no manual invocation needed)

| Prompt file | Triggered by | What it does |
|-------------|-------------|---------------|
| `research-and-plan.prompt.md` | Telling `@orchestrator` what to implement | Runs the full GATHER (researchers) + PLAN (planner) loop |
| `implement-and-review.prompt.md` | Approving a plan in `@orchestrator` chat | Runs the full EXECUTE (implementers) + REVIEW (reviewer) loop |

---

## Interaction Rules (summary)

These rules come from `.github/instructions/copilot-interactions.instructions.md` and apply to every Copilot interaction in this repo:

1. **Plan first** — Copilot presents a plan and waits for your explicit approval before touching any file.
2. **Clarify first** — If a request is ambiguous, Copilot asks questions before proposing a plan.
3. **Constructive challenge** — If Copilot spots an inconsistency (e.g., you asked for Day 2 data but only Day 1 exists), it flags it rather than guessing.
4. **No silent changes** — Copilot never makes "small" changes without approval. Explicit approval phrases: `Yes`, `Approved`, `Go ahead`, `Proceed`, `Do it`.
