# src/omp_plugin/

## Responsibility
In-Process Extension Layer for Oh My Pi (`omp`). Runs natively within the `omp` TypeScript runtime to monitor agent prompt/turn lifecycle, manage a local TCP wake listener, and inject inter-agent messages directly into prompt contexts.

## Design
- **Plugin Architecture**: Adheres to `@oh-my-pi/pi-coding-agent` extension contracts (`ExtensionAPI`, `InputEvent`).
- **Loopback Notification Listener**: Runs an internal `net.Server` that registers its TCP port with `hcom` to receive immediate wake pings upon new message arrival.
- **Process Sub-Bridge**: Executes asynchronous CLI commands (`hcom send`, `hcom hook`) via `child_process.spawn` with timeout bounds.

## Flow
1. **Activation**: `omp` initializes the extension on launch; the extension binds an ephemeral TCP server.
2. **Endpoint Registration**: Fires `hcom hook omp-status --port <port>` to register in `notify_endpoints`.
3. **Event Observation**: Hooks into agent inputs and tool states to keep `hcom` instance state synchronized.
4. **Wake & Injection**: Upon receiving a TCP wake ping, the plugin retrieves incoming messages and feeds them into the conversation buffer.

## Integration
- **Consumed by**: Oh My Pi agent runtime.
- **Depends on**: Node.js standard libraries (`node:net`, `node:child_process`, `node:fs`), `hcom` executable.

