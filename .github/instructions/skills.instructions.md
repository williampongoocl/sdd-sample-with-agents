---
applyTo: "**"
---

# Agent Skills — Best Practices

> Source: https://code.visualstudio.com/docs/copilot/copilot-customization

## What Skills Are For

Agent skills package **complex, multi-step workflows** that involve scripts and external tools. They are more powerful than prompt files and are designed to be **reusable across agents and tools**.

Use a skill when:
- The workflow involves shell scripts or external service calls
- The capability needs to be shared across multiple agents
- The workflow is too complex or stateful for a single prompt file

For simpler repeatable tasks, use a `.prompt.md` instead.

## File Location

```
.github/
└── skills/
    └── SKILL-NAME/
        └── SKILL.md
```

Each skill lives in its own subdirectory under `.github/skills/`. The `SKILL.md` file is the entry point and the only required file.

## SKILL.md Structure

A `SKILL.md` file typically contains:

1. **Purpose statement** — what the skill does and when to invoke it
2. **Trigger signals** — conditions that indicate the skill should be used
3. **Protocol/steps** — the ordered workflow the agent executes inline
4. **Output format** — what the skill produces
5. **Rules and constraints** — what the skill must and must not do

There is no required frontmatter for skills. The content is instructional prose consumed by the agent that invokes the skill.

## Rules and Constraints

- **Skills are invoked by agents, not users directly** — agents (e.g., the orchestrator) call a skill inline; skills do not appear in the user-facing prompt picker.
- **Keep skills stateless where possible** — skills should describe a reusable protocol, not maintain state between invocations.
- **One skill, one concern** — each skill should have a single, well-defined purpose. Compose multiple skills rather than building one monolithic skill.
- **Document trigger signals** — clearly describe when the invoking agent should activate the skill, so it is used at the right time and not over-applied.
- **Skills complement agents** — use skills for cross-cutting capabilities (e.g., brainstorming, memory improvement) that multiple agents might need, rather than embedding the logic in each individual agent.
- **Skills vs. prompts** — if the capability needs tool access, scripts, or external integrations, it belongs in a skill. If it is a simple repeatable task, use a prompt.

## When to Use Skills vs. Other Customisations

| Need | Use |
|------|-----|
| Cross-cutting reusable capability with scripts/tools | Skill (`SKILL.md`) |
| Repeatable task workflow | `.prompt.md` |
| Persistent coding standards or behavioral rules | `.instructions.md` |
| Specialised AI persona with tool restrictions | Custom agent (`.agent.md`) |

## Generating Skills

Use `/create-skill` in Copilot Chat to generate skill files with AI assistance.
