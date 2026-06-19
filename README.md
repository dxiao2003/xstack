# Web App Project Bootstrap Skill

This repository contains an agent skill for dynamically bootstrapping new full-stack web app projects.

The skill instructs the agent to check current official documentation for the selected frameworks, run the best supported scaffold commands, generate project-specific files, and set up Docker Compose for reproducible local development with bind-mounted source and automatic reload.

Use it by asking Codex:

```text
Use the bootstrap webapp project skill to initialize this repo as a new Docker-first web app project.
```

The skill lives at `.agent/skills/bootstrap-webapp-project`.
