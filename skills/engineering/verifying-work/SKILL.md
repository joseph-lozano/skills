---
name: verifying-work
description: Verifies completed code changes through reproducible evidence from the real user-facing surface. Use after implementing a feature, fixing a bug, completing a migration or refactor, or before declaring code work done.
---

# Verifying Work

Verification is a **replayable proof** that the finished change satisfies its contract. A green build supports the proof; it does not replace observing the behaviour the user asked for.

This skill verifies. It does not implement the fix or lower the bar after a failure. `/tdd` owns the red-green implementation loop, `/code-review` owns static review against the spec and standards, and `/diagnosing-bugs` owns root-cause investigation.

## 1. Pin the claims

Read the user request, spec or ticket, every acceptance criterion, the diff, and the repository instructions. Pin the subject with a commit SHA, or with the base SHA plus a content fingerprint or immutable snapshot of every tracked and untracked product change. Keep verification-only artifacts outside that fingerprint.

If repository instructions define a **surface matrix**, walk every row and record `applies` or `does not apply` with a reason. Every applicable client, entry point, adapter, contract, reversible state, documentation surface, or deployment mode must map to a claim or an explicit acceptance-criterion exclusion; an undecided row is missing coverage.

Rewrite each required outcome as a **falsifiable claim**:

> Given <condition>, when <action>, then <observable result and threshold>.

Include the affected user entry points and load-bearing side effects. Do not invent requirements to fill gaps. If the desired result is too ambiguous to observe, ask the user to sharpen it before continuing.

**Done when:** every acceptance criterion, affected user path, and applicable surface-matrix row maps to a falsifiable claim, with out-of-scope behaviour kept out.

## 2. Design the proof

Locate and load the matching project-local `verifying-*` skill before planning. Use its doctor, drive, evidence, cleanup, and feature map as the repository-specific verification contract. If an expected verifier exists on disk but the active harness cannot discover it, return `INCONCLUSIVE`.

Every behavioural claim starts at the affected user-facing entry point and observes the result plus its load-bearing side effects. Make that path deterministic and replayable where possible, using:

1. Existing automation that drives the real interface: browser interaction, API request, CLI invocation, or library consumer.
2. A baseline/treatment comparison using the same command, inputs, and environment.
3. A focused checker or harness that can be rerun.
4. A structured human check when automation cannot reach the surface.

Build, typecheck, lint, and unit tests are supporting evidence unless they cross the claimed surface. Expected results must come from the contract, a known-good fixture, or a captured baseline—not from the implementation being checked. For migrations, refactors, and performance work, capture the baseline before judging the treatment. Calibrate every new or modified checker against a known-negative fixture, baseline, or control that proves it can fail; an uncalibrated checker is `INCONCLUSIVE`.

Treat tools as drivers, not proof. "The browser test passed" is weak; "submitted checkout through the browser, observed the success state, one payment request, and one persisted order, with the trace preserved" names the action and observations. Apply the same pattern to APIs (response plus downstream state), CLIs (invocation, streams, exit code, and changed state), libraries (a public-API consumer), migrations (baseline versus treatment), and performance work (repeated measurements against a threshold).

When the project has no safe, replayable launch, doctor, drive, observe, isolate, and cleanup path, return `INCONCLUSIVE` and recommend that the user run `/building-verification`. Do not improvise against an unidentified or shared instance.

Write a proof plan with one row per claim:

| Claim | Surface | Action or command | Expected observation | Evidence |
| --- | --- | --- | --- | --- |

**Done when:** every required claim has a runnable check and a defined artifact or captured output. A required claim with no viable check is `INCONCLUSIVE`, not a pass.

## 3. Execute independently

When the harness exposes subagents, dispatch verification to a fresh one. Give it the authoritative request, spec or ticket, acceptance criteria, repository instructions, subject identifier, and proof plan—not the implementer's reasoning alone. It must challenge claim coverage as well as execute the plan. Prefer a different model family when one is available; self-review does not count as independent review.

When no independent context exists, same-context verification can establish `VERIFIED` only through deterministic checks whose outputs mechanically decide every claim. Evidence requiring agent interpretation is `INCONCLUSIVE`.

For each claim:

1. Confirm the environment is healthy and isolated enough for the result to mean anything.
2. Drive the real user path. Test-only endpoints and internal setters prove only themselves.
3. Capture the action and resulting state, including side effects—not only the final screen or exit code.
4. Record exact commands, inputs, outputs, and artifact paths. Redact secrets and personal data.
5. Repeat measurements where noise or flakiness could change the verdict.

Then run the repository's required quality checks.

A decisive failure stops verification. Capture it and report it to the implementer; do not repair product code inside the verification pass. After any fix, rerun the failed check and every check whose result the changed code could affect.

**Done when:** either a decisive failure has current evidence, or every planned check has current evidence tied to the subject identifier and an independent verifier has challenged the claims and proof when the harness supports one.

## 4. Issue the verdict

Use exactly one verdict:

- **VERIFIED** — every required claim and repository check passed with reproducible evidence.
- **NOT VERIFIED** — at least one required claim or repository check failed. A known failure takes precedence over checks not yet run.
- **INCONCLUSIVE** — no decisive failure was established, but evidence is missing, noisy, confounded, blocked on a human or environment, or does not cover every required path.

Changing the verified code state invalidates the verdict wherever the change could affect the proof. `INCONCLUSIVE` is a stop, never a soft pass.

## 5. Hand back the proof

Publish the proof where the work lives—a ticket or PR when available, otherwise the conversation. Link artifacts instead of paraphrasing them. Keep sensitive evidence local and say where it is.

```markdown
## Verification: VERIFIED | NOT VERIFIED | INCONCLUSIVE

**Subject:** commit SHA, or base SHA plus product-state fingerprint/snapshot
**Verifier:** project-local skill name and canonical path

| Claim | Method | Result | Evidence |
| --- | --- | --- | --- |
| ... | ... | PASS / FAIL / BLOCKED | command output, screenshot, trace, or artifact path |

**Repository checks:** exact commands and results
**Calibration:** negative-control commands, expected failures, and preserved outputs for new or modified checkers
**Independent review:** verifier/model used, or why unavailable
**Not covered:** explicit gaps and residual risks
**Next action:** ready for review, return to implementation, or unblock the proof
```

The proof is complete when a reviewer can inspect or rerun the load-bearing checks without trusting the agent's summary.
