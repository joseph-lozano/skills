---
name: building-verification
description: Build a project-local skill and harness when a repository lacks a safe way to prove the real product works.
disable-model-invocation: true
---

# Building Verification

Build a project-local verifier for the next agent to use cold, mid-task. The output is a `verifying-<product>` skill, a user-facing feature map, and only the helper code needed to launch, drive, observe, isolate, and clean up the real product.

This skill creates verification infrastructure. `/verifying-work` uses that infrastructure to prove a particular change.

## 1. Interview the repository

Read the repository before asking the user. Ground these five answers in existing commands, code, and documentation:

- **Surface** — what users touch: web UI, CLI or TUI, desktop or mobile app, API, library, or more than one of these.
- **Run** — how the real product builds and starts, including readiness, ports, environment, authentication, fixtures, and seed data.
- **Drive** — how an agent can control each surface. Prefer the project's existing browser, E2E, PTY, HTTP, or consumer harness.
- **Observe** — which visible outcomes and load-bearing side effects can be captured as evidence.
- **Isolate** — which ports, data directories, browser profiles, accounts, queues, and external systems a verification run could share or corrupt.

Run the narrowest existing build-only check while interviewing. Start the product only after drafting a provisional isolated launch and cleanup plan. Even an exploratory launch needs a unique run ID, isolated resources, an ownership record, and guaranteed cleanup; otherwise return `BLOCKED`. A broken baseline is also `BLOCKED`: report the failure instead of writing commands against an imagined working product. Ask the user only about choices the repository cannot answer, especially unsafe external effects or multiple equally primary surfaces.

Identify every agent harness that must consume the verifier. Choose its project-local root from the repository's existing convention; common roots are `.agents/skills/` and `.claude/skills/`. For multiple harnesses, use one canonical directory and link it into every discovery root when the repository permits symlinks. If it does not, ask the user to choose a single harness or approve mirrored copies rather than silently creating divergent sources.

**Done when:** each answer cites an observed command, path, or runtime result; the primary surface, target harnesses, and canonical skill root are settled; and every shared resource is accounted for.

## 2. Design the verifier

Create `verifying-<product>/SKILL.md` with valid frontmatter and a model-facing description that names the product, surfaces, and verification situations. The generated verifier is model-invoked: omit `disable-model-invocation` and omit `policy.allow_implicit_invocation: false`. Add `agents/openai.yaml` when the chosen skill system supports it, then install or link the canonical directory into every selected harness's discovery root.

The generated skill must contain exact, repository-specific instructions under these headings:

- **Launch** — build and start commands, unique run ID, isolated resources, readiness signal, and resource ownership record. Short-lived CLIs launch once per drive rather than pretending to be a server.
- **Doctor** — one read-only check that proves the expected build or revision is ready, attached to the intended ports, data, profile, account, queue, project, and external endpoints, authenticated where needed, and safe to drive. A generic healthy response does not prove instance identity. Doctor fails closed when identity or ownership is uncertain.
- **Drive** — literal commands and stable handles from this product. Prefer ARIA roles, accessible labels, data attributes, command names, prompts, and routes over coordinates or tab order.
- **Evidence** — exact artifact location and what records the action, resulting user-visible state, and load-bearing side effects. Preserve commands, stdout, stderr, exit codes, traces, screenshots, logs, or read-only state queries as appropriate; redact secrets and personal data.
- **Cleanup** — remove scratch state and stop only resources owned by this run. Before acting, revalidate immutable ownership such as process start identity plus run marker, container ID or label, or a marker inside a run-owned directory. A stale PID, port, or path is not ownership proof. Refuse uncertain cleanup and report the exact manual action instead. Cleanup runs after failed attempts and preserves evidence.
- **Feature map** — where the maintained index and feature recipes live, and how to select the affected paths.

Prefer direct existing commands. Add a helper only when it makes the procedure safer or replayable; make it executable, document every invocation in the generated skill, and keep it inside the generated skill unless the repository already owns that responsibility elsewhere.

Read [PROOF-PATTERNS.md](PROOF-PATTERNS.md) when matching the product's surfaces to drive and evidence choices.

**Done when:** an agent with no prior context can run the verifier without guessing, and unsafe shared instances produce a refusal rather than a best effort.

## 3. Seed the feature map

Use [FEATURE-MAP-TEMPLATE.md](FEATURE-MAP-TEMPLATE.md) to create an index and one recipe for each of the three to five most important user-facing features identifiable from routes, commands, menus, public APIs, or documentation. Cover all known user entry points for each selected feature; do not claim the initial map covers features it does not list.

Describe user actions and observable results, not implementation internals. Pair every action with an exact harness command and expected observation. Record prerequisites, stable handles, invisible side effects, and gotchas that can invalidate a run.

**Done when:** the index names every seeded recipe, and each recipe can be followed from a known baseline to preserved proof without consulting product source.

## 4. Implement the smallest safe harness

Reuse production entry points and existing test or control infrastructure first. Add only the missing launch, doctor, drive, evidence, or cleanup seam.

- Assign every run a unique identity and record every resource it owns.
- Make readiness state-based, not a fixed sleep.
- Use mocks only where production already defines the external boundary.
- Observe what a dry-run or test mode actually skips; its name is not evidence.
- Verify side effects through an independent read path where possible.
- Put transient run state and proof in ignored, repository-approved locations. Cleanup removes run state and retains proof.
- Ask before changing product behavior solely to make it observable.

**Done when:** doctor is read-only, drives are deterministic enough to replay, concurrent runs are isolated or explicitly refused, and cleanup cannot target resources the verifier did not start.

## 5. Prove the generated verifier

Execute its own cold-start instructions end to end:

1. Confirm a fresh agent context or the harness's skill-list command can discover `verifying-<product>` by name.
2. From that cold context, launch the real product in an isolated run.
3. Run doctor and confirm the expected identity.
4. Drive one mapped feature through its real user entry point.
5. Capture the action, result, and load-bearing side effect.
6. Calibrate each new load-bearing checker with a safe known-negative control that makes it fail for the intended reason. Preserve its exact setup and command, expected failure, actual output and exit status, and why that output is the intended failure rather than an incidental error.
7. Restore and rerun the positive baseline after calibration.
8. Run cleanup, including after every failed iteration.
9. Confirm owned resources are gone and both positive and negative evidence still exist at the documented path.

Use a controlled wrong build ID, absent fixture, false expected value, or other reversible negative control. Do not create a destructive failure merely to prove a checker can fail.

A verifier that has not completed this loop is `BLOCKED`, not ready.

**Done when:** every new load-bearing checker has preserved, replayable positive and negative calibration; teardown leaves no owned runtime state; and a cold agent can discover and use the verifier.

## 6. Hand off

Return exactly one outcome:

- **READY** — link the generated skill and feature map; name every discovery root; state the launch, doctor, and cleanup commands; identify the feature proved and its positive and negative calibration artifacts; state concurrency limits and unseeded surfaces.
- **BLOCKED** — identify the failed step, exact command and output, resources cleaned up, and the smallest next action. Leave no half-documented verifier presented as usable.

---

Adapted from Lauren Tan's `create-verification-skill` in Cursor's `pstack`. See [LICENSE](LICENSE) for the MIT license and attribution.
