# Edge Snapping Capability Specification

## ADDED Requirements

### Requirement: Button snaps to edges when near boundary
The button SHALL automatically snap to left or right viewport edge when positioned within 20 pixels of the edge.

#### Scenario: Snap to left edge
- **WHEN** user drags button to within 20 pixels of left viewport edge
- **AND** releases mouse button
- **THEN** the button SHALL automatically align to left edge (left: 0)
- **AND** the button SHALL enter docked state

#### Scenario: Snap to right edge
- **WHEN** user drags button to within 20 pixels of right viewport edge
- **AND** releases mouse button
- **THEN** the button SHALL automatically align to right edge (right: 0)
- **AND** the button SHALL enter docked state

#### Scenario: No snap when beyond threshold
- **WHEN** user positions button more than 20 pixels from any edge
- **AND** releases mouse button
- **THEN** the button SHALL remain at the dragged position
- **AND** the button SHALL NOT enter docked state

### Requirement: Docked state visual indication
The button SHALL provide visual indication when in docked state.

#### Scenario: Docked visual feedback
- **WHEN** button enters docked state
- **THEN** the button SHALL apply 'docked' CSS class
- **AND** button SHALL be aligned precisely to the edge (0 pixel gap)

### Requirement: Only left and right edges support snapping
The system SHALL only support edge snapping for left and right viewport edges.

#### Scenario: Top and bottom edges ignored
- **WHEN** user drags button near top or bottom viewport edge
- **THEN** the button SHALL NOT snap to top or bottom edges
- **AND** button SHALL remain in free-floating position

---

## Metadata

- **Created**: 2026-02-05
- **Source Change**: button-smart-interaction
- **Sync Status**: Synced from delta spec