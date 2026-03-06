# Changelog

## v2.0.0

### Breaking Changes

- **`/project` renamed to `/batch`.** The v1.x `/project` skill (single-batch
  ticket orchestration) is now `/batch`. The `/project` name is used by a new,
  higher-level workflow (see below). If you were using `/project` to implement
  a single batch of tickets, use `/batch` instead.

### New Skills

- **`/project` — Full-lifecycle project workflow.** Orchestrates an entire
  multi-batch project: implements batches via `/batch`, runs smoke tests, then
  executes a comprehensive quality pipeline (refactor, arch-review,
  test-review, doc-review, release-review). Maximizes autonomy with andon cord
  escalation.

- **`/scope-project` — Adversarial project planning.** Plans a multi-batch
  project through adversarial review. Drafts tickets organized into batches,
  then pits a planner against an implementer agent to find gaps and
  ambiguities. Produces tagged tickets ready for `/project` consumption.

### Improvements

- Rewrote top-level README to present skills as a cohesive layered system
  rather than a flat list
- Added `make release` target for tagging and publishing releases

## v1.1.0

Initial tagged release.
