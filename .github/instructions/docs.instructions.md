---
applyTo: "**"
---

# Docs Folder — Best Practices

## What the `docs/` Folder Is For

The `docs/` folder is the project's **knowledge base** — the authoritative source of context, documentation, and reference material that Copilot and agents use to understand the project. It is not a VS Code-native feature, but a project convention that makes all Copilot customisations more effective by giving them grounded, accurate context to draw from.

Well-structured docs mean agents produce more accurate plans, more relevant code, and fewer hallucinated assumptions.

## Recommended Structure

```
docs/
├── setup/          # Environment setup, tool configuration, onboarding guides
├── stories/        # User stories, feature specs, acceptance criteria
├── transcript/     # Meeting transcripts, interview recordings (source material)
├── architecture/   # System design, component diagrams, decision records
└── reference/      # API references, data models, third-party integration notes
```

## Rules and Constraints

- **Store source material here, not in instructions** — transcripts, specs, and reference docs belong in `docs/`. Instruction files (`.instructions.md`) should contain rules and standards, not raw content.
- **Keep docs up to date** — outdated docs are worse than no docs; they lead agents to make decisions based on stale context. Update docs when the project changes.
- **Use descriptive filenames** — name files so their purpose is obvious without opening them (e.g., `module-4-1-harness.md`, `auth-flow-diagram.md`).
- **Prefer markdown** — Copilot reads markdown efficiently. Use headings, tables, and code blocks to structure content so agents can locate specific sections quickly.
- **One file per concern** — avoid monolithic docs. Split large documents into focused files; agents retrieve context more precisely from smaller, well-scoped files.
- **Link, don't duplicate** — if two docs reference the same information, link to the authoritative file rather than copying content that may drift out of sync.
- **Add a summary at the top** — begin each doc file with a brief statement of what it contains; this helps agents decide whether to read the full file before doing so.
- **Use version or date markers on time-sensitive docs** — agents assess recency; unmarked documents may be treated as current even when outdated.

## What Belongs in `docs/` vs. Other Locations

| Content type | Location |
|-------------|----------|
| Coding standards, conventions | `.github/instructions/*.instructions.md` |
| Repeatable task workflows | `.github/prompts/*.prompt.md` |
| Agent personas and tool configs | `.github/agents/*.agent.md` |
| Cross-cutting reusable workflows | `.github/skills/*/SKILL.md` |
| Lifecycle automation | `.github/hooks/*.json` |
| Project context, specs, transcripts | `docs/` |
| Architecture decisions (ADRs) | `docs/architecture/` |
| Setup and onboarding guides | `docs/setup/` |

## How Agents Use `docs/`

The `researcher` agent searches `docs/` when gathering context for a task. To make docs maximally useful to agents:

- Add a brief summary at the top of each file describing what it contains
- Use consistent heading levels so agents can navigate via structure
- Avoid deeply nested content — flat, well-headered files are easier to search and retrieve
- Include dates or version markers on time-sensitive documents so agents can assess recency
