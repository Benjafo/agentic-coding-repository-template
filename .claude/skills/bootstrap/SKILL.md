---
name: bootstrap
description: Once per project, interactively, on Fable. Interview the owner, write docs/SPEC.md, plan in plan mode, scaffold the stack with CI wired and one tested example of each pattern, fill CLAUDE.md and docs/ARCHITECTURE.md, open the bootstrap PR, then file the probe issue. Never unattended.
---

# /bootstrap

> **Skeleton.** This file records the contract from the Issue-to-PR Loop design so the skill can be written against it. Replace the step bodies with real instructions before first use.

## Preconditions (check before anything else, refuse and say what to fix)

1. The model is Fable (`claude --model fable` or `/model fable` first). Frontmatter can't set it for a conversation.
2. `.claude/project.json` exists in the clone: `setup.sh` has run. If not, stop and say so.
3. `development` is the default branch and is protected: this skill works on a `bootstrap` branch and opens a PR; a direct push to `development` must fail.
4. `gh auth status` succeeds with the owner's account.

## Interview

Ask, one topic at a time, and don't proceed until each is answered:

- What the app is and for whom, in plain English, as to a new hire.
- Runtime target: browser, Node, or both.
- Framework and test runner preference, or none (the template is JS/TS-family; a stack outside it does not fit).
- Persistence: none, SQLite, Postgres, other; migrations and seed if so.
- Auth: none, or which approach.
- Preview host, and whether it deploys branches whose commits are authored by the owner's GitHub App.
- Which areas are `risk:high` by nature (auth, payments, deletion, migrations, plus anything domain-specific).
- Whether there is a UI the runner can render (decides screenshot capture).

## Produce, in order

1. **docs/SPEC.md** from the answers. Show it and get approval before anything else.
2. **Plan** in plan mode: stack, layout, the list of patterns docs/ARCHITECTURE.md will name (route, component, data access, whatever the stack has), what CI needs (DB service, env vars, migrate, seed). The owner approves.
3. **Scaffold** on branch `bootstrap`: the framework, the test runner wired into `package.json` scripts (`lint`, `typecheck`, `test`, `test:coverage` emitting `coverage/lcov.info`, `coverage:patch`, `build`, `dev`), and **one real vertical slice through every pattern, each with a test** named `AC-n: …`, so a run has something to copy. Delete the placeholder `src/example.js` and `test/example.test.js`. Keep `test/lint-rules.test.js`.
4. **CI wiring**: the runner's lcov output path, `PATCH_COVERAGE_INCLUDE` set to the real source roots, any DB service/env/migrate/seed steps in `ci.yml` and in the implementer jobs (`claude-issues.yml`, `claude.yml`) but **not** in `claude-review.yml`. Add the dev-server, migrate, seed, and screenshot commands to `.claude/settings.json` `permissions.allow` by name.
5. **Screenshot capture**, when there is a renderable UI: a headless browser used as a tool in the run (not an e2e suite), writing to `screenshots/issue-N/`. Record in `CLAUDE.md` whether screenshots are expected or "no renderable UI".
6. **CLAUDE.md** (concise: conventions that can only be prose, the screenshot decision, the commands) and **docs/ARCHITECTURE.md** (a real map: every layer, where each pattern's example lives). First `.claude/rules/*.md` for path-scoped conventions.
7. **Open the bootstrap PR** against `development`. It gets CI (first run on the skeleton) and no Claude review (the review workflow skips head `bootstrap`); the owner reads it.
8. After the owner merges: **file the probe issue** in the issue form's shape (Summary, Why, Acceptance criteria AC-1…, Out of scope, How to verify), small and real, and tell the owner to add `claude`. Bootstrap is not done until the pipeline has produced its first PR.

## Not this skill's job

Labels, branch protection, secrets, the project board (`setup.sh`). Feature planning (`/plan`). Dispatch (`/dispatch`).
