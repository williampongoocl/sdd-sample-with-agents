---
applyTo: "**"
---

# Prompt Files — Best Practices

> Source: https://code.visualstudio.com/docs/copilot/copilot-customization

## What Prompt Files Are For

Prompt files (`.prompt.md`) define **repeatable, task-focused workflows** — things like scaffolding a component, preparing a pull request, or generating stories from a transcript. They are task-specific by design, unlike instruction files which must be task-agnostic.

For workflows that involve shell scripts or external service calls, use an agent skill instead.

## File Location

```
.github/
└── prompts/
    └── NAME.prompt.md
```

## Frontmatter Fields

```yaml
---
description: "Brief description shown in the prompt picker"
mode: agent # ask | edit | agent
tools: # tools available when mode is "agent"
  - codebase
  - search
  - editFiles
model: claude-sonnet-4-6 # optional model override
---
```

| Field         | Values                 | Purpose                                    |
| ------------- | ---------------------- | ------------------------------------------ |
| `description` | String                 | Shown as placeholder/picker text           |
| `mode`        | `ask`, `edit`, `agent` | Execution context for the prompt           |
| `tools`       | List of tool names     | Tools available when mode is `agent`       |
| `model`       | Model ID string        | Override the default model for this prompt |

**`description` style:** imperative verb phrase → what it does → when to run it, in a single sentence.
Example: `"Execute an approved plan via @implementer streams then gate with @reviewer — run this after research-and-plan.prompt.md."`

## Mode Reference

| Mode    | When to use                                           |
| ------- | ----------------------------------------------------- |
| `ask`   | Question-and-answer; read-only; no file changes       |
| `edit`  | Targeted file edits with explicit scope               |
| `agent` | Full agentic loop; can use tools, search, write files |

## Rules and Constraints

- **Task-specific** — prompts are designed for repeatable tasks. General standards and behavioral rules belong in `.instructions.md` files instead.
- **Use `agent` mode for orchestration** — prompts that dispatch sub-agents or use multiple tools must use `mode: agent`.
- **Keep prompts focused** — one prompt per workflow. Split complex multi-phase flows into separate prompts (e.g., `research-and-plan.prompt.md` and `implement-and-review.prompt.md`).
- **Instructions are inherited** — prompts automatically receive all active instruction files; do not duplicate rules from instruction files inside prompts.
- **No tool access in `ask` mode** — if a prompt needs to read the codebase or call tools, use `agent` or `edit` mode instead.
- **Prompts vs. skills** — use a prompt for simpler, self-contained task workflows. Use a skill when the workflow is complex, multi-step, or needs to be shared across multiple agents.

## When to Use Prompts vs. Other Customisations

| Need                                                    | Use                        |
| ------------------------------------------------------- | -------------------------- |
| Repeatable task workflow (scaffold, PR prep, story gen) | `.prompt.md`               |
| Persistent coding standards or behavioral rules         | `.instructions.md`         |
| Complex workflow with scripts or external tools         | Skill (`SKILL.md`)         |
| Specialised AI persona with tool restrictions           | Custom agent (`.agent.md`) |

## Generating Prompt Files

Use `/create-prompt` in Copilot Chat to generate prompt files with AI assistance.
