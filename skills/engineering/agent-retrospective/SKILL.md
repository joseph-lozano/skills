---
name: agent-retrospective
description: Audit agent-session history and turn recurring failures into evidence-backed instruction changes.
disable-model-invocation: true
---

# Agent Retrospective

Improve agent behaviour from observed friction, not borrowed prompt folklore. The output is an evidence-backed change proposal; editing starts only after the user chooses what to adopt.

## 1. Scope the sample

Agree on:

- repositories or working directories;
- harnesses and models;
- history locations;
- time range or number of sessions;
- any sensitive projects or content to exclude.

Discover history read-only. Do not scan unrelated machines, accounts, or directories merely because they are accessible. Record the task mix for each model and harness so difficult work assigned to one model is not mistaken for a model effect.

**Done when:** every history source and exclusion is explicit, and the sample can be reproduced.

## 2. Build the evidence set

Look for observable outcomes, especially:

- explicit user corrections and reversals;
- abandoned attempts or repeated commands;
- tool errors and unsafe or destructive actions;
- unrequested edits and scope growth;
- overbuilding or unnecessary delegation;
- premature completion and missing verification;
- unclear explanations or unusable artifacts.

Count both sessions/tasks and user messages where the source permits it. Rates such as corrections per 100 user messages are useful comparisons, not causal proof. Redact secrets, personal data, and irrelevant source content from excerpts.

Produce an evidence table:

| Failure | Model / harness | Tasks affected | Rate denominator | Cost | Representative evidence |
| --- | --- | ---: | --- | --- | --- |

**Done when:** every reported pattern has a count, denominator, and at least one inspectable example.

## 3. Diagnose representative failures

Read representative sessions from each high-cost category. Separate the visible symptom from the likely decision point: stale instructions, missing project context, ambiguous user intent, poor tool affordances, weak code seams, or model behaviour.

Challenge confounders:

- Was this model assigned harder or different work?
- Was the failure specific to one repository or tool?
- Did the user correct style, correctness, process, or all three?
- Did a later message reveal context unavailable at the decision point?
- Is the supposed fix already present but failing to trigger?

A single bad run is a reason to investigate, not yet a global rule.

**Done when:** each recommended intervention is tied to a plausible, evidence-supported decision point and its confidence is stated.

## 4. Choose the narrowest lever

Prefer the narrowest durable correction:

1. **Environment or tooling** — make the safe command obvious, encode a check, or remove a footgun.
2. **Code or module design** — improve the seam when instructions are compensating for hard-to-change code.
3. **Project instructions** — project invariants, vocabulary, surface matrices, and local hazards.
4. **Global instructions** — stable personal preferences that genuinely apply everywhere.
5. **Existing skill** — repair its trigger, steps, or completion criterion.
6. **New skill** — only for a reusable process with an independent invocation condition.

Do not solve a discoverable command, transient model quirk, or one-off preference by permanently expanding global context.

## 5. Propose before editing

Rank proposals by expected benefit, evidence strength, and context or cognitive load. Show the user:

| Priority | Evidence | Proposed lever | Exact behavioural change | Scope | Evaluation |
| --- | --- | --- | --- | --- | --- |

For subjective behaviour, include the smallest concrete bad/good pair that distinguishes the preference. State what existing text would be replaced or removed; additions without pruning create sediment.

Present the complete proposal and stop. Do not edit instructions or skills until the user selects the exact changes.

## 6. Apply selected changes

For approved proposals, run the `/writing-for-agents` skill. Preserve invocation boundaries, update every repository index and router required by local instructions, and keep one source of truth for each rule.

## 7. Evaluate and revisit

Define a small evaluation set using future comparable tasks or safe replays with external side effects disabled. Check:

- whether the right document triggers;
- whether the target failure falls;
- whether a neighbouring workflow regresses;
- whether the new instruction remains relevant across models.

Report observations as evidence, not proof of causality. Set a review point after enough comparable sessions, then keep, revise, or remove the change.
