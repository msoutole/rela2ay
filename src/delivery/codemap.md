# src/delivery/

## Responsibility
Tool-Specific Delivery Adapters and Teardown Management Subsystem. Supplements the core PTY delivery loop (`src/delivery.rs`) with custom agent exit handling, soft-stop reconciliation, and specialized lifecycle cleanup (e.g., Antigravity `SessionEnd`).

## Design
- **Lifecycle Teardown Strategy**: Accommodates agent runtimes with multi-phase teardown protocols (such as Antigravity/Gemini soft stops) where an agent marks itself inactive before the outer PTY exits.
- **Defensive Resource Reclaim**: Selectively cleans up process bindings, TCP notify endpoints, and pub/sub subscriptions without corrupting shared instance metadata.

## Flow
1. **PTY Child Exit**: When an agent's child process closes, `src/delivery.rs` dispatches to tool-specific cleanup handlers.
2. **State Check**: `cleanup_antigravity_pty_exit` verifies if the instance was already transitioned to `inactive` by a previous `SessionEnd` event.
3. **Endpoint Purge**: Safely deletes obsolete wake endpoints and subscriptions from `notify_endpoints`.
4. **Binding Release**: Releases PID and process bindings in SQLite.

## Integration
- **Consumed by**: `src/delivery.rs`.
- **Depends on**: `src/db/`, `src/shared/`, `src/log.rs`.

