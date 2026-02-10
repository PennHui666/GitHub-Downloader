## ADDED Requirements

### Requirement: Control panel positions based on button location
The control panel SHALL automatically calculate and use the optimal position based on the button's current location and viewport space availability.

#### Scenario: Position up-left of button
- **WHEN** button is in lower-right quadrant of viewport
- **AND** sufficient space exists above and to the left of button
- **THEN** panel SHALL open above and to the left of button
- **AND** panel bottom edge SHALL align with button top edge minus gap
- **AND** panel right edge SHALL align with button left edge minus gap

#### Scenario: Position up-right of button
- **WHEN** button is in lower-left quadrant of viewport
- **AND** sufficient space exists above and to the right of button
- **THEN** panel SHALL open above and to the right of button
- **AND** panel bottom edge SHALL align with button top edge minus gap
- **AND** panel left edge SHALL align with button right edge plus gap

#### Scenario: Position down-left of button
- **WHEN** button is in upper-right quadrant of viewport
- **AND** sufficient space exists below and to the left of button
- **THEN** panel SHALL open below and to the left of button
- **AND** panel top edge SHALL align with button bottom edge plus gap
- **AND** panel right edge SHALL align with button left edge minus gap

#### Scenario: Position down-right of button
- **WHEN** button is in upper-left quadrant of viewport
- **AND** sufficient space exists below and to the right of button
- **THEN** panel SHALL open below and to the right of button
- **AND** panel top edge SHALL align with button bottom edge plus gap
- **AND** panel left edge SHALL align with button right edge plus gap

### Requirement: Panel position adjusts for space constraints
When optimal direction lacks sufficient space, the system SHALL select alternative direction.

#### Scenario: Fallback when primary direction lacks space
- **WHEN** button is in lower-right quadrant
- **AND** insufficient space exists above button for panel height
- **THEN** panel SHALL open below button instead of above
- **AND** position SHALL be constrained to remain within viewport

### Requirement: Panel follows button during drag
When control panel is open and user drags the button, the panel SHALL update its position in real-time.

#### Scenario: Panel follows dragged button
- **WHEN** control panel is open
- **AND** user drags the button to new position
- **THEN** panel position SHALL update continuously during drag
- **AND** panel SHALL use updated direction calculation based on new position

### Requirement: Panel constrained within viewport
The control panel SHALL always remain fully within the viewport boundaries.

#### Scenario: Panel position clamped to viewport
- **WHEN** calculated panel position would place panel partially outside viewport
- **THEN** panel position SHALL be adjusted to ensure full visibility
- **AND** panel SHALL be at least 0 pixels from all viewport edges
