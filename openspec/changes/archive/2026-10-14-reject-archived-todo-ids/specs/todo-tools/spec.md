# Spec Delta

## ADDED Requirements

### Requirement: Archived-ID failures provide actionable recovery

The system SHALL surface a same-session archived-ID write failure as a tool error identifying the stale ID and advising use of a current item ID or omission of the ID, without a successful metadata result.

#### Scenario: Archived-ID error preserves the list and supports ordinary recovery

- **GIVEN** a session has a non-empty active list and an archived item
- **WHEN** `todowrite` receives that archived ID
- **THEN** the tool fails with an error naming the ID and advising use of a current item ID or omission of the ID
- **AND** the stored list and revision remain unchanged and no successful metadata is returned
- **WHEN** the caller reads the current list and resubmits the full intended list with current IDs or without the stale ID, without fresh intent
- **THEN** the write succeeds and `listCreated` is false

## MODIFIED Requirements

### Requirement: `todowrite` reports list creation independently of statuses

The system SHALL report `listCreated` true for a successful non-empty write exactly when the pre-write list has no non-archived items or `freshList` is true; otherwise it reports false. Completed and cancelled non-archived items count as an existing list. Archived-ID rejection produces no successful creation result.

#### Scenario: Initial mixed-status list reports creation

- **GIVEN** the session has no non-archived todos
- **WHEN** a non-empty list containing an in-progress item is written with `freshList` omitted or false
- **THEN** `listCreated` is true

#### Scenario: Ordinary non-empty updates do not report creation

- **GIVEN** the session has a non-empty list
- **WHEN** ordinary successful writes change statuses, add or remove items, reorder items, repeat the list, or replace all items without `freshList: true`
- **THEN** each result reports `listCreated` false, including all-pending writes and writes following completed or cancelled lists

#### Scenario: Explicit fresh list can carry existing items

- **GIVEN** the session has a non-empty list with an unfinished item
- **WHEN** `todowrite` writes a non-empty list with `freshList: true` retaining that item's ID
- **THEN** `listCreated` is true and the carried item's identity and existing timestamp semantics are preserved

#### Scenario: Repeated explicit fresh writes report creation

- **GIVEN** the session has a non-empty list
- **WHEN** the same unchanged list is written repeatedly with `freshList: true`
- **THEN** each successful result reports `listCreated` true

#### Scenario: Empty writes do not report creation

- **WHEN** an empty list is written with `freshList` omitted, false, or true
- **THEN** `listCreated` is false and normal omission archival and revision increment behavior applies

#### Scenario: Non-empty list after clearing reports creation

- **GIVEN** the session's active list was cleared and its former items remain archived
- **WHEN** a non-empty list with no same-session archived IDs is successfully written without explicit fresh intent
- **THEN** `listCreated` is true regardless of archived history
