# docs/features

Features are **business-level capabilities**. Each feature lives in its own folder (`F<NNN>-<slug>/`) containing:

- `overview.md` — the feature's purpose, scope, linked stories, and open questions
- `T<NNN>-<slug>.md` — one file per technical task under this feature

Tasks (`TXXX`) are the **technical children** of features. A feature typically spans 2–6 tasks. Tasks map 1:1 with implementation-plan units and are the unit of work an `@implementer` agent picks up.

## Using the TEMPLATE/

`TEMPLATE/` holds blank templates for new features and tasks. When starting a new feature:

1. Copy `TEMPLATE/` to `F<NNN>-<feature-slug>/` (e.g. `F001-start-focus-timer/`).
2. Rename `TXXX-task-template.md` to `T<NNN>-<task-slug>.md` (e.g. `T001-render-timer-view.md`). Create further `T002-`, `T003-` files as needed.
3. Fill in the `{{PLACEHOLDER}}` tokens in each file.
4. Link the feature from the root `README.md` under a "Features" section.

Numbering is **sequential across the repo** — `F001`, `F002`, `T001`, `T002` — not scoped per feature area. This keeps cross-linking unambiguous and the backlog easy to scan.
