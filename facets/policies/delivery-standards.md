# Delivery Standards Policy

Standards for producing reviewable, mergeable, CI-validated increments.

## CI/CD First

- Phase 0 sets up CI (lint + test) before any implementation phase
- Every subsequent phase must pass CI before it is considered complete
- If CI does not exist yet, creating it is the first task — no exceptions

## PR Size

- Each phase targets ~500 lines of diff
- If a phase approaches this limit, stop and mark it complete. Remaining work becomes the next phase
- A PR with 1,000+ lines of diff is a defect in planning, not a sign of productivity

## Independent Phases

Each phase must leave the codebase in a working state:
- All tests pass
- Lint passes
- The application builds successfully
- No broken imports or dead references from incomplete work

## No Draft PRs

PRs are created as ready-for-review. Draft PRs indicate incomplete work that should not have been pushed.

## Test Coverage

Every phase includes tests for the code it introduces:
- New endpoints get API tests
- New services get unit tests
- New UI components get component tests (when applicable)
- Tests must actually run and pass, not just exist

## Commit Hygiene

- One logical change per commit
- Commit messages follow conventional format: `feat:`, `fix:`, `test:`, `ci:`, `chore:`
- Do not bundle unrelated changes
