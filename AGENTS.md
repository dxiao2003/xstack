# Repository Guidance

Guidance for coding agents working in this repository (Claude Code reads it via the
`CLAUDE.md` symlink). This repo contains agent skills for building full-stack web
apps — see `README.md` for the skill catalog and `.agent/skills/<skill-name>` for the
canonical source of each skill.

## Working Agreement

1. **Single source of truth.** Each skill is authored once under
   `.agent/skills/<skill-name>`. The `.claude/skills/<skill-name>` entries are symlinks,
   not copies — edit the files under `.agent/skills` and both agents pick up the change.

2. **Keep changes surgical.** Touch only what the request needs. Don't refactor what
   isn't broken or reformat untouched lines — it hides the real change and widens the
   blast radius of review.

3. **Title PRs as Conventional Commits.** Every pull request title must follow the
   [Conventional Commits](https://www.conventionalcommits.org/) pattern
   `<type>[optional scope]: <description>` (e.g. `feat: add user login`,
   `fix(api): handle empty payload`). Use a recognized type such as `feat`, `fix`,
   `docs`, `refactor`, `test`, `chore`, `build`, `ci`, or `perf`, and append `!`
   after the type/scope for a breaking change.
