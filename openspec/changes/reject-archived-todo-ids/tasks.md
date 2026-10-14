# Tasks

## 1. Atomic storage validation

- [x] 1.1 Add name-matched storage regression tests for late stale-ID mixed-batch rejection (ordinary and fresh writes), rejection with an empty active list, and acceptance of unknown/other-session archived IDs; observe rejection tests failing before implementation and verify all stored rows and revision are unchanged on failure.
- [x] 1.2 Validate all supplied IDs against same-session archived history inside the existing write transaction before mutations, preserving active matching; verify new regressions and existing identity/order/revision tests pass.

## 2. Tool recovery contract

- [x] 2.1 Add a name-matched tool regression for the actionable archived-ID error, unchanged list/revision, no successful result, and corrected ordinary retry with `listCreated: false`; observe it failing before the storage fix and verify it passes afterward through existing error propagation.
- [x] 2.2 Document rejection and recovery in README's `todowrite` guidance, including reading current IDs or omitting stale IDs and retaining the full intended list; verify documentation matches the tested contract.

## 3. Integration verification

- [x] 3.1 Run project test, lint, and build scripts; verify all succeed without schema, dependency, hook, or UI changes.
- [x] 3.2 Self-review for simplification and obtain code review; verify every blocker is resolved and each warning has a recorded disposition, with scenario-to-test coverage and design conformance checked.

Verification: 76-test baseline passed; rejection regressions failed before implementation; final 80 tests, project lint/build, and scoped hooks pass. Acceptance-boundary and existing successful-write clarification tests preserve behavior, so no artificial red step applies to those unchanged cases. Self-review found no new duplication, speculative abstraction, or redundant comments. Code review approved with no blockers or warnings. Optional suggestions: keep main-spec test references and verify archival; no additional housekeeping note because rejection already covers all same-session archived rows; retain prepared-query style consistent with existing writes.

## Workflow follow-up

- Commit implementation and archive the change with its spec updates in a separate commit.
- Push the branch, open the PR, verify remote status, and remove the isolated worktree.
- Installation or a service restart requires separate explicit authorization.
