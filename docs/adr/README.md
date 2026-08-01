# Architecture decision records

This directory contains architecture decision records (ADRs) for decisions
that materially affect this fork's direction, structure, compatibility,
security, licensing, or supported platforms.

ADRs record why a decision was made as well as what was decided. They are not
implementation plans and should not be rewritten after acceptance merely
because the implementation evolves. If a decision changes, add a new ADR that
supersedes the previous one.

## Naming

Use a four-digit sequence followed by a short kebab-case title:

```text
0001-project-direction-and-cross-platform-architecture.md
0002-example-decision.md
```

Copy [`template.md`](template.md) when starting a new record. Use one of these
statuses:

- `Proposed`: under discussion and not yet authoritative.
- `Accepted`: the current project decision.
- `Rejected`: considered but not adopted.
- `Deprecated`: retained for history but no longer recommended.
- `Superseded by ADR-NNNN`: replaced by a later decision.

## Index

| ADR | Status | Decision |
| --- | --- | --- |
| [0001](0001-project-direction-and-cross-platform-architecture.md) | Accepted | Project direction and cross-platform architecture |
