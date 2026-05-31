# Compound Engineering (Pi)

AI-powered development tools for code review, research, design, and workflow automation.

## Dependencies

This plugin requires:

- **pi-subagents** (required) — provides the `subagent` tool used by skills that dispatch parallel agents
  ```bash
  pi install npm:pi-subagents
  ```
- **pi-ask-user** (recommended) — provides the `ask_user_question` tool; skills fall back to numbered options in chat when it is missing
  ```bash
  pi install npm:pi-ask-user
  ```

## Skills

All skills use the `ce-` prefix and are invoked via `/skill:ce-<name>`.

Key workflows:
- `/skill:ce-compound` — Full compound cycle (research + plan + work + review)
- `/skill:ce-code-review` — Structured code review with persona agents
- `/skill:ce-plan` — Implementation planning
- `/skill:ce-work` — Execute planned work
- `/skill:ce-brainstorm` — Requirements exploration

## Agents

Agents are resolved by pi-subagents when dispatched via the `subagent` tool.
All agent names use the `ce-` prefix (e.g., `ce-repo-research-analyst`).

## Conventions

- Agent names: `ce-<descriptive-name>` (e.g., `ce-security-reviewer`)
- Skill names: `ce-<workflow-name>` (e.g., `ce-code-review`)
- The `lfg` skill runs the full autonomous pipeline end-to-end
