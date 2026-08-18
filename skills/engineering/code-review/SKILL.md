---
name: code-review
description: Review a branch, PR, or work-in-progress diff against repository standards, its originating spec, and adversarial failure cases. Use when the user asks to review changes since a commit, branch, tag, or merge-base.
---

Three-axis review of the diff between `HEAD` and a fixed point the user supplies:

- **Standards** — does the code conform to this repo's documented coding standards?
- **Spec** — does the code faithfully implement the originating issue / spec?
- **Adversarial** — how can the change be made to fail, corrupt state, leak access, or falsely appear correct?

The axes run as **parallel sub-agents** so they don't pollute each other's context, then this skill aggregates their findings.

The issue tracker should have been provided to you — run `/setup-matt-pocock-skills` if `docs/agents/issue-tracker.md` is missing.

## Process

### 1. Pin the fixed point

Whatever the user said is the fixed point — a commit SHA, branch name, tag, `main`, `HEAD~5`, etc. If they didn't specify one, ask for it.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the list of commits via `git log <fixed-point>..HEAD --oneline`.

Before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty. A bad ref or empty diff should fail here — not inside the parallel sub-agents.

### 2. Identify the spec source

Look for the originating spec, in this order:

1. Issue references in the commit messages (`#123`, `Closes #45`, GitLab `!67`, etc.) — fetch via the workflow in `docs/agents/issue-tracker.md`.
2. A path the user passed as an argument.
3. A spec file under `docs/`, `specs/`, or `.scratch/` matching the branch name or feature.
4. If nothing is found, ask the user where the spec is. If they say there isn't one, the **Spec** sub-agent will skip and report "no spec available".

### 3. Identify the standards sources

Anything in the repo that documents how code should be written, such as `CODING_STANDARDS.md` or `CONTRIBUTING.md`.

On top of whatever the repo documents, the Standards axis always carries the **smell baseline** below — a fixed set of Fowler code smells (_Refactoring_, ch.3) that applies even when a repo documents nothing. Two rules bind it:

- **The repo overrides.** A documented repo standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Each smell is a labelled heuristic ("possible Feature Envy"), never a hard violation — and, like any standard here, skip anything tooling already enforces.

Each smell reads *what it is* → *how to fix*; match it against the diff:

- **Mysterious Name** — a function, variable, or type whose name doesn't reveal what it does or holds. → rename it; if no honest name comes, the design's murky.
- **Duplicated Code** — the same logic shape appears in more than one hunk or file in the change. → extract the shared shape, call it from both.
- **Feature Envy** — a method that reaches into another object's data more than its own. → move the method onto the data it envies.
- **Data Clumps** — the same few fields or params keep travelling together (a type wanting to be born). → bundle them into one type, pass that.
- **Primitive Obsession** — a primitive or string standing in for a domain concept that deserves its own type. → give the concept its own small type.
- **Repeated Switches** — the same `switch`/`if`-cascade on the same type recurs across the change. → replace with polymorphism, or one map both sites share.
- **Shotgun Surgery** — one logical change forces scattered edits across many files in the diff. → gather what changes together into one module.
- **Divergent Change** — one file or module is edited for several unrelated reasons. → split so each module changes for one reason.
- **Speculative Generality** — abstraction, parameters, or hooks added for needs the spec doesn't have. → delete it; inline back until a real need shows.
- **Message Chains** — long `a.b().c().d()` navigation the caller shouldn't depend on. → hide the walk behind one method on the first object.
- **Middle Man** — a class or function that mostly just delegates onward. → cut it, call the real target direct.
- **Refused Bequest** — a subclass or implementer that ignores or overrides most of what it inherits. → drop the inheritance, use composition.

### 4. Set the adversarial posture

Treat the change as hostile until the evidence earns confidence. Author-supplied claims are untrusted input: commit messages, comments, names, types, tests, screenshots, and happy-path demonstrations may all be mistaken or crafted to hide a defect. Do not grant correctness for plausible intent, clean presentation, passing tests, or familiarity with the author.

Be adversarial toward the **artifact**, never abusive toward the person. Do not speculate about the author's actual motives or competence, and do not invent findings to satisfy the posture. Every finding needs a concrete failure path, violated invariant, or evidence gap whose consequence can be stated precisely. Try to disprove correctness; report what survives that attempt honestly.

The Adversarial axis must:

1. Map every changed trust boundary and side effect: callers, inputs, authorization, persistence, network or process interaction, concurrency, configuration, and externally visible output. Read beyond the diff wherever a caller, callee, schema, or deployment contract determines whether the changed hunk is safe.
2. Trace unhappy paths: malformed and boundary inputs; missing or hostile identity; partial failure, timeout, retry, and cancellation; races and reordered events; duplicate delivery and non-idempotent replay; stale or mixed-version state; migration and rollback; resource exhaustion; and irreversible data loss. Apply only cases relevant to this change, but account for every changed boundary.
3. Audit the tests as skeptically as the product code. Look for assertions that never prove the claim, mocks that remove the failure mode, fixtures that avoid dangerous values, false-positive test setup, untested negative paths, and tests that exercise a different path from production.
4. Run focused tests or construct a minimal reproduction when static reading cannot settle a plausible high-impact failure. Review is read-only: do not repair the change while reviewing it.
5. For each finding, give severity, confidence, file and line, trigger, exact consequence, evidence, and the smallest credible fix or regression test. Drop a concern if you cannot make it concrete after tracing it.

### 5. Spawn all sub-agents in parallel

**Standards sub-agent prompt** — include:

- The full diff command and commit list.
- The list of standards-source files you found in step 3, **plus the smell baseline from step 3** pasted in full — the sub-agent has no other access to it.
- The brief: "Report — per file/hunk where relevant — (a) every place the diff violates a documented standard: cite the standard (file + the rule); and (b) any baseline smell you spot: name it and quote the hunk. Distinguish hard violations from judgement calls — documented-standard breaches can be hard, but baseline smells are always judgement calls, and a documented repo standard overrides the baseline. Skip anything tooling enforces. Under 400 words."

**Spec sub-agent prompt** — include:

- The diff command and commit list.
- The path or fetched contents of the spec.
- The brief: "Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that wasn't asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong. Quote the spec line for each finding. Under 400 words."

If the spec is missing, skip the Spec sub-agent and note this in the final report.

**Adversarial sub-agent prompt** — include:

- The diff command and commit list.
- The originating spec when available, repository instructions, and commands for focused tests.
- The adversarial posture and five obligations from step 4 pasted in full — the sub-agent has no other access to them.
- The brief: "Attempt to falsify the change's correctness. Inspect relevant code beyond the diff and run focused checks when needed. Report only concrete findings, ordered by severity. Each finding must include severity, confidence, file:line, trigger, consequence, evidence, and the smallest credible fix or regression test. A passing test suite is evidence, not absolution. If no issue survives investigation, say `No adversarial findings` and list the boundaries and failure classes examined. Under 600 words."

### 6. Aggregate

Present the reports under `## Standards`, `## Spec`, and `## Adversarial` headings, verbatim or lightly cleaned. Do **not** merge or rerank findings — the three axes are deliberately separate (see _Why three axes_). "No adversarial findings" means the reviewer found no concrete defect in the boundaries it examined, not that the change is certified safe.

End with a one-line summary: total findings per axis, and the worst issue _within each axis_ (if any). Don't pick a single winner across axes — that's the reranking the separation exists to prevent.

## Why three axes

A change can pass one axis and fail another:

- Code that follows every standard but implements the wrong thing → **Standards pass, Spec fail.**
- Code that does exactly what the issue asked but breaks the project's conventions → **Spec pass, Standards fail.**
- Code that follows the standards and spec but fails under retry, hostile input, rollback, or concurrency → **Standards and Spec pass, Adversarial fail.**

Reporting them separately stops one axis from masking another.
