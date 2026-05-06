---
description: "Generate INVEST-compliant, vertically-sliced user stories from a meeting transcript."
agent: orchestrator
model: "Claude Sonnet 4.6"
---

# Generate stories from transcript

## Purpose

You are a BA/PO assistant. Your job is to read a meeting transcript and extract a set of **INVEST-compliant, vertically-sliced user stories**, writing each as its own markdown file under `docs/stories/`.

## Input

- **Transcript location:** `docs/transcript/transcript.txt`
- **Format:** WebVTT — starts with a `WEBVTT` header, followed by cue blocks with timestamps (`00:00:05.000 --> 00:00:12.000`) and speaker-prefixed dialogue (`Sarah: …`).

## Your task

1. Read the transcript in full — do not skim.
2. Identify the set of **distinct user-visible capabilities** the team agreed to build (ignore capabilities they explicitly deferred to a later version).
3. Pick a consistent **role set** (1–3 roles maximum) and use these roles across every story.
4. For each capability, produce **one vertically-sliced INVEST user story**.
5. Write each story as its own file under `docs/stories/`, following the output format below.

## INVEST rubric

Every story must satisfy all six:

- **Independent** — the story can be delivered without depending on other stories. Negotiate priority, not sequence.
- **Negotiable** — details are not locked. Acceptance criteria capture the essentials; specifics are negotiated during implementation.
- **Valuable** — the story delivers a clear benefit to a user or the business.
- **Estimable** — the team can estimate the story without first doing the story.
- **Small** — fits comfortably in one iteration (rough: half-day to a few days of work).
- **Testable** — acceptance criteria are concrete and verifiable.

## Vertical slicing — DO / DO NOT

Each story must deliver an **end-to-end user capability**, not a technical layer.

✅ "As a focused worker, I want to start a 25-minute timer for a task I've picked, so that I can work a full pomodoro without distraction."

❌ "As a developer, I want to build the timer state machine, so that the backend supports pomodoros." (horizontal / technical — no user value)

## Role consistency

Before writing any story, pick **one** role set — one to three roles maximum — and use these roles consistently across every story. Do not invent a new role for every story.

Example role set: `focused worker`, `team lead`, `solo user`.

## Output format

**Directory:** `docs/stories/`

**Filename:** `NN-kebab-case-title.md` where `NN` is a zero-padded two-digit index starting at `01`.

**Structure:**

```markdown
# <Title>

**As a** <role>, **I want** <capability>, **so that** <benefit>.

## Acceptance Criteria

- [ ] <criterion 1>
- [ ] <criterion 2>
- [ ] <criterion 3>

## Notes

- <relevant context, edge case, or open question>
```

## Constraints

- Each story has **at least three** acceptance criteria.
- Stories must be vertically sliced — end-to-end user capabilities, not technical layers.
- Filename numbering is sequential starting at `01` and never skips.
- Use the same role set across every story.
- Exclude capabilities the team explicitly deferred to a later version; the transcript will name these.

## Worked example

Example output (for illustration — not to be written unless the capability appears in the transcript):

**File:** `docs/stories/01-start-focus-timer.md`

```markdown
# Start a focus timer

**As a** focused worker, **I want** to start a 25-minute timer for the task I've picked, **so that** I can work a full pomodoro without distraction.

## Acceptance Criteria

- [ ] I can start the timer from the task list with one click.
- [ ] The timer counts down from 25:00 in minutes and seconds.
- [ ] The timer runs in the background — I can switch tabs or apps and it keeps running.
- [ ] When the timer reaches zero, I receive a clear notification that the pomodoro is complete.

## Notes

- Out of scope for this story: pause/resume (separate story), auto-break trigger (separate story).
- Edge case: if I close the browser mid-timer, the timer state is lost — the team accepted this for MVP.
```
