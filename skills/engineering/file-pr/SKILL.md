---
name: file-pr
description: Prepare or file a pull request. Use when the user asks to open, create, file, or draft a PR.
---

# File PR

Prepare a concise pull request whose title and description explain why the change matters. Filing is an external communication: draft the exact content first and wait for approval.

## 1. Pin the change

Identify the repository, head branch, intended base branch, and originating request, issue, or spec. Before drafting:

- check whether a PR already exists for the head branch;
- review the complete diff against the merge-base;
- confirm the diff matches the original goal and call out unrelated changes;
- inspect recently merged PRs and repository guidance for title and body conventions;
- collect current verification evidence without inventing results.

If an existing PR covers the branch, report it and ask whether the user wants to update it instead of creating another.

**Done when:** the exact diff, goal, repository conventions, and verification state are known.

## 2. Draft for the reviewer

Titles often become commit messages. Prefer a concise, human-readable title that communicates the outcome or impact, while following repository conventions.

- ❌ **Bad title:** `perf(server): negotiate permessage-deflate on the websocket`
- ✅ **Good title:** `perf(server): cut websocket frame size by 70%+ with gzipping`

Open the body with the problem in language from the user's original request. Then explain the solution briefly. Do not lead with an inventory of files, functions, or implementation steps.

- ❌ **Bad opening:** “Removed implicit workspace carry-over from every new-thread entry point. Deleted the contextual thread-option helpers and seed-context machinery.”
- ✅ **Good opening:** “My ‘new worktree’ default was ignored when starting new threads on existing worktrees. Now your preferences always apply.”

Include verification evidence and any meaningful residual risk. Use the repository's template when one exists; otherwise use only the headings the content earns. Do not include secrets, private logs, or unsupported claims.

**Done when:** a reviewer can understand the problem, outcome, verification, and risk without first reading the diff.

## 3. Approval boundary

Present all external content exactly as it will appear:

- repository, base, and head;
- draft or ready-for-review state;
- complete title;
- complete body.

Then stop for explicit approval of that exact content. A request to “file a PR” authorizes preparation, not publication. If the user already supplied and approved the exact title and body in the current conversation, do not ask twice.

## 4. File after approval

Immediately before filing, recheck that the branch, diff, and existing-PR state have not changed. If the approved content is stale, revise it and obtain approval again.

Create a ready-for-review PR unless the user approved a draft. Do not add comments, reviewers, labels, auto-merge, or other external messages/actions that were not included in the approval.

Return the PR URL and the exact checks that remain. Monitoring is a separate concern: run `/babysit-pr` only when the user also asks to watch, monitor, or babysit the PR.
