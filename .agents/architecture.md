# Architecture Guidance

Read the root `ARCHITECTURE.md` and the relevant records under `docs/adr/`
before changing repository structure or architectural boundaries.

## Sources of truth

- Architecture decision records explain why consequential decisions were made
  and are authoritative for those decisions.
- `ARCHITECTURE.md` is the current high-level map of the repository and its
  intended boundaries.
- Source code and local READMEs contain implementation and operating details.

If the map conflicts with an accepted ADR, preserve the ADR and correct the
map. If a decision itself must change, add an ADR that supersedes the existing
record rather than silently changing the architecture document.

## Maintaining `ARCHITECTURE.md`

Keep the document short enough that recurring contributors can read it as an
orientation guide. Prefer information that changes infrequently.

The document should:

- Begin with a bird's-eye explanation of the system and its purpose.
- Provide a coarse codemap that answers where a concern belongs and what each
  major area does.
- Name important files, modules, types, and boundaries without reproducing
  their implementation documentation.
- Explicitly state architectural invariants, especially forbidden dependency
  directions and responsibilities that must remain separate.
- Describe important boundaries between processes, layers, platforms, and
  external systems.
- Summarize cross-cutting concerns that affect multiple areas.
- Distinguish the current implementation from intended architecture.

The document should not become:

- An exhaustive directory or file listing.
- A roadmap, issue tracker, or migration checklist.
- A substitute for API documentation, local READMEs, or inline comments.
- A duplicate of an ADR's rationale and alternatives.

Update `ARCHITECTURE.md` when a change moves a major responsibility, creates or
removes a boundary, changes a dependency direction, or makes its codemap
materially misleading. Ordinary implementation changes do not require an
architecture-document update.

## Review checklist

For architecture and repository-layout work, verify that:

- Physical directory placement matches the conceptual ownership described in
  the codemap.
- Portable code does not acquire platform-specific dependencies.
- New platform behavior crosses an explicit adapter or capability boundary.
- User-facing applications do not take ownership of protocol, transport,
  pairing, or credential state.
- Local READMEs remain beside the implementation-specific material they
  describe.
- Relevant changes are reflected in `ARCHITECTURE.md` without adding unstable
  detail.

## Further reading

- [ARCHITECTURE.md](https://matklad.github.io/2021/02/06/ARCHITECTURE.md.html),
  by matklad
