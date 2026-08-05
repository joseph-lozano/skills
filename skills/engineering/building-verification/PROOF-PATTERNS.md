# Proof patterns

The harness drives the product; the observation proves the claim. Prefer an existing project harness over the example tool category.

| Surface | Drive through | Capture as proof |
| --- | --- | --- |
| Web UI | Existing browser or E2E harness using roles and labels | User action, resulting UI, relevant network or console state, persisted effect, trace or screenshot |
| API or service | Real public request against an isolated instance | Request, status and body, downstream state, relevant logs |
| CLI or TUI | Built or installed artifact through a shell or PTY | Invocation, stdout, stderr, exit code, changed files or persisted state |
| Library | Minimal consumer through the public API | Consumer input and output, errors, and externally visible side effects |
| Migration or refactor | Baseline and treatment with the same representative inputs | Comparable outputs and state, plus intentional differences |
| Performance | Repeated comparable benchmark in a pinned environment | Raw samples, threshold, variance, and baseline comparison |

Tool completion alone is supporting evidence:

- **Weak:** "The browser test passed."
- **Strong:** "Submitted checkout through the browser; observed the success state, one payment request, and one persisted order; preserved the trace and screenshot."

For every surface, include an independent read path for invisible side effects when one exists. A final screenshot, success message, or zero exit code does not by itself prove persistence or downstream effects.
