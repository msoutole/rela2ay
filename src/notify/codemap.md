# src/notify/

## Responsibility
Inter-Process Signaling and Wake Notification Subsystem. Implements high-speed, zero-payload signaling using ephemeral loopback TCP connect-and-drop pings to immediately unblock waiting loops without busy-polling SQLite.

## Design
- **Reactor / Interrupt Signal Pattern**: Long-running loops bind an ephemeral TCP port and sleep on a socket accept rather than constantly querying the database.
- **Endpoint Registry**: Active listeners register their port and `WakeKind` in the `notify_endpoints` SQLite table.
- **Fan-Out Wake Dispatch**: `wake::wake` and `wake::wake_all` query active endpoints and send concurrent TCP connection drops within tight millisecond timeouts (`WAKE_TARGETED_MS`).
- **Typed Wake Classification (`WakeKind`)**: Differentiates target listeners across `Pty`, `Hook`, `Listen`, `ListenFilter`, `EventsWait`, and `Plugin`.

## Flow
1. **Server Bind**: A delivery or listener thread constructs `NotifyServer::bind_random()`, securing an ephemeral localhost port.
2. **Registration**: The port is saved into `notify_endpoints` for the agent's instance name.
3. **Wait Loop**: The consumer blocks on `NotifyServer::wait_timeout()`.
4. **Signal Dispatch**: When a message or status change occurs, the sender calls `notify::wake(db, instance, WakeKind::Pty)`.
5. **Wakeup & Cleanup**: The TCP connection drops instantly; `NotifyServer` returns `true`; consumer reads new DB events; upon termination, `NotifyServer` drops and cleans up the endpoint row.

## Integration
- **Consumed by**: `src/delivery.rs`, `src/commands/listen.rs`, `src/commands/events.rs`, `src/commands/send.rs`, `src/hooks/`.
- **Depends on**: `src/db/`, `std::net::TcpListener`, `std::net::TcpStream`.

