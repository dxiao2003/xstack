# Web App Project Bootstrap Skill

This repository contains an agent skill for dynamically bootstrapping new full-stack web app projects.

The skill instructs the agent to check current official documentation for the selected frameworks, run the best supported scaffold commands, generate project-specific files, and set up Docker Compose for reproducible local development with bind-mounted source and automatic reload.

Use it by asking your agent:

```text
Use the bootstrap webapp project skill to initialize this repo as a new Docker-first web app project.
```

## Skill location and agent compatibility

The skill is authored once and exposed to multiple agents:

- **Codex / OpenAI**: reads the skill from `.agent/skills/bootstrap-webapp-project` (with interface metadata in `agents/openai.yaml`).
- **Claude / Claude Code**: discovers the skill from `.claude/skills/bootstrap-webapp-project`, which is a symlink to the canonical `.agent/skills/bootstrap-webapp-project` directory. The `SKILL.md` frontmatter (`name` + `description`) is the format Claude Code expects, so no separate copy is needed.

Because `.claude/skills/bootstrap-webapp-project` is a symlink rather than a copy, there is a single source of truth: edit the files under `.agent/skills/bootstrap-webapp-project` and both agents pick up the change.
