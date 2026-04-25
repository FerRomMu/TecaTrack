# Versioning Strategy

TecaTrack follows **Semantic Versioning (SemVer)**: `MAJOR.MINOR.PATCH`

This repository is part of the **TecaTrack ecosystem**, composed of multiple repositories
(Frontend, Backend, and Documentation).  
Versioning is managed with a **shared ecosystem version** to keep all repositories aligned.

---

## Versioning Rules

### MAJOR (`X.0.0`)

Incremented when changes introduce **breaking compatibility** across the ecosystem.

Examples:

- Backend API breaking changes that require frontend modifications.
- Database schema changes that require major migrations or redesign.
- Architectural refactors that invalidate previous documentation or flows.

---

### MINOR (`0.Y.0`)

Incremented when new features or improvements are added in a **backwards-compatible** way.

Examples:

- Adding new endpoints while keeping existing ones unchanged.
- Adding new modules or workflows.
- Extending documentation with new architecture components or design decisions.

---

### PATCH (`0.0.Z`)

Incremented when changes are **backwards-compatible fixes** or small updates.

Examples:

- Bugfixes.
- Small documentation corrections or clarifications.
- Minor refactors with no external impact.

---

## Cross-Repository Consistency

All repositories within the TecaTrack ecosystem must share the **same MAJOR.MINOR** version.

Meaning:

- `MAJOR.MINOR` represents the ecosystem compatibility level.
- `PATCH` represents repo-specific adjustments.

### Example

- Frontend: `1.2.3`
- Backend: `1.2.1`
- Documentation: `1.2.5`

All repositories are compatible because they share `1.2`.

---

## Release Guidelines

A new release should be created whenever:

- A meaningful milestone is reached.
- A stable set of features is completed.
- Documentation is updated to reflect an architectural or functional change.

Each release must update:

- `CHANGELOG.md`
- Version tag in the repository (`git tag vX.Y.Z`)

---

## Tagging Convention

Git tags must follow:
`MAJOR.MINOR.PATCH`

Example:
`v1.0.0`
`v1.1.0`
`v1.1.2`

---

## Notes

This repository is documentation-focused, therefore PATCH increments may occur more frequently
than in code repositories, without impacting compatibility.
