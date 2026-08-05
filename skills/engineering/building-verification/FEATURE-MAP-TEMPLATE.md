# Feature map template

Replace every bracketed field with repository-specific facts. Keep commands literal and feature descriptions user-facing.

## Index: `features/README.md`

```markdown
# [Product] verification map

This directory is the maintained source for verifying [product]'s user-facing behavior.

## Baseline preconditions

- [Exact launch state, isolated data, fixtures, authentication, and doctor result.]

## Driving conventions

- [Harness command prefix, stable-handle rules, reset behavior, and ordering constraints.]

## Proof and skip reporting

- [Required artifacts, side-effect checks, artifact location, and unreachable-path reporting.]

## Features

- [Feature name](./feature-name.md) — [user-visible scope].
```

## Recipe: `features/<feature>.md`

```markdown
# [Feature name]

[One paragraph describing the user-visible behavior.]

## Sub-features

- `[short-id]` — [one observable behavior].

## How to get to it (user POV)

- [Every known user entry point for this feature.]

## Driving it with [harness]

Preconditions:

- [Exact baseline state and successful doctor result.]

- **[User action].** Run `[literal command]`. Observe [visible result and load-bearing side effect]. Preserve [artifact or output].
- **Restore baseline.** Run `[literal cleanup or reset command]`. Observe [original state restored while proof remains].

## Gotchas

- [Timing, focus, state, selector, cleanup, or interpretation trap that could invalidate proof.]
```

Each recipe uses these four H2 headings in this order. Add action rows as needed, including alternate entry points and a read-only confirmation of mutations. An unreachable entry point records the attempted route and unmet prerequisite; proof from another route does not cover it.
