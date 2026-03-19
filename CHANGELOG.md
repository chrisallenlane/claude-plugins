# Changelog

## v4.4.0

### New Skills

- **`/review-perf` — Performance review.** Detects whether a project contains web content, non-web source code, or both, and dispatches the appropriate performance reviewer(s) in parallel. Produces a consolidated report with cross-domain synthesis. Advisory only.

### New Agents

- **`swe-web-perf-reviewer`** — Web performance reviewer. Identifies network, caching, loading, and asset delivery issues from source code analysis. Covers caching strategy, asset delivery, critical rendering path, resource hints, image optimization, JS/CSS cost, network overhead, and Core Web Vitals risk factors. Advisory only.

### Improvements

- **Agent naming standardized.** All advisory "examine and report" agents now use the `reviewer` suffix. Test-related QA agents grouped under `qa-test-*` namespace. Web-specific agents namespaced with `web`. Full rename list:
  - `swe-review-arch` → `swe-arch-reviewer`
  - `swe-refactor` → `swe-code-reviewer`
  - `swe-diagnostician` → `swe-bug-investigator`
  - `swe-perf-engineer` → `swe-perf-reviewer` (now advisory only)
  - `qa-release-eng` → `qa-release-engineer`
  - `qa-accessibility-auditor` → `qa-web-a11y-reviewer`
  - `qa-test-auditor` → `qa-test-reviewer`
  - `qa-coverage-analyst` → `qa-test-coverage-reviewer`
  - `qa-fuzz-analyst` → `qa-test-fuzz-reviewer`
- **`swe-perf-reviewer` is now advisory only.** Formerly `swe-perf-engineer` with implementation authority. Now follows the same advisory pattern as all other reviewers, routing findings to language SMEs.
- **Versioning policy documented.** Skills (slash commands) are the public interface. Agent names are internal implementation details and may change without a major version bump.

## v4.3.0

### New Skills

- **`/review-a11y` — WCAG accessibility audit.** Advisory-only skill that dispatches an accessibility auditor to identify WCAG conformance gaps, prioritize findings by user impact, and recommend fixes. No changes are made — the output is an assessment report.

### Improvements

- **`/audit-source` now defaults to production-only scope.** Test code, dev-only dependencies, generated code, and vendored code are excluded by default. Users can override these exclusions during scope selection.
- **`/audit-source` now reports all severity levels.** LOW findings are no longer suppressed when CRITICAL or HIGH findings exist. All priority levels (CRITICAL, HIGH, MEDIUM, LOW) now get dedicated focused red-teamer agents, with a target cap increased from 10 to 25.

## v4.2.0

### New Skills

- **`/review-source` — Code health assessment.** Advisory-only skill that detects all languages in a project, dispatches language-specific SME agents (or generalists for unsupported languages) to evaluate idiomatic usage, consistency, and quality, and produces a consolidated health report with per-language ratings. Use to decide whether `/refactor` is needed.

## v4.1.0

### New Skills

- **`/audit-source` — White-box security audit.** Orchestrates a comprehensive security assessment using both defensive and offensive analysis. The blue-teamer evaluates defensive posture first; the lead red-teamer performs reconnaissance informed by those gaps; focused red-teamers investigate each vector in depth; findings are synthesized and exploit chains explored until no new chains emerge.

### New Agents

- **`sec-red-teamer`** — Adversarial security analyst. Attacks the codebase from an attacker's perspective to find concrete exploitable vulnerabilities. Works as the offensive counterpart to `sec-blue-teamer`.
- **`sec-blue-teamer`** — Defensive security analyst (renamed from `sec-reviewer`). Evaluates security posture through control inventory, consistency checking, defense-in-depth assessment, and dependency hygiene. Runs as the first step in `/audit-source`, feeding its defense evaluation to the red team.

### Improvements

- Both `sec-blue-teamer` and `sec-red-teamer` report incidental non-security bugs discovered during their analysis in a dedicated `NON-SECURITY BUGS` section.
- **`/scope`** and **`/scope-project`** ticket templates now include a **Security considerations** field, prompting authors to note new attack surface, input handling, auth/authz changes, and trust boundary impacts.

## v4.0.0

### Breaking Changes

- **Five skills and agents renamed for verb-noun consistency:**
  - `/arch-review` → `/review-arch` (agent `swe-arch-review` → `swe-review-arch`)
  - `/doc-review` → `/review-doc`
  - `/release-review` → `/review-release`
  - `/test-review` → `/review-test`
  - `/test-mutate` → `/test-mutation`

  Update any invocations or scripts that reference the old names.

### New Agents

- **`swe-sme-html`** — HTML structure, semantics, and accessibility specialist. Dispatched by `/implement`, `/refactor`, `/review-arch`, and `/bugfix` for web projects.
- **`swe-sme-css`** — CSS styling, layout, and responsive design specialist. Covers Flexbox, Grid, custom properties, and modern CSS features.
- **`swe-sme-javascript`** — Vanilla JavaScript implementation specialist (ES modules, async/await, DOM APIs). Defers to TypeScript SME when the project uses TypeScript.
- **`swe-sme-typescript`** — TypeScript implementation specialist. Covers strict-mode configuration, type design, generics, and compiler discipline.
- **`qa-accessibility-auditor`** — WCAG 2.2 AA accessibility auditor. Advisory-only role: identifies barriers, prioritizes by user impact, and provides remediation guidance for HTML/CSS/JS implementers.

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
  executes a comprehensive quality pipeline (refactor, review-arch,
  review-test, review-doc, review-release). Maximizes autonomy with andon cord
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
