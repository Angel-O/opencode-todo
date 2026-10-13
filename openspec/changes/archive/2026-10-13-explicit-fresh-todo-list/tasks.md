# Tasks

## 1. Transaction-bound write signal

- [x] 1.1 Add scenario-named store tests for sequential creation, fresh-intent timestamp/identity preservation, invalid intent, and rollback; observe the relevant new assertions fail before implementing the signal.
- [x] 1.2 Add optional fourth boolean fresh-intent argument preserving the third timestamp argument, validate before mutation, and return transaction-derived `listCreated`; verify the new tests and existing store/housekeeping contract tests pass without a database schema change.

## 2. Tool contract and caller documentation

- [x] 2.1 Add or update scenario-named tool tests for metadata shape/read false, invalid flag, mixed-status initial creation, ordinary updates, carryover, repeated explicit fresh intent, empty writes, and recreation after clearing; observe each new behavior assertion fail before its tool implementation.
- [x] 2.2 Extend the tool schema/runtime validation with optional `freshList`, forward the committed creation signal through shared metadata, and keep reads false; verify focused tool tests and existing plugin wiring tests pass.
- [x] 2.3 Document the flag, operation-signal limitations, carryover IDs, and boolean metadata in the tool description and README; verify examples match the accepted schema and all-pending updates are not described as creation.

## 3. Integration verification

- [x] 3.0 Bootstrap and audit the missing `.secrets.baseline` required by the existing hook (scope extension approved separately); scan found zero findings, with no exclusions or suppressed findings added. Baseline generation is a one-off tooling artifact, so behavioral TDD does not apply.
- [x] 3.1 Run repository test, lint, and build scripts; confirm all pass and no generated build artifacts or unrelated source changes enter the diff. Formatting-only hunks in touched files are required by the existing Prettier hook; the separately approved secret-scanning baseline is intentional.
- [x] 3.2 Verify the emitted metadata is consumable by the existing auto-instruct `dataEquals` condition for creation but not updates or reads, using the approved rule shape without editing auto-instruct source or external config.
- [x] 3.3 Perform simplification self-review and source review against proposal, design, and delta specs; resolve blockers and disposition warnings before commit and archival. Code review approved with no blockers or warnings; architectural conformance review found none. Accepted suggestions: explain required formatting in the commit body and point test spec comments at stable main-spec paths applied during archival.
