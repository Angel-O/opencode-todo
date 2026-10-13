# todo-storage Delta

## ADDED Requirements

### Requirement: Write results bind creation signals to committed session state

The system SHALL return boolean `listCreated` alongside the resulting todos and revision of a successful write, derived from that transaction's non-archived pre-write items and optional fresh intent, without persisting list identity or altering existing item, timestamp, archival, ordering, or revision semantics.

#### Scenario: Sequential ordinary writes signal creation only once

- **GIVEN** a session has no non-archived items
- **WHEN** two ordinary non-empty writes succeed for that session
- **THEN** the first returns `listCreated` true and the second false, each alongside its own resulting todos and revision

#### Scenario: Explicit fresh intent preserves timestamp-aware writes

- **GIVEN** a session has existing items with IDs and timestamps
- **WHEN** a non-empty write with explicit fresh intent and a caller-supplied timestamp carries items forward, completes an item, reorders items, and omits another item
- **THEN** the result reports creation while item matching, creation/completion timestamps, archival, ordering, and revision use the existing rules and supplied timestamp

#### Scenario: Invalid store fresh intent is rejected before mutation

- **GIVEN** a session has an existing list and revision
- **WHEN** a direct write supplies non-boolean fresh intent
- **THEN** it fails before mutation and stored todos and revision remain unchanged

#### Scenario: Failed write rolls back creation and session changes

- **GIVEN** a write would report list creation if it succeeded
- **WHEN** a failure occurs during that write
- **THEN** no successful creation result is returned and the session's todos and revision retain their pre-write values
