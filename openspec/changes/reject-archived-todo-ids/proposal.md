# Proposal

## Why

Reusing IDs from an archived todo list can silently empty the active list, causing a subsequent recovery write to report creation and fire the creation reminder again. Reject stale IDs without losing the current list so agents can correct the input safely.

## What Changes

- Reject a write containing an ID belonging to an archived item in the same session, before changing any rows or revision.
- Surface an actionable error identifying the stale ID and advising use of a current item ID or omission of the ID.
- Preserve acceptance of active IDs, genuinely new IDs, and IDs archived only in another session; `freshList` does not bypass validation.
- Add regression tests and document recovery from the error.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `todo-storage`: Distinguish new IDs from archived same-session IDs and reject stale-ID writes atomically.
- `todo-tools`: Surface stale-ID failures without a successful result or mutation of the current list.

## Impact

Implementation is limited to `src/store.ts`, with regression tests in `src/store.test.ts` and `src/tools.test.ts`, plus `README.md` and OpenSpec artifacts. Same-session archived IDs that previously produced an invalid success now fail explicitly; public signatures and metadata shape stay unchanged. There is no database migration, dependency, TUI, hook configuration, installation, or service restart change. Existing `freshList` and `listCreated` rules remain unchanged.
