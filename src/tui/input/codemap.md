# src/tui/input/

## Responsibility
User Input Controller and Text Composition Subsystem. Translates raw terminal keyboard and mouse events into UI navigation, modal controls, confirmation dialog handling, and agent message authoring.

## Design
- **State-Dependent Event Router**: Evaluates input relative to the active `InputMode` (`Normal`, `Compose`, `Launch`, `Relay`, `Confirm`, `Help`).
- **Interactive Compose Buffer**: `compose.rs` implements inline text editing capabilities (cursor repositioning, backspace, kill-ring actions, and `@agent` mention parsing via `parse_outbound_message`).
- **Cross-Platform Keyboard Normalization**: Differentiates Ctrl modifiers from AltGr combinations on international layouts.

## Flow
1. **Key Event Ingestion**: Crossterm event loop captures keystrokes and routes them to `App::handle_key`.
2. **Modal Evaluation**: If an overlay (confirmation dialog, help popup, launch wizard) is open, input is consumed by the corresponding modal handler.
3. **Compose Processing**: In compose mode, characters are appended to the active draft; pressing Enter triggers `parse_outbound_message` and queues an async `RpcOp::Send` task.
4. **Navigation & Commands**: In normal mode, directional keys adjust selection cursors and tab switching.

## Integration
- **Consumed by**: `src/tui/app.rs`, `src/tui/mod.rs`.
- **Depends on**: `crossterm::event`, `src/tui/model.rs`, `src/tui/rpc_async.rs`, `src/shared/constants.rs`.

