## 1. Setup and Configuration

- [x] 1.1 Update Tampermonkey metadata to add GM API grants (`@grant GM_setValue`, `@grant GM_getValue`, `@grant GM_registerMenuCommand`)
- [x] 1.2 Add CONFIG constant with all configuration parameters (SNAP_THRESHOLD, HIDE_DELAY, ANIMATION_DURATION, HIDE_RATIO, BUTTON_SIZE, PANEL_WIDTH, PANEL_HEIGHT, PANEL_GAP, DRAG_THRESHOLD, SUPPORTED_EDGES, DEFAULT_RIGHT, DEFAULT_BOTTOM)
- [x] 1.3 Add state object with all state properties (x, y, docked, edge, hidden, isDragging, hasDragged, dragStartX, dragStartY, buttonStartX, buttonStartY, isMouseOver, panelOpen, panelDirection, autoHideEnabled, hideTimer)

## 2. State Management Functions

- [x] 2.1 Implement `saveState()` function to persist button state using GM_setValue
- [x] 2.2 Implement `loadState()` function to restore button state using GM_getValue with error handling
- [x] 2.3 Implement `applyState()` function to apply loaded state to button DOM element
- [x] 2.4 Implement `resetButton()` function to reset button to default position and clear storage
- [x] 2.5 Add error handling for corrupted or missing state data

## 3. Drag Functionality

- [x] 3.1 Implement `onMouseDown()` handler to initiate drag operation
- [x] 3.2 Implement `onMouseMove()` handler to update button position during drag with boundary clamping
- [x] 3.3 Implement `onMouseUp()` handler to complete drag operation and save state
- [x] 3.4 Implement `clampPosition()` function to constrain button within viewport boundaries
- [x] 3.5 Implement 3px threshold detection in mousemove to distinguish drag from click
- [x] 3.6 Add visual feedback CSS class 'dragging' during drag operations
- [x] 3.7 Prevent click event from toggling panel when drag is detected

## 4. Edge Snapping and Docking

- [x] 4.1 Implement `checkEdgeSnap()` function to detect if button is within 20px of left/right edges
- [x] 4.2 Implement `applyDocked()` function to align button to edge and set docked state
- [x] 4.3 Implement `clearDocked()` function to remove docked state when button is undocked
- [x] 4.4 Add CSS class 'docked' when button is docked to edge
- [x] 4.5 Ensure docking only supports left and right edges (ignore top/bottom)

## 5. Auto-hiding Mechanism

- [x] 5.1 Implement `scheduleHide()` function to start 400ms timer for auto-hide
- [x] 5.2 Implement `cancelHide()` function to clear pending hide timer
- [x] 5.3 Implement `hideButton()` function to apply 50% transform translation toward edge
- [x] 5.4 Implement `showButton()` function to restore button to full visibility
- [x] 5.5 Implement `onMouseEnter()` handler to show button immediately when hovered
- [x] 5.6 Implement `onMouseLeave()` handler to schedule hide when mouse leaves (if panel closed)
- [x] 5.7 Ensure auto-hide is cancelled when control panel is open
- [x] 5.8 Implement `toggleAutoHide()` function to enable/disable auto-hide via menu

## 6. Smart Panel Positioning

- [x] 6.1 Implement `calculatePanelDirection()` function using four-quadrant algorithm
- [x] 6.2 Implement `updatePanelPosition()` function to position panel based on calculated direction
- [x] 6.3 Implement `clampPanelPosition()` function to ensure panel stays within viewport
- [x] 6.4 Update panel position in real-time during button drag (if panel is open)
- [x] 6.5 Ensure panel uses 10px gap from button edges

## 7. Menu Commands

- [x] 7.1 Implement `registerMenuCommands()` function to register all GM_registerMenuCommand handlers
- [x] 7.2 Implement reset position menu command with confirmation dialog
- [x] 7.3 Implement toggle auto-hide menu command with status message
- [x] 7.4 Implement view current state menu command with formatted alert display
- [x] 7.5 Implement clear state storage menu command with confirmation dialog

## 8. Window Responsiveness

- [x] 8.1 Implement `onWindowResize()` handler to check button position on window resize
- [x] 8.2 Reset button to default position if it becomes outside viewport after resize
- [x] 8.3 Update panel position when window is resized (if panel is open)

## 9. CSS Styles and Visual Updates

- [x] 9.1 Update button CSS to use cursor: grab and add user-select: none
- [x] 9.2 Add CSS for .dragging class (cursor: grabbing, box-shadow enhancement, scale 1.1)
- [x] 9.3 Add CSS for .docked class (transition for transform)
- [x] 9.4 Add CSS transitions for smooth hide/show animations (200ms)
- [x] 9.5 Update button creation to support dynamic left/top positioning

## 10. Integration and Event Binding

- [x] 10.1 Update `createControlPanel()` to bind mousedown, mouseenter, mouseleave events to button
- [x] 10.2 Add global document event listeners for mousemove and mouseup
- [x] 10.3 Add window resize event listener
- [x] 10.4 Update button click handler to check hasDragged flag before toggling panel
- [x] 10.5 Update panel open/close functions to interact with auto-hide mechanism
- [x] 10.6 Update `init()` function to call loadState(), applyState(), and registerMenuCommands()

## 11. Testing and Verification

- [x] 11.1 Test drag functionality: button follows mouse, boundary constraints work
- [x] 11.2 Test drag vs click distinction: 3px threshold prevents panel toggle during drag
- [x] 11.3 Test edge snapping: left/right edges snap at 20px threshold
- [x] 11.4 Test auto-hide: 400ms delay, 50% hide, hover to show
- [x] 11.5 Test panel positioning: four directions based on button location
- [x] 11.6 Test state persistence: position and settings saved/restored across page reloads
- [x] 11.7 Test menu commands: reset, toggle auto-hide, view state, clear storage, toggle debug
- [x] 11.8 Test window resize: button resets if outside viewport
- [x] 11.9 Test backward compatibility: existing download functionality still works
- [x] 11.10 Test error handling: corrupted state, missing APIs

## 12. Documentation and Cleanup

- [x] 12.1 Update script version number in metadata
- [x] 12.2 Add JSDoc comments to new functions
- [x] 12.3 Verify all console.log messages use [GitHub Downloader] prefix
- [x] 12.4 Clean up any TODO comments and debugging code
- [x] 12.5 Final test of complete user workflow

## 13. Debug Logging Improvement

- [x] 13.1 Replace window.GITHUB_DOWNLOADER_DEBUG with GM_getValue('debug_enabled', false)
- [x] 13.2 Update log() and error() functions to use DEBUG constant
- [x] 13.3 Implement toggleDebug() function with user feedback alerts and auto-refresh functionality
- [x] 13.4 Add "🔧 切换调试模式" menu command to registerMenuCommands()
- [x] 13.5 Update log prefix to [GitHub Downloader] (English)
- [x] 13.6 Update proposal.md to include debug logging improvement
- [x] 13.7 Update toggleDebug() to add 500ms auto-refresh after confirmation
- [x] 13.8 Update confirmation messages to indicate auto-refresh behavior
- [x] 13.7 Update specs/menu-commands/spec.md with new requirements
