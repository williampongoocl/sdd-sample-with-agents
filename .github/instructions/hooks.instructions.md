---
applyTo: "**"
---

# Hooks — Best Practices

> Source: https://code.visualstudio.com/docs/copilot/copilot-customization

## What Hooks Are For

Hooks **run shell commands at key lifecycle points** in the Copilot workflow. They are used to automate tasks that should happen consistently at specific moments — such as running a formatter after every file edit, enforcing security policies before tool use, or logging tool activity for audit purposes.

## File Location

```
.github/
└── hooks/
    └── HOOK-NAME.json
```

## Lifecycle Points

| Hook point | When it fires |
|------------|--------------|
| `session-start` | When a Copilot agent session begins |
| `pre-tool-use` | Before any tool is invoked |
| `post-tool-use` | After a tool completes |

## Hook File Format (JSON)

```json
{
  "event": "pre-tool-use",
  "command": "echo 'tool: $TOOL_NAME'",
  "description": "Brief description of what this hook does"
}
```

## Valid Use Cases

- **Formatters** — run `prettier` or `eslint --fix` after every file edit
- **Security guards** — block destructive commands (e.g., `rm -rf /`, `git push --force`) before tool use
- **Logging** — append a timestamped entry to a log file after each tool use
- **Session setup** — display a welcome message or load environment context at session start
- **Policy enforcement** — validate that changes conform to rules before they are applied

## Rules and Constraints

- **Keep hooks fast** — hooks run synchronously at lifecycle points; slow hooks will delay the entire workflow. Avoid network calls or heavy computation in hooks.
- **Be specific** — target only the lifecycle events you actually need. Do not add a `pre-tool-use` hook for something that only needs to run at session start.
- **Fail loudly for guards** — if a hook is meant to block a dangerous action, ensure the command exits with a non-zero code so the action is halted.
- **No secrets in hook files** — hook JSON files are committed to the repository. Never embed API keys, tokens, or credentials in hook commands.
- **Test hooks independently** — verify the shell command works correctly in isolation before relying on it in the Copilot workflow.
- **Hooks are not task logic** — hooks handle lifecycle automation only. Task-specific logic belongs in prompts or skills.

## When to Use Hooks vs. Other Customisations

| Need | Use |
|------|-----|
| Lifecycle automation (format, guard, log) | Hook (`.json`) |
| Repeatable task workflow | `.prompt.md` |
| Persistent coding standards or behavioral rules | `.instructions.md` |
| Complex multi-step workflow | Skill (`SKILL.md`) |

## Generating Hooks

Use `/create-hook` in Copilot Chat to generate hook files with AI assistance.
