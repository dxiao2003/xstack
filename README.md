# Web App Agent Skills

This repository contains agent skills for building full-stack web apps.

## `bootstrap-webapp-project`

Dynamically bootstraps a new full-stack web app project. The skill instructs the agent to check current official documentation for the selected frameworks, run the best supported scaffold commands, generate project-specific files, and set up Docker Compose for reproducible local development with bind-mounted source and automatic reload.

Use it by asking your agent:

```text
Use the bootstrap webapp project skill to initialize this repo as a new Docker-first web app project.
```

## `build-webapp-initial-increment`

Builds the first real product increment in a greenfield or near-empty web app with a runnable foundation. The skill narrows the product prompt to one vertical slice, establishes initial architecture decisions in `ARCHITECTURE.md`, derives acceptance criteria, writes focused tests, implements the workflow, validates it, and opens a review PR by default.

Use it by asking your agent:

```text
Use the build webapp initial increment skill to build the first product workflow for this app: <product description>.
```

## `build-webapp-feature`

Builds a new feature into an existing web app from a natural-language prompt, running non-interactively end to end. The skill works on a new feature branch: it vets the request against the current code, derives concrete testable acceptance criteria, writes unit/integration/e2e tests, implements the feature, and iterates until the tests pass — logging risks, warnings, ambiguities, and a final summary to a git-ignored `WORKLOG.md`. By default it finishes by committing the work and opening a pull request for the user to review.

Use it by asking your agent:

```text
Use the build webapp feature skill to add <feature description> to this app.
```

## Skill location and agent compatibility

Each skill is authored once and exposed to multiple agents:

- **Codex / OpenAI**: reads the skill from `.agent/skills/<skill-name>` (with interface metadata in `agents/openai.yaml`).
- **Claude / Claude Code**: discovers the skill from `.claude/skills/<skill-name>`, which is a symlink to the canonical `.agent/skills/<skill-name>` directory. The `SKILL.md` frontmatter (`name` + `description`) is the format Claude Code expects, so no separate copy is needed.

Because each `.claude/skills/<skill-name>` is a symlink rather than a copy, there is a single source of truth: edit the files under `.agent/skills/<skill-name>` and both agents pick up the change.
