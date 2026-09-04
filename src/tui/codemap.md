# src/tui/

## Responsibility
Interactive Terminal User Interface (TUI) Dashboard. Provides a real-time monitor and control center powered by `ratatui` and `crossterm` for tracking active agent instances, observing inter-agent conversations, inspecting logs, launching agents, and injecting operator messages.

## Design
- **Model-View-Update (MVU) Architecture**:
  - `state.rs` & `model.rs`: Holds UI state (selected instances, active tabs, filter criteria, compose modal state).
  - `render/`: Decomposes UI layouts into specialized widget renderers.
  - `actions.rs` & `app.rs`: Processes user intent into typed state mutations.
- **Dual View Modes**: Supports full-screen alternate terminal screen buffer (`ViewMode::FullScreen`) or inline non-destructive viewport (`ViewMode::Inline`) embedded into shell scrollback.
- **Async Database & RPC Feeder**: `db.rs` and `rpc_async.rs` query SQLite state and trigger background operations asynchronously, preventing frame rate drops.
- **Command Palette Engine**: `commands.rs` supports vim-style command inputs (`:q`, `:kill`, `:filter`).

## Flow
1. **Startup**: Terminal enters raw mode; `tui::run` instantiates `App` and configures the Crossterm backend.
2. **Event Loop**: `App::run` executes a loop polling keyboard events, mouse clicks, and periodic tick timeouts.
3. **Action Dispatch**: Keypresses route through `input/` into `actions::Action`, updating `AppState`.
4. **Frame Rendering**: Each frame calls `render::draw_ui`, composing agent tables, message logs, and status bars.
5. **Teardown**: Restores terminal modes and flushes buffers upon exit.

## Integration
- **Consumed by**: `src/commands/status.rs`, `src/router.rs`.
- **Depends on**: `ratatui`, `crossterm`, `src/db/`, `src/shared/`, `src/commands/`.

