# src/pi_plugin/

## Responsibility
In-Process Extension Layer for Pi Coding Agent (`pi`). Implements the `@earendil-works/pi-coding-agent` extension contract to bind running sessions, track tool execution lifecycle, listen for TCP wake notifications, and inject inter-agent messages into the Pi environment.

## Design
- **Extension API Integration**: Implements `ExtensionAPI`, `ExtensionContext`, and `InputEvent` hooks.
- **Embedded Loopback Server**: Listens on a local ephemeral TCP port to receive low-latency wake notifications from `hcom`.
- **Subprocess CLI Delegator**: Delegates DB reads, event insertions, and status reporting to the core `hcom` binary via wrapped child processes.

## Flow
1. **Startup**: Pi loads the extension on launch; the extension binds an ephemeral TCP port.
2. **Registration**: Emits status hook via `hcom hook pi-status` registering the wake port.
3. **Session Interception**: Hooks user input and agent tool executions to synchronize agent status.
4. **Message Reception**: Upon TCP ping, fetches unread messages from SQLite and delivers them into the current interaction turn.

## Integration
- **Consumed by**: Pi coding agent runtime.
- **Depends on**: `@earendil-works/pi-coding-agent`, Node.js runtime, `hcom` CLI binary.

