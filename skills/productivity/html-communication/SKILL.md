---
name: html-communication
description: Present a plan, spec, findings, report, comparison, or static UI mock set as a readable HTML artifact. Use when the user asks for HTML, a visual write-up, or side-by-side options.
---

# HTML Communication

Create a readable artifact for a human outside the terminal. This skill communicates work; it does not add HTML to the product.

## Choose the right artifact

Use this skill for a plan, spec, findings, report, comparison, or static set of UI mocks. If the question requires the user to exercise behaviour, push a state model through cases, or judge UI inside the real product context, run `/prototype` instead.

## Build one stable file

Write one self-contained HTML file with inline CSS and JavaScript and no required network dependencies. Put it in the OS temp directory unless the user asks for it in the repository. Choose a descriptive stable path for the task and overwrite that same file on revisions so the link or path does not move.

Resolve the temp directory from `$TMPDIR`, then `/tmp` on Unix-like systems or `%TEMP%` on Windows. Do not overwrite an unrelated existing artifact.

## Write it like a document

Optimize for comprehension rather than landing-page aesthetics:

- lead with the decision, question, or executive summary;
- make headings and navigation reveal the information hierarchy;
- keep prose selectable and important evidence inspectable;
- use tables for aligned facts and diagrams only when relationships are graph-shaped;
- make the layout responsive and keyboard-readable;
- include source links or file references beside the claims they support.

For comparisons or UI mocks, label options `A`, `B`, `C`, and so on. Put them in one file, make direct comparison easy, and keep the labels stable across revisions. Distinguish structural alternatives rather than presenting cosmetic variations as separate options.

## Hand it over

Open the file with the platform command (`open`, `xdg-open`, or `start`) and return its absolute path. Do not claim it opened successfully unless the command succeeded. Browser-driven visual verification, screenshots, and uploads are extra work; do them only when requested.

Uploading publishes an external artifact. Before uploading, show the exact file and destination, confirm that the host is trusted and credentials are present, and obtain approval for that publication. Never invent an upload target or expose secrets and private source material.

On revision, update the same file, reopen it, and summarize what changed.
