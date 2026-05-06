---
description: "Execute an approved plan via @implementer streams then gate with @reviewer — run this after research-and-plan.prompt.md has produced a participant-approved plan."
agent: orchestrator
model: "Claude Sonnet 4.6"
tools:
  - read
  - search
  - agent
  - todo
---

# Implement & Review

Run the EXECUTE and REVIEW phases for the approved plan below.

## Phase 3: EXECUTE

**Before dispatching implementers, confirm the plan is approved.**

Do NOT start implementation from a draft. The plan input to this prompt must be the explicitly-approved output of `research-and-plan.prompt.md`. If the plan is ambiguous or contains unresolved open questions, stop and return to the participant rather than guessing.

**Plan the dispatch order.**

Review the approved plan's streams and identify:

1. **Independent streams** — can run in parallel. Verify their file scopes do not overlap.
2. **Dependent streams** — must run sequentially. Note each dependency.
3. **Serial-only tasks** — anything that the plan explicitly marks as serial or that touches shared state.

**Dispatch implementers:**

For each parallel-safe stream, dispatch one `@implementer` with a focused brief:

```
runSubagent(agentName: "implementer", prompt: "Execute Stream [N] — [Name] from the approved plan.

Assigned files:
- [file path 1]
- [file path 2]

Changes required: [from plan]

Acceptance criteria applicable to this stream:
- [ ] [AC 1]
- [ ] [AC 2]

Dependencies: [none | completed stream X]

Report back with a summary of files affected, any issues encountered, and any concerns about overlap with other streams.")
```

Wait for all parallel implementers in the current batch to return before moving on. If an implementer reports a blocker, surface it to the participant before proceeding.

**After each batch completes, proceed to REVIEW for that batch before starting the next batch.**

---

## Phase 4: REVIEW

Dispatch `@reviewer` with the approved plan, the acceptance criteria for the completed batch, and the list of files touched:

```
runSubagent(agentName: "reviewer", prompt: "Review the implementation of Stream [N] — [Name] against the approved plan.

Approved plan stream scope:
[paste relevant plan section]

Acceptance criteria to verify:
- [ ] [AC 1]
- [ ] [AC 2]

Files affected in this stream:
[list from implementer report]

Return a structured report: verdict, per-AC results with evidence, issues found, and non-blocking observations.")
```

**Handle the reviewer verdict:**

- `All phase ACs satisfied — phase complete` → log completion, move to the next batch or to Exit if this was the last.
- `Revisions required` → route each issue back to the originating `@implementer` with a focused fix brief. After fixes, re-dispatch `@reviewer` for the same stream. Repeat until verdict is `All phase ACs satisfied`.
- `Blocked — need orchestrator decision` → return to the participant with the reviewer's specific question and wait for an answer.

### Step 4B — Memory Improvement Check

After the final review gate passes, check whether the EXECUTE or REVIEW phase surfaced any Proactive Improvement Opportunity signals: repeated gotchas, undocumented patterns encountered, or places where the plan proved incomplete.

Load `.github/skills/memory-improvement/SKILL.md` and present any proposals to the participant using the format defined there.

Memory proposals are NEVER automatically applied — they are presented as standalone proposals requiring separate explicit approval.

---

**Exit**: All streams implemented, all ACs verified by the reviewer, participant explicitly signs off.

Sign-off phrases: "done", "signed off", "ship it", "approved", "merge"
Ambiguous ("looks good", "nice") → clarify before closing

---

## Plan

$plan
