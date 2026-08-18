# Joseph's Agent Skills

My personal fork of [Matt Pocock's skills](https://github.com/mattpocock/skills), updated and expanded to reflect how I want agents to work.

## Reference

These split on one axis — who can invoke them. **User-invoked** skills are reachable only when you type them; **model-invoked** skills can be invoked by you or reached for automatically by the agent when the task fits.

### Engineering

Skills I use daily for code work.

#### User-invoked

- **[ask-matt](./skills/engineering/ask-matt/SKILL.md)** — Ask which skill or flow fits your situation. A router over the user-invoked skills in this repo.
- **[agent-retrospective](./skills/engineering/agent-retrospective/SKILL.md)** — Audit agent-session history and turn recurring failures into evidence-backed instruction changes.
- **[building-verification](./skills/engineering/building-verification/SKILL.md)** — Build a project-local skill and harness that can safely prove the real product works.
- **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)** — Grilling session that also builds your project's domain model, sharpening terminology and updating `CONTEXT.md` and ADRs inline.
- **[triage](./skills/engineering/triage/SKILL.md)** — Move issues through a state machine of triage roles.
- **[improve-codebase-architecture](./skills/engineering/improve-codebase-architecture/SKILL.md)** — Scan a codebase for deepening opportunities, present them as a visual HTML report, then grill through whichever one you pick.
- **[setup-matt-pocock-skills](./skills/engineering/setup-matt-pocock-skills/SKILL.md)** — Configure this repo for the engineering skills (issue tracker, triage labels, domain doc layout). Run once per repo.
- **[to-spec](./skills/engineering/to-spec/SKILL.md)** — Turn the current conversation into a spec and publish it to the issue tracker.
- **[to-tickets](./skills/engineering/to-tickets/SKILL.md)** — Break any plan, spec, or conversation into a set of tracer-bullet tickets, each declaring its blocking edges — text in a local file, or native blocking links on a real tracker.
- **[implement](./skills/engineering/implement/SKILL.md)** — Build the work described by a spec or set of tickets, drive `/tdd` at pre-agreed seams, review it with `/code-review`, then require `/verifying-work` proof before committing.
- **[wayfinder](./skills/engineering/wayfinder/SKILL.md)** — Plan a huge chunk of work — more than one agent session can hold — as a shared map of decision tickets on the issue tracker, resolved one at a time until the way to the destination is clear.

#### Model-invoked

- **[prototype](./skills/engineering/prototype/SKILL.md)** — Build a throwaway prototype to answer a design question: a single shareable HTML file for state/logic, or several toggleable UI variations.
- **[file-pr](./skills/engineering/file-pr/SKILL.md)** — Prepare and file a concise pull request when the user asks to open, create, file, or draft one.
- **[babysit-pr](./skills/engineering/babysit-pr/SKILL.md)** — Monitor a pull request through review and CI without letting feedback expand its scope.
- **[diagnosing-bugs](./skills/engineering/diagnosing-bugs/SKILL.md)** — Disciplined diagnosis loop for hard bugs and performance regressions: build a feedback loop that goes red on this bug → minimise → hypothesise → instrument → fix → regression-test.
- **[research](./skills/engineering/research/SKILL.md)** — Investigate a question against high-trust primary sources and capture the findings as a cited Markdown file in the repo, run as a background agent.
- **[tdd](./skills/engineering/tdd/SKILL.md)** — Test-driven development with a red-green-refactor loop. Builds features or fixes bugs one vertical slice at a time.
- **[domain-modeling](./skills/engineering/domain-modeling/SKILL.md)** — Actively build and sharpen a project's domain model — challenge terms, stress-test with scenarios, update `CONTEXT.md` and ADRs inline.
- **[codebase-design](./skills/engineering/codebase-design/SKILL.md)** — Shared discipline and vocabulary for designing deep modules: small interfaces, clean seams, testable through the interface.
- **[logging-wide-events](./skills/engineering/logging-wide-events/SKILL.md)** — Design and review safe, correlated operational logging around one canonical wide event per unit of work.
- **[code-review](./skills/engineering/code-review/SKILL.md)** — Three-axis review of the diff since a fixed point: **Standards**, **Spec**, and an evidence-bound **Adversarial** attempt to break the change, run as parallel sub-agents.
- **[verifying-work](./skills/engineering/verifying-work/SKILL.md)** — Prove completed code changes through reproducible evidence from the real user-facing surface, with a fail-closed verdict.
- **[resolving-merge-conflicts](./skills/engineering/resolving-merge-conflicts/SKILL.md)** — Work through an in-progress git merge or rebase conflict hunk by hunk, resolving by intent traced to each side's primary source, then finish the operation — never `--abort`.
- **[wizard](./skills/engineering/wizard/SKILL.md)** — Generate an interactive bash wizard that walks a human through steps only they can perform: provisioning infrastructure, setting up credentials or CI secrets, walking an unfamiliar third-party dashboard, or running a one-off migration or cutover.

### Productivity

General workflow tools, not code-specific.

#### User-invoked

- **[grill-me](./skills/productivity/grill-me/SKILL.md)** — Get relentlessly interviewed about a plan or design until every branch of the design tree is resolved.
- **[handoff](./skills/productivity/handoff/SKILL.md)** — Compact a conversation into a document another agent can continue from.
- **[teach](./skills/productivity/teach/SKILL.md)** — Teach the user a new skill or concept over multiple sessions, using the current directory as a stateful teaching workspace.
- **[to-questionnaire](./skills/productivity/to-questionnaire/SKILL.md)** — Turn a decision you can't answer alone into a Markdown questionnaire for the one person who can — filled in async, or together over a meeting.
- **[wait-what](./skills/productivity/wait-what/SKILL.md)** — Fire this the moment a message doesn't land. The agent re-pitches it with the context you're missing, in plain English, using your `CONTEXT.md` vocabulary.

#### Model-invoked

- **[grilling](./skills/productivity/grilling/SKILL.md)** — Interview the user relentlessly about a plan, decision, or idea until every branch of the design tree is resolved.
- **[html-communication](./skills/productivity/html-communication/SKILL.md)** — Present plans, specs, findings, reports, comparisons, or UI mock sets as readable HTML artifacts.
- **[writing-for-agents](./skills/productivity/writing-for-agents/SKILL.md)** — Writing documents for agents: skills, AGENTS.md/CLAUDE.md, and any doc an agent reaches by a pointer.

## Attribution

- This repository is a personal fork of [Matt Pocock's skills](https://github.com/mattpocock/skills), adapted and extended for my own workflows and opinions.
- **[building-verification](./skills/engineering/building-verification/SKILL.md)** is adapted from [Lauren Tan's `create-verification-skill`](https://github.com/cursor/plugins/blob/main/pstack/skills/create-verification-skill/SKILL.md) under the MIT License.
- **[logging-wide-events](./skills/engineering/logging-wide-events/SKILL.md)** adapts the wide-event discipline from [Boris Tane's `logging-best-practices`](https://github.com/boristane/agent-skills/tree/main/skills/logging-best-practices).
