# src/hooks/

## Responsibility
Tool Lifecycle Integration and Interception Middleware Layer. Serves as the primary bridge between external AI coding tool runtimes (Claude Code, Gemini CLI, OpenAI Codex, OpenCode, Kilo, Pi, Oh My Pi, Antigravity, Cursor, Kimi, Copilot) and `hcom`, converting tool-specific hook executions into unified instance status updates, transcript tracking, and message delivery.

## Design
- **Adapter Pattern**: Specialized adapters (`claude.rs`, `gemini.rs`, `codex.rs`, `opencode.rs`, `cursor.rs`, etc.) translate heterogeneous tool event formats (stdin JSON, CLI flags, file watchers) into normalized domain representations.
- **Template Method Pattern**: `common.rs` defines reusable hook workflows (session binding, PID association, status transition, message queue inspection, wake signaling).
- **Synchronous Delivery Gating**: Intercepts tool execution points (e.g. post-tool-use, prompt turn) to synchronously deliver pending inter-agent messages into the tool's context.

## Flow
1. **Hook Invocation**: An AI tool triggers a configured hook script (e.g., `hcom hook claude post-tool-use` or `hcom hook gemini session-start`).
2. **Payload Deserialization**: The adapter parses input JSON or environment context into structured metadata.
3. **Session & Instance Binding**: Maps the current OS process and tool session ID to an active `hcom` instance name via `instance_binding`.
4. **State Transition & Event Logging**: Updates instance status (`active`, `listening`, `blocked`) and logs lifecycle events to SQLite `events`.
5. **Message Injection**: Retrieves queued messages from DB, formats them for the tool's input protocol, and outputs or injects them before relinquishing control.

## Integration
- **Consumed by**: AI coding CLI runtimes via hook configurations (Claude hooks, Gemini settings, Cursor rules).
- **Depends on**: `src/db/`, `src/shared/`, `src/notify/`, `src/transcript/`, `src/sys/`.

