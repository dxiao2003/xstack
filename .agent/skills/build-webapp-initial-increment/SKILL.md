---
name: build-webapp-initial-increment
description: Use when asked to turn a runnable greenfield or near-empty web app into its first usable product workflow, establishing the initial product architecture future features should follow.
---

# Build Webapp Initial Increment

## Philosophy

This skill builds the first usable product workflow in a web app that has little or no existing product functionality. It is not tied to any particular scaffold, generator, or template. Assume there is some runnable app foundation, then create the first durable conventions future features will inherit.

The skill's value is:

1. It narrows an early product idea into one end-to-end increment instead of sprawling into a whole MVP.
2. It establishes initial architecture deliberately, recording lasting decisions in `ARCHITECTURE.md`.
3. It turns assumptions into concrete, testable acceptance criteria before implementation.
4. It leaves the user with reviewable code, validation results, and an audit trail of decisions, risks, and tradeoffs.

## Operating Mode: Optimistic And Non-Interactive

Run unattended unless the user explicitly asks for checkpoints.

- **Proceed with reasonable defaults.** When there is enough product intent to infer a narrow first workflow, choose a conservative path and continue.
- **Log instead of asking.** Write ambiguities, assumptions, risks, and warnings to `WORKLOG.md` as they arise. Do not interrupt the run for ordinary uncertainty.
- **Constrain scope aggressively.** Build one coherent vertical slice. Defer adjacent features unless they are required for the requested workflow to work.
- **Create conventions only where needed.** Establish initial patterns for the first increment, but avoid broad architecture that is not exercised by the code being added.

### When You May Still Stop

Stop and report only after writing the current state to `WORKLOG.md` when continuing is impossible or destructive, for example:

- The prompt does not describe any product goal or user workflow, and no plausible first increment can be inferred.
- The repository is not a web app or has no runnable foundation to extend.
- The app is in a broken state that prevents meaningful code changes or validation.
- The user explicitly requires approval before continuing.

In other cases, choose the safest viable default, record it, and continue.

## WORKLOG.md Conventions

`WORKLOG.md` is the run log. It is not the lasting architecture record.

- **Location.** Create or append to `WORKLOG.md` at the repository root.
- **Never commit it.** Ensure `WORKLOG.md` is ignored. If it is not covered by `.gitignore`, add a `WORKLOG.md` entry. If it is already tracked, run `git rm --cached WORKLOG.md`, then add the ignore rule.
- **Structure per run.** Append a new top-level section headed with the product increment name and timestamp. Include the original request and one subsection per workflow step below.
- **Write as you go.** Update the log at the end of each workflow step.
- **Flag clearly.** Use `RISK:`, `WARNING:`, `ASSUMPTION:`, and `AMBIGUITY:` tags so the user can scan what needs attention.

## ARCHITECTURE.md Conventions

`ARCHITECTURE.md` is committed product architecture guidance. Create it if missing; update it if present. Keep it high-level, flexible, concise, and factual so future features can fit without constant rewrites.

Record only decisions future agents must understand to extend the app safely:

- Major runtime boundaries and app layers.
- Product domain boundaries and ownership.
- Cross-cutting conventions for data flow, APIs, persistence, validation, and errors.
- Testing, operational, security, and deferred-decision guidance that materially affects future work.

Prefer principles, extension points, and stable constraints over narrow rules. Do not document file-by-file structure, one-off implementation details, transient execution details, failed attempts, or choices ordinary feature work should be free to change; put run-specific caveats in `WORKLOG.md`.

## Workflow

Execute these steps in order. Write the result of each step to `WORKLOG.md` before moving on.

### 1. Orient To The App Foundation

- Confirm the working directory is a web app repository with a runnable foundation.
- Identify frameworks, languages, package managers, scripts, tests, routing, persistence, API shape, environment configuration, and build/run commands.
- Do not assume Docker, health checks, bootstrap decision docs, specific directories, or any particular scaffold source.
- Create and check out a descriptively named feature branch, such as `feature/<short-slug>`, before making product changes. If the working tree has uncommitted changes, log a `WARNING:` and branch from the current state anyway.
- Start the `WORKLOG.md` run section with the original request and a short stack summary.

### 2. Clarify The First Product Increment

- Convert the prompt into one narrow, end-to-end product workflow a user can actually perform.
- Define explicit non-goals so the increment does not expand into a full MVP.
- Prefer one primary screen or flow, one small domain concept, and the minimum backend or persistence needed to support it.
- If details are missing but the product intent is clear, choose reasonable defaults and log them as `ASSUMPTION:` entries.
- Stop only if no product goal or user workflow can be inferred.

### 3. Define Lasting Architecture Decisions

- Create or update `ARCHITECTURE.md` before or alongside implementation planning.
- Record the high-level conventions this increment establishes for future product work.
- Write decisions as adaptable guidance: concrete enough to orient later changes, but broad enough to accommodate ordinary new features without edits.
- If an architecture choice is provisional, mark it as such and state what future signal should revisit it.

### 4. Derive Concrete Acceptance Criteria

Turn the increment into high-level but testable criteria. Include:

- User-facing UI behavior and navigation.
- Data creation, reading, updating, deletion, or persistence behavior as applicable.
- API, server-action, or backend behavior as applicable.
- Empty, loading, validation, and error states.
- Important data rules, constraints, and security expectations.

Each criterion should have a clear pass/fail signal. Log underspecified criteria with `AMBIGUITY:` and the adopted `ASSUMPTION:`.

### 5. Design The First Vertical Slice

Design only the slice required by the acceptance criteria:

- Frontend route or screen, component boundaries, and state/data-loading path.
- Backend endpoint, server action, handler, service, or job path.
- Domain model, data access, migration, seed data, or local storage choice.
- Shared types, schemas, validation, and error contract.
- How the workflow becomes reachable from the existing app surface.

Prefer boring, local conventions that future increments can extend without rework.

### 6. Write Tests For The Acceptance Criteria

Use the repo's existing test frameworks and commands.

- Add unit tests for new isolated logic when meaningful.
- Add integration tests for the first domain/API/persistence boundary when the stack supports them.
- Add a browser-level or end-to-end test for the primary user workflow if e2e infrastructure already exists.
- Do not introduce a large new test stack just for this increment unless the repo has no practical validation path and the cost is justified.
- Map every acceptance criterion to at least one test or validation check. Log any uncovered criterion and why.

It is acceptable and expected for tests to fail before implementation.

### 7. Implement The First Product Surface

- Build the workflow through the real app surface so a user can reach and use it.
- Add or update frontend, backend, persistence, migration, configuration, and documentation files as needed.
- Follow the app foundation's existing framework conventions where they exist.
- Where no convention exists yet, implement the simplest durable pattern and record it in `ARCHITECTURE.md`.
- Keep generic operational endpoints or developer tooling when they remain useful.
- Do not weaken tests to make implementation pass. Change tests only when they were incorrect, and log why.

### 8. Validate, Summarize, Commit, And Open A PR

- Run the relevant test, lint, type-check, build, migration, and app validation commands discovered in step 1.
- Iterate until the new tests and affected existing checks pass, or log any remaining failure with a clear `RISK:` or `WARNING:`.
- Write a final **Summary** subsection to `WORKLOG.md` covering what was built, acceptance-criteria status, final validation results, files changed, and all flagged risks, warnings, assumptions, and ambiguities.
- Commit the product changes and `ARCHITECTURE.md`. Never commit `WORKLOG.md`.
- Push the branch and open a pull request by default. Use a Conventional Commits PR title, such as `feat: add initial product workflow`.
- Put the important summary, validation result, and flagged items in the PR body.

If committing, pushing, or opening a PR fails, log a `WARNING:`, leave the local branch in a reviewable state, and report the manual handoff steps.

## Notes

- Use this skill after a web app foundation exists, regardless of how that foundation was created.
- Use `build-webapp-feature` instead when the app already has meaningful product conventions and the task is a normal follow-on feature.
- Keep the first increment narrow. The best output is a real workflow with durable conventions, not a broad but shallow product shell.
