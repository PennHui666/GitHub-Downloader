# Menu Commands Capability Specification

## ADDED Requirements

### Requirement: Menu commands registered on script load
The system SHALL register user-accessible menu commands using GM_registerMenuCommand on script initialization.

#### Scenario: Menu commands available
- **WHEN** script initializes successfully
- **THEN** following menu commands SHALL be registered:
  - "🔄 重置按钮位置" - Reset button to default position
  - "⏱️ 切换自动隐藏" - Toggle auto-hide on/off
  - "📊 查看当前状态" - Display current button state information
  - "🗑️ 清除状态存储" - Clear all saved state from storage
  - "🔧 切换调试模式" - Toggle debug logging on/off

### Requirement: Reset button position command
The reset command SHALL restore button to default state and position.

#### Scenario: Execute reset command
- **WHEN** user selects "🔄 重置按钮位置" from menu
- **THEN** confirmation dialog SHALL be displayed
- **AND** upon confirmation, button SHALL return to default position (right: 20px, bottom: 30px)
- **AND** docked state SHALL be cleared
- **AND** hidden state SHALL be cleared
- **AND** saved state SHALL be removed from storage
- **AND** control panel SHALL be closed if open

### Requirement: Toggle auto-hide command
The toggle command SHALL enable or disable auto-hide functionality.

#### Scenario: Enable auto-hide via menu
- **WHEN** user selects "⏱️ 切换自动隐藏" from menu
- **AND** auto-hide is currently disabled
- **THEN** auto-hide SHALL be enabled
- **AND** user SHALL receive confirmation message "✅ 自动隐藏已启用"
- **AND** if button is docked, auto-hide timer SHALL start

#### Scenario: Disable auto-hide via menu
- **WHEN** user selects "⏱️ 切换自动隐藏" from menu
- **AND** auto-hide is currently enabled
- **THEN** auto-hide SHALL be disabled
- **AND** user SHALL receive confirmation message "❌ 自动隐藏已禁用"
- **AND** any pending hide timers SHALL be cancelled
- **AND** button SHALL be fully visible if hidden

### Requirement: View current state command
The view state command SHALL display current button configuration and status.

#### Scenario: Display state information
- **WHEN** user selects "📊 查看当前状态" from menu
- **THEN** alert dialog SHALL display formatted information including:
  - Current X and Y position (or "未设置" if not set)
  - Docked status (是/否) and docked edge (if docked)
  - Hidden status (是/否)
  - Panel open status (是/否) and panel direction (if open)
  - Auto-hide enabled status (启用/禁用)

### Requirement: Clear state storage command
The clear command SHALL remove all saved state from persistent storage.

#### Scenario: Clear all stored state
- **WHEN** user selects "🗑️ 清除状态存储" from menu
- **THEN** confirmation dialog SHALL be displayed with warning
- **AND** upon confirmation, all saved state SHALL be removed
- **AND** button SHALL be reset to default position
- **AND** user SHALL receive confirmation message "✅ 状态存储已清除"

### Requirement: Toggle debug logging via menu command
The system SHALL provide a menu command to enable or disable debug logging persistence.

#### Scenario: Enable debug logging
- **WHEN** user selects "🔧 切换调试模式" from menu
- **AND** debug logging is currently disabled
- **THEN** debug logging SHALL be enabled and persisted using GM_setValue
- **AND** user SHALL receive confirmation message "✅ 调试模式已开启\n\n页面将自动刷新以应用设置"
- **AND** page SHALL automatically refresh after 500ms delay to apply the new debug setting

#### Scenario: Disable debug logging
- **WHEN** user selects "🔧 切换调试模式" from menu
- **AND** debug logging is currently enabled
- **THEN** debug logging SHALL be disabled and persisted using GM_setValue
- **AND** user SHALL receive confirmation message "❌ 调试模式已关闭\n\n页面将自动刷新以应用设置"
- **AND** page SHALL automatically refresh after 500ms delay to apply the new debug setting

#### Scenario: Debug state persists across page reloads
- **WHEN** user enables debug logging via menu
- **AND** user reloads the page
- **THEN** debug logging SHALL remain enabled
- **AND** logs SHALL be displayed immediately without manual re-enabling

---

## Metadata

- **Created**: 2026-02-05
- **Source Change**: button-smart-interaction
- **Sync Status**: Synced from delta spec