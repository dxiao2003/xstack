---
name: build-webapp-feature
description: Use when asked to build a new feature into an existing web app from a natural-language description. Runs non-interactively end to end on a new feature branch — vets the request against the current codebase, derives concrete testable acceptance criteria, writes unit/integration/e2e tests, implements the feature, and iterates until tests pass. Logs risks, warnings, ambiguities, and a final summary to a git-ignored WORKLOG.md, then commits and opens a pull request by default.
---

# Build Webapp Feature

## Philosophy

This skill builds a feature into an **existing** web app from a single user prompt, with no human in the loop during execution. The whole point is to run unattended to completion and hand the user a reviewable record, not to pause for confirmation.

The skill's value is:

1. It refuses to build blindly — it first checks the request against the real code so it does not duplicate existing behavior or break important functionality.
2. It turns a fuzzy request into concrete, testable acceptance criteria before writing code.
3. It writes the tests first, implements against them, and iterates until they pass.
4. It records every meaningful decision, risk, and warning in `WORKLOG.md` so the user can audit what happened after the fact instead of being interrupted during it.

## Operating Mode: Optimistic And Non-Interactive

This is the default contract. Read it before doing anything else.

- **Proceed optimistically.** Whenever a reasonable default exists, take it and keep going. Do not stop to ask the user to choose between options, confirm intent, or resolve ambiguity.
- **Log instead of asking.** Every time you would normally ask a question, raise a concern, or warn about a risk, write it to `WORKLOG.md` and continue with your best-judgment choice. The user reviews `WORKLOG.md` after the run.
- **Record the path not taken.** When you pick a default in the face of ambiguity, log both the ambiguity and the assumption you made, so the user can correct it later.
- **Exceptions override this mode.** If the user's prompt contains explicit instructions to stop, ask, or get approval at a specific point, honor those exactly. Such explicit exceptions take precedence over optimistic auto-proceed. Absent an explicit exception, never stop.

### When You May Still Stop

Auto-proceeding does not mean ignoring genuine blockers. Stop and report (after writing the current state to `WORKLOG.md`) only when continuing is impossible or destructive, for example:

- The request fundamentally conflicts with or would destroy existing critical functionality, and no non-destructive implementation path exists.
- The repository is not a web app, or the working tree is in a broken state that prevents building or testing at all.
- An explicit exception in the user's prompt directs you to stop.

In every other case, choose a default, log the reasoning, and continue.

## WORKLOG.md Conventions

`WORKLOG.md` is the durable record of this run. It is the deliverable the user reviews.

- **Location.** Create or append to `WORKLOG.md` at the repository root.
- **Never commit it.** `WORKLOG.md` must not be checked into git. Before writing to it, ensure it is ignored: if `WORKLOG.md` is not already covered by `.gitignore`, add a `WORKLOG.md` line to `.gitignore`. If it was already tracked, run `git rm --cached WORKLOG.md` so it stops being tracked, then add the ignore rule. Do not stage or commit `WORKLOG.md` itself in any commit you create.
- **Structure per run.** Append a new top-level section for this feature, headed with the feature name and a timestamp, so repeated runs accumulate rather than overwrite. Within it, keep one subsection per workflow step below (Vetting, Acceptance Criteria, Tests, Implementation, Summary).
- **Write as you go.** Update `WORKLOG.md` at the end of each step, not all at once at the end. If the run is interrupted, the log should already reflect everything done so far.
- **Flag clearly.** Mark items that need user attention with an explicit tag such as `RISK:`, `WARNING:`, `ASSUMPTION:`, or `AMBIGUITY:` so they are easy to scan.

## Workflow

Execute these steps in order. Write the results of each step to `WORKLOG.md` before moving on.

### 1. Orient

- Confirm the working directory is an existing web app repository. Identify the frameworks, languages, package managers, test runners, and how the app is built and run. Look for project skills, scripts, or docs that describe how to test and run the app.
- **Create a feature branch.** Do not work directly on the default branch. Create and check out a new descriptively named branch for this feature (e.g. `feature/<short-slug>`) before making any changes. If the working tree already has uncommitted changes, log a `WARNING:` and branch from the current state anyway.
- Start the `WORKLOG.md` section for this feature with the original request verbatim and a short note on the stack you detected.
- Ensure `WORKLOG.md` is git-ignored per the conventions above.

### 2. Vet The Request Against The Code

Before designing anything, sanity-check that the request makes sense against the current code. Investigate, then catalog findings in `WORKLOG.md` under a **Vetting** subsection.

Check for at least:

- **Duplication.** Does the feature, or a close equivalent, already exist? Search for existing routes, components, endpoints, models, or utilities that already do this. If a partial version exists, plan to extend it rather than rebuild it, and log that decision.
- **Conflicts / breakage.** Would implementing this break or regress important existing functionality (shared components, public APIs, data models, auth, critical flows)? Identify the blast radius.
- **Feasibility.** Are the required dependencies, data, or integration points present? Note anything missing.

Log each discovered risk with a `RISK:` tag, its severity, and how you intend to mitigate or work around it. Per the operating mode, do not stop for risks unless one is a genuine blocker (see "When You May Still Stop"). Proceed with the safest viable approach and record it.

### 3. Derive Concrete Acceptance Criteria

Turn the request into a set of high-level but concrete, testable acceptance criteria. Record them in `WORKLOG.md` under an **Acceptance Criteria** subsection.

- If the request already states acceptance criteria, verify each is concrete enough to test (observable behavior, specific inputs/outputs, clear pass/fail). Tighten any that are vague.
- If it does not, derive them from the request with best effort. Prefer user-observable behavior over implementation detail.
- Each criterion should be phrased so a test can unambiguously confirm it.
- Flag anything underspecified with an `AMBIGUITY:` tag and state the assumption you are adopting to proceed (`ASSUMPTION:`). Do not stop — choose the most reasonable interpretation and continue.

### 4. Write Tests For The Acceptance Criteria

Implement tests that exercise and confirm the criteria from step 3, using the repo's existing test framework and conventions. Summarize what you wrote in `WORKLOG.md` under a **Tests** subsection (which criteria each test covers, file paths, test types).

- **Unit tests** for the new logic in isolation.
- **Integration tests** that exercise the feature through its real seams (API + persistence, component + state, module boundaries).
- **End-to-end tests** when the repo supports them — e.g. a headless browser runner such as Playwright or Cypress is configured. If e2e infrastructure exists, add an e2e test for the primary user-facing flow. If it does not exist, do not stand up a new e2e stack from scratch; log a `WARNING:` that e2e coverage was skipped and why.
- Map every acceptance criterion to at least one test. Note in the log any criterion you could not cover and why.
- It is expected and correct that these tests fail before the feature is implemented.

### 5. Implement The Feature And Make Tests Pass

Implement the feature following the existing code's structure, patterns, and conventions. Then run the tests and iterate until they pass.

- Prefer extending existing code over duplicating it, consistent with the vetting findings.
- **Use installed testing skills/tooling.** If the user has skills or project scripts that govern how tests are run (e.g. a `run`, `verify`, or project-specific test skill), use those rather than ad hoc commands. Detect and prefer the repo's canonical test/lint/build commands.
- Run the full relevant test suite — the new tests plus any existing tests in the affected area — to catch regressions. Iterate on the implementation (not by weakening tests) until they pass.
- If a test reveals that an acceptance criterion or assumption was wrong, adjust the implementation and log the correction. Only change a test if it was itself incorrect, and log that change with justification.
- Record progress in `WORKLOG.md` under an **Implementation** subsection: what was built, files touched, test runs, and the final test result. Log any test that remains failing with a `RISK:`/`WARNING:` and the reason.

### 6. Summarize, Commit, And Open A PR

Write a final **Summary** subsection to `WORKLOG.md`. The summary must let the user review the run without re-reading everything:

- What was built, and the final state of the acceptance criteria (which are met, which are not).
- Final test results (counts, pass/fail, anything skipped).
- A consolidated list of all `RISK:`, `WARNING:`, `ASSUMPTION:`, and `AMBIGUITY:` items raised during the run, so the user has one place to review what needs their attention.
- Files added/changed and any follow-up work recommended.

Then, by default, ship the work for review:

- **Commit** the feature changes on the feature branch created in step 1. Never stage or commit `WORKLOG.md` (it is git-ignored). Use a clear commit message describing the feature.
- **Push** the branch to the remote.
- **Open a pull request** against the default branch. Make the PR body a concise version of the `WORKLOG.md` summary — what was built, acceptance-criteria status, test results, and the consolidated list of flagged risks/warnings/assumptions/ambiguities — so a reviewer sees the caveats without reading the log.
- Report the PR URL to the user.

This is the default behavior so the run ends with reviewable work, consistent with the non-interactive operating mode. The user's prompt may override it with an explicit exception (e.g. "don't open a PR" or "just commit locally"); honor such instructions. If committing or pushing fails (e.g. no remote or missing credentials), log a `WARNING:`, leave the commit on the local branch, and report what the user needs to do to finish pushing.

## Notes

- This skill assumes an existing project. To create a new web app from scratch, use the bootstrap skill instead.
- Keep `WORKLOG.md` factual and scannable; it is a log, not prose. The tags are what make it auditable.
