# Agent Skills

Provider-agnostic Agent Skills for AI coding agents.

This repository stores reusable skills that follow the open Agent Skills format. Skills are intended to work across compatible agents such as Claude Code, OpenCode, Codex, Cursor, and related tools.

## Structure

```text
skills/
  skill-name/
    SKILL.md
```

Each skill lives in its own folder and includes a `SKILL.md` file with YAML frontmatter.

No skills are included yet. The `skills/` directory is reserved for future skills.

## Install

Install with the `skills` CLI:

```sh
npx skills add nestoralonsovina/agent-skills
```

List available skills before installing:

```sh
npx skills add nestoralonsovina/agent-skills --list
```

Install to specific agents:

```sh
npx skills add nestoralonsovina/agent-skills -a claude-code -a opencode -a codex
```

## Skill format

```markdown
---
name: example-skill
description: Explain what this skill does and when agents should use it.
license: MIT
---

## Instructions

Follow these steps...
```

## References

- Agent Skills specification: https://agentskills.io/specification
- Skills directory: https://skills.sh

## License

MIT
