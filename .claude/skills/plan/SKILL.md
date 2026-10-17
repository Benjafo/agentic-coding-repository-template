---
name: plan
description: Turn a plain-English feature description into an approved, ordered breakdown of issues in the project's Backlog, as sub-issues of an epic parent with blocked-by set. Runs on Opus (high). Creates nothing until the owner approves the breakdown.
---

# /plan

> **Skeleton.** This file records the contract from the Issue-to-PR Loop design so the skill can be written against it. Replace the step bodies with real instructions before first use.

## Preconditions (check first, refuse and say what to fix)

1. The model is Opus (`/model opus` first; the override from frontmatter lasts one turn and this is a conversation).
2. `gh --version` is 2.94.0 or later (`--parent`, `--blocked-by` on `gh issue create`).
3. `.claude/project.json` exists (project node id, Status field id, option ids for Backlog / Ready / In Progress / Done).

## Read

`SPEC.md`, `ARCHITECTURE.md`, the open issues (`gh issue list --state open --json number,title,labels`), and the feature description the owner gave.

## Ask the clarifying questions now

So the implementing run doesn't have to. Keep asking until a breakdown can be written without guessing.

## Propose a breakdown

Ordered, vertical slices, one PR each, independent where possible. For each issue: title; Given/When/Then criteria numbered AC-1, AC-2, …; the area of the codebase; dependencies on other issues in the breakdown; suggested `risk:high`; a size. The owner edits or approves in the conversation. **Nothing is created before approval.**

## Create, after approval

1. The parent feature issue, labeled `epic`. It holds the sub-issues, not work: `/dispatch` skips it and the owner closes it when the last sub-issue merges.
2. Each sub-issue in dependency order (every blocker exists before it is named) with `gh issue create --parent <epic> --blocked-by <m,n> --label risk:high` (label only where suggested). `gh` cannot fill a YAML issue form, so write the body in the form's section shape, with the same headings: **Summary**, **Why**, **Acceptance criteria**, **Out of scope**, **How to verify in the preview**, **Dependencies**.
3. Add each to the project and set Status = Backlog with `gh project item-add` and `gh project item-edit`, ids from `.claude/project.json`. If the project call fails for lack of permission, the issue still exists: report it the way `/dispatch` does ("created #N; card not added: <reason>").

Never add `claude`, never promote to Ready: creating an issue spends nothing; the owner promotes.

## Report

The epic number, each sub-issue number with its blockers, and anything that didn't land.
