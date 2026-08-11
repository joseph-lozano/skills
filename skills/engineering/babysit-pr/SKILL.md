---
name: babysit-pr
description: Monitor a pull request through review and CI. Use when the user asks to watch, monitor, or babysit a PR.
---

# Babysit PR

Keep a pull request current and focused until it is ready for a human decision. Monitoring never grants permission to speak for the user, merge, close, or expand the PR's goal.

## 1. Snapshot the PR

Read the PR, its originating issue or spec, the current diff, the latest pushed commit and timestamp, checks, reviews, unresolved threads, and movement on the base branch. Record the original goal in one sentence; it is the scope boundary for every later decision.

Only treat checks and review comments newer than the latest push as current. Older findings may explain history, but do not act on them unless they still reproduce against the current head.

**Done when:** the head SHA, latest-push time, scope boundary, current checks, and current feedback are pinned.

## 2. Triage current feedback

For every current failure or finding:

1. Verify it against the source and current diff.
2. Classify it as a real defect, repository failure, infrastructure flake, stale finding, false positive, or out-of-scope suggestion.
3. Record the evidence and intended action.

Review bots are inputs, not authorities. Address real shortcomings and CI failures, but do not let review feedback turn the PR into a different project. Escalate ambiguous scope to the user.

## 3. Repair the branch narrowly

Make only changes required by the original goal or a verified regression introduced by the PR. Run focused checks before broad ones. Rebase or merge the base branch only when drift blocks the PR or repository policy requires it; warn before any force-push or destructive rewrite.

After each push, pin the new head SHA and restart the freshness boundary. Feedback attached to the previous head must reproduce before it drives another edit.

## 4. External communication boundary

A request to babysit a PR is not approval to communicate externally. Before posting a reply, resolving a thread, requesting review, changing labels, closing, merging, or performing another communicative action:

- present the exact proposed text and action;
- identify the PR and thread or target;
- wait for explicit approval in the current conversation.

Batch proposed replies when possible. Written dismissals of false positives should state the source-backed reason, not merely disagree with the reviewer. Once approved, format every reply written for Joseph as:

```md
[MODEL-SLUG] RESPONDING ON BEHALF OF JOSEPH

====
✨
[actual reply]
```

Use the real model slug; do not guess. Include screenshots or videos when they make the finding or fix easier to inspect, using only an approved upload target.

## 5. Continue or stop

Use a harness monitor when available. Otherwise poll at sensible intervals while the invocation remains active; do not busy-wait or claim background monitoring that the harness cannot provide.

Stop and report when one of these is true:

- ✅ **Ready:** required checks are green, required approvals are present, and no current actionable findings remain.
- ⛔ **Blocked:** access, infrastructure, ambiguity, or a human decision prevents progress.
- 🔁 **Superseded:** another merged or open PR makes this one obsolete.
- ⚠️ **Scope break:** the next requested change falls outside the original goal.

Report the current head SHA, checks, review state, actions taken, proposed communications awaiting approval, and the next human decision. Never merge or close unless that exact action was explicitly authorized.
