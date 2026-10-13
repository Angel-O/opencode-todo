# Proposal

## Why

The todo-creation reminder currently infers a fresh list from every item being pending, so it misses creation with an in-progress item and can fire on ordinary updates. The user wants the reminder whenever a fresh list is created, including later tasks in the same session.

## What Changes

- Add optional `freshList: boolean` to `todowrite`; omission keeps normal write behavior. Callers explicitly mark subsequent fresh lists that replace a non-empty list.
- Emit boolean `listCreated` metadata on both todo tools. A successful non-empty write reports true when the active list was empty before the transaction or `freshList` is true. Updates, empty writes, and reads report false.
- Preserve existing item identity, timestamps, archival, ordering, and revision behavior, including when a fresh list carries unfinished items forward.
- Determine automatic creation within the existing write transaction. No persistent list identity or database migration is needed.
- Document the flag in the tool description and README; cover lifecycle behavior with tests.
- Coordinate a separate managed-config change: the first auto-instruct rule will use `dataEquals` on `listCreated: true`, scoped to `todowrite`. Auto-instruct source changes are unnecessary.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `todo-tools`: Explicit fresh-list input and creation signal in structured tool results.
- `todo-storage`: Transaction-derived creation signal in successful write results without changing persistence or item identity.

## Impact

- Source: `src/tools.ts`, `src/store.ts`, `src/metadata.ts`.
- Tests: `src/tools.test.ts`, `src/store.test.ts`; documentation: `README.md`.
- Existing tool calls remain valid; metadata is additive and database schema version stays unchanged. No UI changes, new dependencies, or agent instruction file edits.
- The managed auto-instruct template lives in a separate repository and will have its own review and git lifecycle; no external files are edited under this repo-local OpenSpec change.
- Installation of the updated plugin and application of configuration require separate per-action approval after verification. The reminder cannot match the new field until the updated plugin is running.

## Proposal Review Dispositions

Architect review: no blockers. All six concerns accepted:

1. Define active as all non-archived items and specify exact predicate precedence, including explicit false and empty writes.
2. Document operation-signal semantics: repeated explicit fresh writes signal creation; unflagged non-empty replacement does not.
3. Derive and return the signal inside the existing write transaction; test sequential same-session writes and rollback without claiming overlapping-writer guarantees.
4. Reject non-boolean input before mutation and preserve existing timestamp-aware store calls; reads emit false.
5. Add focused lifecycle scenarios and preserve existing identity/timestamp/archive/order/revision contracts without duplicating their existing tests.
6. Keep deployment separate and verify plugin metadata before applying the new matcher.
