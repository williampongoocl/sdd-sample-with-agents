---
applyTo: "**"
---

# Custom Instructions — Best Practices

> Source: https://code.visualstudio.com/docs/copilot/copilot-customization
> Source: https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot

## What Instruction Files Are For

Instruction files (`.instructions.md`) define **persistent coding standards, project conventions, and behavioral rules** that Copilot applies automatically across interactions. They are **not** for task-specific workflows — use a `.prompt.md` file for that.

## File Locations

| Type | Location | Scope |
|------|----------|-------|
| Repository-wide (always-on) | `.github/copilot-instructions.md` | Applies to all requests in this repo |
| Path-specific | `.github/instructions/NAME.instructions.md` | Applies to files matching `applyTo` glob |
| Subdirectories | `.github/instructions/subdir/NAME.instructions.md` | Supported for organisation |

## Required Frontmatter

Path-specific instruction files **must** include `applyTo` in their frontmatter:

```yaml
---
applyTo: "**"                    # all files
# applyTo: "**/*.ts,**/*.tsx"   # TypeScript files only
# excludeAgent: "code-review"   # optional: skip for a specific agent
---
```

## Glob Pattern Reference

| Pattern | Matches |
|---------|---------|
| `**` or `**/*` | All files in all directories |
| `*.py` | All `.py` files in current directory only |
| `**/*.py` | All `.py` files recursively |
| `src/**/*.py` | All `.py` files recursively under `src/` |
| `**/*.ts,**/*.tsx` | Multiple patterns — comma-separated |

## Rules and Constraints

- **Not task-specific** — instructions encode general, persistent rules. For repeatable tasks, use a `.prompt.md` file instead.
- **Keep concise** — auto-generated instructions must be no longer than 2 pages. Longer instructions risk being ignored or truncated.
- **Whitespace is ignored** — instructions can be a single paragraph or separated by blank lines; formatting within the file does not affect how they are applied.
- **Priority order** — Personal instructions override Repository instructions, which override Organisation instructions.
- **PR reviews** — When reviewing pull requests, Copilot uses instructions from the base branch.
- **`excludeAgent`** — Use the optional `excludeAgent` frontmatter field to prevent specific agents from receiving an instruction file.

## When to Use Instructions vs. Other Customisations

| Need | Use |
|------|-----|
| Persistent coding standards or behavioral rules | `.instructions.md` |
| Repeatable task workflow | `.prompt.md` |
| Complex workflow with scripts or external tools | Skill (`SKILL.md`) |
| Specialised AI persona with tool restrictions | Custom agent (`.agent.md`) |
| Lifecycle automation (format, guard, log) | Hook (`.json`) |

## Generating Instruction Files

Use `/create-instruction` in Copilot Chat to generate instruction files with AI assistance.
