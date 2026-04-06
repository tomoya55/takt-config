# Design: Delivery Standards (Planner Persona + Policy)

## Problem

takt's `phased-delivery` workflow produces a single oversized PR (2,100+ lines) because:

1. The planner persona lacks guidance on phase decomposition standards
2. No policy enforces CI/CD setup, PR sizing, or merge-readiness
3. The coder has no constraints on what "one phase" means

## Solution

Enhance the planner persona and add a `delivery-standards` policy facet to encode delivery knowledge into the takt workflow.

## Changes

### 1. Enhanced `facets/personas/planner.md`

**Current state:** Generic "break into phases" instruction with ~500-line target.

**New additions:**

- **Phase 0 mandate:** Always emit a Phase 0 for CI/CD setup before any implementation phase. Phase 0 uses the PRD's tech stack to set up GitHub Actions (lint + test for each stack).
- **Phase decomposition rules:** Each phase must be independently deployable and testable. No phase should depend on a later phase. Each phase produces a working increment.
- **Output format:** Phases must be concrete enough to serve as `takt add` task descriptions. Each phase description includes: goal, files to create/modify, and acceptance criteria.
- **Size enforcement:** ~500 lines of diff per phase. If a natural boundary exceeds this, split further.

### 2. New `facets/policies/delivery-standards.md`

A policy applied to both planner and coder steps:

- **CI/CD first:** Phase 0 must set up GitHub Actions with lint + test for the project's tech stack. No implementation code in Phase 0.
- **PR size:** Each task/PR must stay under ~500 lines of diff. If approaching the limit, stop and mark the phase complete.
- **Independent phases:** Each phase must leave the codebase in a working state (tests pass, lint passes, app builds).
- **No draft PRs:** PRs must be created as ready-for-review, not draft.
- **Test coverage:** Each phase must include tests for the code it introduces.

### 3. Updated `workflows/phased-delivery.yaml`

- Add `policy: delivery-standards` to the `plan` step
- Add `policy: [coding, delivery-standards]` to the `implement` step (currently has no policy)
- Add `policy: [review, delivery-standards]` to the `review` step (currently has no policy)

## File Structure After Changes

```
takt-config/
├── facets/
│   ├── personas/
│   │   ├── planner.md          # UPDATED - phase decomposition knowledge
│   │   ├── coder.md            # unchanged
│   │   └── reviewer.md         # unchanged
│   └── policies/
│       └── delivery-standards.md  # NEW - CI/CD, PR size, merge standards
├── workflows/
│   └── phased-delivery.yaml    # UPDATED - references new policy
├── config.yaml
└── README.md
```

## Out of Scope

- Orchestrator script (decided against - use `takt add` + `takt run` directly)
- Auto-merge logic (handled by takt's `auto_pr` config + GitHub auto-merge)
- CD setup (CI only for now; CD can be added to Phase 0 later)
