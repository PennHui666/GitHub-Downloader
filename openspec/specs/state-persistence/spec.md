# State Persistence Capability Specification

## ADDED Requirements

### Requirement: Button position persisted across sessions
The system SHALL save button position and state to persistent storage using GM_setValue/GM_getValue.

#### Scenario: Save state on position change
- **WHEN** user completes a drag operation
- **OR** button docks/undocks
- **OR** auto-hide setting changes
- **THEN** current state SHALL be saved to storage
- **AND** state SHALL include: x position, y position, docked flag, edge, autoHideEnabled flag

#### Scenario: Restore state on page load
- **WHEN** script initializes on GitHub page
- **AND** saved state exists in storage
- **THEN** button SHALL be positioned at saved coordinates
- **AND** docked state SHALL be restored if previously docked
- **AND** auto-hide preference SHALL be restored

#### Scenario: Default position when no saved state
- **WHEN** script initializes
- **AND** no saved state exists
- **THEN** button SHALL use default position (right: 20px, bottom: 30px)
- **AND** auto-hide SHALL be enabled by default

### Requirement: State storage handles errors gracefully
The system SHALL handle storage read/write errors without crashing.

#### Scenario: Failed state save
- **WHEN** save operation fails (e.g., storage quota exceeded)
- **THEN** error SHALL be logged to console
- **AND** operation SHALL continue without saved state

#### Scenario: Corrupted state data
- **WHEN** loading state and data is corrupted or invalid JSON
- **THEN** error SHALL be logged to console
- **AND** default state SHALL be used
- **AND** operation SHALL continue normally

### Requirement: State updated in real-time during interactions
The system SHALL maintain accurate state throughout user interactions.

#### Scenario: State during drag operation
- **WHEN** user drags button
- **THEN** state.x and state.y SHALL update in real-time
- **AND** state.isDragging SHALL be true

#### Scenario: State on dock completion
- **WHEN** button completes docking to edge
- **THEN** state.docked SHALL be set to true
- **AND** state.edge SHALL store the docked edge ('left' or 'right')
- **AND** state SHALL be saved to storage

---

## Metadata

- **Created**: 2026-02-05
- **Source Change**: button-smart-interaction
- **Sync Status**: Synced from delta spec