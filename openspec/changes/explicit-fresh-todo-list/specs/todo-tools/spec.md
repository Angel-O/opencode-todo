# todo-tools Delta

## MODIFIED Requirements

### Requirement: Tool results carry a structured metadata contract

The system SHALL attach a `metadata` payload to every `todowrite` and `todoread` result containing
`source`, `schemaVersion`, `sessionID`, `revision`, the full current `todos` array, a `counts`
summary by status, and boolean `listCreated`, using the same shape for both tools.

#### Scenario: todowrite result includes metadata

- **WHEN** `todowrite` completes successfully
- **THEN** its result's `metadata` includes `source`, `schemaVersion`, `sessionID`, `revision`,
  `todos`, `counts`, and boolean `listCreated` fields describing the session's current list and write

#### Scenario: todoread result includes metadata

- **WHEN** `todoread` completes successfully after a list was created
- **THEN** its result's `metadata` includes the same `source`, `schemaVersion`, `sessionID`,
  `revision`, `todos`, `counts`, and `listCreated` fields as `todowrite`'s result shape
- **AND** `listCreated` is false and the read leaves the stored todos and revision unchanged

## ADDED Requirements

### Requirement: `todowrite` accepts explicit fresh-list intent

The system SHALL accept an optional boolean `freshList` flag on `todowrite`, with omission equivalent to false, and reject supplied non-boolean values before modifying session state.

#### Scenario: Invalid fresh-list flag is rejected before mutation

- **GIVEN** a session has an existing list and revision
- **WHEN** `todowrite` receives a supplied non-boolean `freshList`
- **THEN** the call fails and the stored todos and revision remain unchanged

### Requirement: `todowrite` reports list creation independently of statuses

The system SHALL report `listCreated` true for a successful non-empty write exactly when the pre-write list has no non-archived items or `freshList` is true; otherwise it reports false. Completed and cancelled non-archived items count as an existing list.

#### Scenario: Initial mixed-status list reports creation

- **GIVEN** the session has no non-archived todos
- **WHEN** a non-empty list containing an in-progress item is written with `freshList` omitted or false
- **THEN** `listCreated` is true

#### Scenario: Ordinary non-empty updates do not report creation

- **GIVEN** the session has a non-empty list
- **WHEN** ordinary writes change statuses, add or remove items, reorder items, repeat the list, or replace all items without `freshList: true`
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
- **WHEN** a non-empty list is written without explicit fresh intent
- **THEN** `listCreated` is true regardless of archived history
