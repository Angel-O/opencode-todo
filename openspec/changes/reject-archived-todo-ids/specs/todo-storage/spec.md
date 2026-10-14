# Spec Delta

## ADDED Requirements

### Requirement: Writes reject archived same-session IDs atomically

The system SHALL reject an entire write before any mutation if a supplied ID belongs to an archived item in the same session, regardless of fresh-list intent, preserving all active and archived rows and the session revision.

#### Scenario: Late archived ID rejects a mixed batch without mutation

- **GIVEN** a session has archived history and an active ordered list with timestamps and revision
- **WHEN** a write includes an active-item update, a new item, and then a same-session archived ID, with fresh intent omitted or true
- **THEN** the write fails and all active and archived rows, timestamps, order, and revision remain unchanged

#### Scenario: Archived ID is rejected with an empty active list

- **GIVEN** a session has archived history but no active items
- **WHEN** a write supplies an ID from that history
- **THEN** the write fails without changing history or revision

#### Scenario: Unknown and other-session archived IDs remain acceptable

- **GIVEN** an ID is archived only in another session
- **WHEN** a write supplies that ID and a genuinely unknown ID in the calling session
- **THEN** both items are created successfully in the calling session and the other session's history remains unchanged

## MODIFIED Requirements

### Requirement: Item identity is preserved across writes

The system SHALL match incoming todo items against existing non-archived rows for the same session by `id`
first, then by exact `content` string for any incoming item with no `id`, before treating a
remaining unmatched item as new, except that a supplied same-session archived ID rejects the write.

#### Scenario: Matching item by id preserves its history

- **GIVEN** a session has an existing todo item with a known `id`, `created_at`, and `status`
- **WHEN** a write includes that same `id` with an unchanged `content` and `status`
- **THEN** the stored item's `created_at` is unchanged and no new row is created

#### Scenario: Id-less item recovered by content match

- **GIVEN** a session has an existing todo item with a known `created_at`
- **WHEN** a write includes an item with no `id` but identical `content` to that existing item
- **THEN** the write is treated as an update to the existing item, not the creation of a new one,
  and the existing `created_at` is preserved

#### Scenario: Genuinely new item is created

- **WHEN** a write includes an item whose `id` (if present) and `content` match no existing
  non-archived row for that session, and no supplied ID belongs to that session's archived history
- **THEN** a new row is created for that item with a freshly minted `id` if none was supplied

### Requirement: Item order and write revision are tracked

The system SHALL persist each item's position from the order of the array supplied to a successful write,
replacing any previously stored order for that session, and SHALL increment a per-session
revision counter on every successful write to that session.

#### Scenario: Item order reflects the most recent write

- **GIVEN** a session's todo list was previously written in one order
- **WHEN** a subsequent successful write supplies the same items in a different order
- **THEN** a read for that session returns the items in the order of the most recent write

#### Scenario: Revision increments on every write

- **WHEN** a session receives two or more successful writes
- **THEN** each write's returned revision is exactly one greater than the previous successful write's
  revision for that session
