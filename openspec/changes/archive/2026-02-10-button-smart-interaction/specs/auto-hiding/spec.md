## ADDED Requirements

### Requirement: Button auto-hides when docked
The button SHALL automatically hide 50% of its width when docked to an edge and auto-hide is enabled.

#### Scenario: Auto-hide after delay
- **WHEN** button is docked to an edge
- **AND** 400 milliseconds have elapsed
- **AND** mouse is not hovering over button
- **AND** control panel is not open
- **THEN** the button SHALL translate 50% of its width toward the edge (25px for 50px button)
- **AND** the hiding SHALL use smooth animation over 200ms

#### Scenario: Show on mouse hover
- **WHEN** button is in hidden state (partially hidden)
- **AND** user moves mouse cursor over the button area
- **THEN** the button SHALL immediately show 100% of its width
- **AND** the showing SHALL use smooth animation over 200ms

#### Scenario: Hide on mouse leave
- **WHEN** user moves mouse cursor away from hidden button
- **AND** control panel is not open
- **AND** 400 milliseconds have elapsed
- **THEN** the button SHALL return to 50% hidden state
- **AND** the hiding SHALL use smooth animation over 200ms

### Requirement: Auto-hide respects panel state
The auto-hide mechanism SHALL not hide button when control panel is open.

#### Scenario: Panel open prevents hide
- **WHEN** control panel is open
- **AND** mouse is not hovering over button
- **THEN** the button SHALL remain fully visible
- **AND** auto-hide timer SHALL be cancelled

#### Scenario: Panel close resumes auto-hide
- **WHEN** control panel is closed
- **AND** button is docked
- **AND** auto-hide is enabled
- **THEN** auto-hide timer SHALL restart after 400ms delay

### Requirement: Auto-hide can be disabled
The system SHALL provide a mechanism to disable auto-hide functionality.

#### Scenario: Auto-hide disabled
- **WHEN** user disables auto-hide via menu command
- **THEN** the button SHALL remain fully visible at all times
- **AND** auto-hide timers SHALL be cancelled

#### Scenario: Auto-hide enabled
- **WHEN** user enables auto-hide via menu command
- **AND** button is docked
- **THEN** auto-hide behavior SHALL resume after 400ms delay
