# src/tui/inline/

## Responsibility
Inline Viewport and Scrollback Ejection Subsystem. Coordinates inline TUI operation, managing a fixed-height terminal window at the bottom of the screen while streaming newly arrived inter-agent messages and events upward into the shell's native scrollback history.

## Design
- **Incremental Stream Ejector**: `eject::Ejector` tracks monotonic event/message IDs (`last_event_row_id`, `last_msg_id`) to only push unprinted messages into scrollback.
- **Batch Replay Throttle**: Handles terminal resize and filter changes by replaying history lines in configurable increments (`replay_lines_per_tick`) to prevent terminal flicker or buffer overflow.
- **Scroll Region Partitioning**: Uses ANSI scroll margin control (`\x1b[r`) to protect shell history while reserving bottom lines for live interactive controls.

## Flow
1. **Event Polling**: `App` tick identifies new events from `DataState`.
2. **Ejection Check**: `Ejector` checks if new message IDs exceed previous high-water mark.
3. **Format & Scroll**: Converts domain events into styled lines and writes them to stdout above the viewport boundary.
4. **Replay Reconciliation**: If the user alters filters or changes terminal dimensions, `Ejector` resets watermarks and queues a re-rendered replay batch.

## Integration
- **Consumed by**: `src/tui/app.rs`, `src/tui/mod.rs`.
- **Depends on**: `src/tui/render/messages.rs`, `src/tui/theme.rs`, `ratatui`, `crossterm`.

