---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

Use /tdd where possible, at pre-agreed seams.

Run typechecking and focused test files regularly, and the full test suite once at the end.

Once done, use /code-review to review the work.

Then use /verifying-work for the final repository checks and to prove the finished change through its real user-facing surface. Commit only a `VERIFIED` result; `NOT VERIFIED` returns to implementation, and `INCONCLUSIVE` stops for the missing evidence or access.

Commit your work to the current branch.
