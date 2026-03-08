# Changelog

## v3.0.0

### Breaking Changes

- **`/iterate` renamed to `/implement`.** The single-ticket implementation workflow is now
  `/implement`. If you were using `/iterate`, use `/implement` instead.

- **`/batch` renamed to `/implement-batch`.** The multi-ticket orchestration workflow is now
  `/implement-batch`. If you were using `/batch`, use `/implement-batch` instead.

- **`/project` renamed to `/implement-project`.** The full-lifecycle project workflow is now
  `/implement-project`. If you were using `/project`, use `/implement-project` instead.

The skill directories have been renamed accordingly (`skills/iterate/` → `skills/implement/`,
etc.). Any local references to the old skill names in scripts or documentation should be updated.

## v2.0.0

### Breaking Changes

- **`/implement-project` renamed to `/implement-batch`.** The v1.x `/implement-project` skill (single-batch
  ticket orchestration) is now `/implement-batch`. The `/implement-project` name is used by a new,
  higher-level workflow (see below). If you were using `/implement-project` to implement
  a single batch of tickets, use `/implement-batch` instead.

### New Skills

- **`/implement-project` — Full-lifecycle project workflow.** Orchestrates an entire
  multi-batch project: implements batches via `/implement-batch`, runs smoke tests, then
  executes a comprehensive quality pipeline (refactor, arch-review,
  test-review, doc-review, release-review). Maximizes autonomy with andon cord
  escalation.

- **`/scope-project` — Adversarial project planning.** Plans a multi-batch
  project through adversarial review. Drafts tickets organized into batches,
  then pits a planner against an implementer agent to find gaps and
  ambiguities. Produces tagged tickets ready for `/implement-project` consumption.

### Improvements

- Rewrote top-level README to present skills as a cohesive layered system
  rather than a flat list
- Added `make release` target for tagging and publishing releases

## v1.1.0

Initial tagged release.
