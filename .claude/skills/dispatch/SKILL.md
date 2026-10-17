---
name: dispatch
description: Apply labels to queue issues for implementing runs. Reads the Ready column (plus answered In Progress cards), fills missing attribute labels from flags, adds `claude` last, moves the card to In Progress. Skips and reports anything already running, waiting for input, an epic, blocked, or with an open PR. Runs for seconds and exits.
---

# /dispatch

> **Skeleton.** This file records the contract from the Issue-to-PR Loop design so the skill can be written against it. Replace the step bodies with real instructions before first use.

## Usage

```
/dispatch [issues] [--model sonnet|opus|fable] [--effort low|medium|high|xhigh|max] [--advisor opus|fable|off] [--risk high] [--limit N]
```

`issues` as `12 13 14` or `12-15`; omitted means the queue in order. Flags **fill gaps, never overwrite**: an issue already labeled `model:opus` keeps it under `--model sonnet`. `--limit` is sized to what the owner will review, never to the usage limit; until the test suite has substance, `--limit 1`.

## The queue

Found through `.claude/project.json`: every card in **Ready**, plus any **In Progress** card that carries neither `claude` nor `needs-input` (an issue the owner answered or rewrote after a run, whose card nothing can move back). In column order.

## For each issue, up to the limit

Skip and report, with the reason, when the issue:

- already has `claude` (a run is queued or in progress);
- has `needs-input` (a run asked or left a draft; answering means removing that label);
- has `epic` (a `/plan` parent, whatever column it sits in);
- has an open blocker (`gh issue view N --json blockedBy`), so "dependent issues need a merge between them" is enforced by the queue, not memory;
- has an open PR authored by the app (`gh pr list --author <app>[bot] --state open`, head matching `claude/issue-N-`), whatever its column: re-dispatching would close a PR under review as superseded.

Otherwise:

1. Add the attribute labels from the flags that the issue doesn't already carry (`gh issue edit N --add-label …`).
2. Add `claude` **last**, as its own call: the build workflow fires on that label and reads every label on the issue at that instant.
3. Move the card to In Progress (`gh project item-edit`, ids from `.claude/project.json`). This needs project permission on the token `gh` holds, which a cloud session may lack: if it fails, the labels still went on; report "dispatched #N; card did not move: <reason>" in the same words `/plan` uses.

## Report

One line per issue: dispatched (with the labels now on it), or skipped and why. Then exit; this skill does not watch runs.
