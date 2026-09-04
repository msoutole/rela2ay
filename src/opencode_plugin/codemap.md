# src/opencode_plugin/

## Responsibility
In-Process Extension Layer for OpenCode. Integrates natively with OpenCode's plugin subsystem (`@opencode-ai/plugin`, `@opencode-ai/sdk`) to track session events, intercept permission requests, maintain instance status, and forward inter-agent communications into the OpenCode context.

## Design
- **Plugin Hook Provider**: Exports a compliant OpenCode plugin interface capturing events across session start, message generation, and tool invocation.
- **Event Filter & Session Extractor**: Extracts session identifiers, agent models, and permission dialogs from complex `HcomEvent` payloads.
- **Asynchronous IPC Gateway**: Uses managed child process spawns to execute `hcom` hook calls without stalling the OpenCode interactive UI.

## Flow
1. **Load**: OpenCode initializes the plugin during agent session setup.
2. **Session Identification**: Intercepts initial event to resolve active `sessionID` and instance association.
3. **Status Synchronization**: Forwards tool execution states and permission dialogues to `hcom hook opencode`.
4. **Message Delivery**: Polls/receives wake triggers and feeds incoming inter-agent messages into the active chat session.

## Integration
- **Consumed by**: OpenCode runtime.
- **Depends on**: `@opencode-ai/plugin`, `@opencode-ai/sdk`, `hcom` CLI binary.

