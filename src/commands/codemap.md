# src/commands/

## Responsibility
CLI Command Controller and Action Dispatch Layer. Implements the complete user- and agent-facing command-line surface of `hcom`, divided into messaging, instance lifecycle, diagnostics, configuration, and distributed relay management.

## Design
- **Command Pattern**: Encapsulates each CLI operation into a dedicated module with its own argument parsing and execution flow:
  - **Messaging**: `send.rs` (targeted and broadcast message dispatch), `listen.rs` (blocking/streaming event listener).
  - **Lifecycle**: `launch.rs` (agent launcher with terminal preset options), `start.rs`, `stop.rs`, `kill.rs` (process tree termination), `resume.rs`, `fork.rs`, `daemon.rs`.
  - **Diagnostics**: `events.rs` (filtered event queries), `list.rs`, `status.rs`, `term.rs` (virtual PTY screen querying), `transcript.rs` (multilevel agent transcript dumping), `bundle.rs`.
  - **Management**: `config.rs`, `help.rs`, `hooks.rs`, `relay.rs` (E2EE mesh pairing and syncing), `reset.rs`, `run.rs`, `update.rs`, `archive.rs`.
- **Context Injection**: Commands receive a shared `HcomContext` providing unified access to SQLite handles, configuration defaults, and resolved caller identity.

## Flow
1. **Routing**: `src/router.rs` decodes CLI subcommands and passes execution to `commands::<subcommand>::run`.
2. **Validation**: Arguments are validated against domain schemas (`src/core/helpers.rs`, `src/tools/launch_arg_validation.rs`).
3. **Execution**: The command coordinates actions across the database (`HcomDb`), process execution (`launcher.rs`), signaling (`notify::wake`), or network relay (`relay`).
4. **Presentation**: Results are emitted to stdout as formatted ANSI text or structured JSON.

## Integration
- **Consumed by**: `src/router.rs`, `src/main.rs`.
- **Depends on**: `src/core/`, `src/db/`, `src/shared/`, `src/launcher.rs`, `src/notify/`, `src/relay/`, `src/pty/`, `src/transcript/`.

