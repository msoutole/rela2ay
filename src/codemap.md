# src/

## Responsibility
Core Application Kernel and Process Coordination Layer. Houses the binary entry point (`main.rs`), global configuration (`config.rs`), dynamic CLI dispatch router (`router.rs`), agent lifecycle state machine (`instance_lifecycle.rs`), launcher engine (`launcher.rs`), message delivery loop (`delivery.rs`), agent bootstrapping prompt generator (`bootstrap.rs`), tool specifications (`integration_spec.rs`), and instance binding subsystems.

## Design
- **Kernel / Supervisor Architecture**: Coordinates all child agent processes, managing their execution environments, terminal multiplexer allocations, virtual PTY screens, and inter-process messaging loops.
- **Dynamic Command Dispatcher**: `router.rs` dynamically parses CLI arguments without heavy CLI frameworks for high-frequency commands, delegating to `commands::*`, `hooks::*`, `tui`, or `pty`.
- **Agent Lifecycle State Machine**: `instance_lifecycle.rs` transitions agent instances across explicit states (`launching` → `listening` ↔ `active` / `blocked` → `inactive`), tracking process liveness via `pidtrack.rs`.
- **Prompt Injection & Capability Bootstrapping**: `bootstrap.rs` synthesizes tool-specific prompt instructions and skill definitions that teach launched AI agents how to discover peers and exchange messages using `hcom`.
- **Unified Delivery Engine**: `delivery.rs` implements the background delivery loop for PTY-wrapped tools, injecting pending messages via TCP and acknowledging delivery via database cursor progression.

## Key Root Modules
- `main.rs`: Process entry point, panic hook registration, and top-level router invocation.
- `router.rs`: Central routing table directing CLI subcommands, hook invocations, PTY mode, and TUI launches.
- `launcher.rs`: Spawns AI agents inside PTY wrappers across terminal multiplexers (`tmux`, `zellij`, `kitty`, `wezterm`, native windows).
- `delivery.rs`: Long-running message delivery loop that inspects screen readiness and injects pending messages.
- `bootstrap.rs`: Generates agent system prompts and hook configurations.
- `instance_binding.rs`: Dynamically maps OS PIDs, tmux panes, and tool session IDs to `hcom` instance names.
- `instance_lifecycle.rs`: Oversees instance creation, status updates, heartbeats, and termination cleanup.
- `integration_spec.rs`: Static registry containing tool schemas, launch flags, hook points, and capabilities for 11 AI coding tools.
- `identity.rs`: Resolves caller identities (human, autonomous agent, subagent, daemon).
- `config.rs`: Loads configuration files and environment variable overrides (`$HCOM_DIR`).

## Flow
1. **Startup**: `main.rs` initializes `Config`, registers file-logging panic hooks, and delegates to `router::dispatch()`.
2. **Launch Workflow**: `launcher::launch` parses requested tool from `integration_spec.rs`, selects terminal preset, sets up environment variables (`runtime_env.rs`), binds instance name (`instance_names.rs`), and forks the tool inside `pty::run_pty`.
3. **Delivery Workflow**: `delivery.rs` starts a background thread waiting on `notify::NotifyServer`; upon waking, queries `HcomDb` for unread messages, inspects `ScreenState`, injects text via TCP, and advances `last_event_id`.
4. **Hook & Event Ingestion**: As the agent acts, hooks update `instance_lifecycle` and persist audit logs to SQLite.

## Integration
- **Consumed by**: End-user terminal sessions, launched AI coding agents, and external scripts.
- **Submodules**: `src/commands/`, `src/core/`, `src/db/`, `src/delivery/`, `src/hooks/`, `src/notify/`, `src/pty/`, `src/relay/`, `src/scripts/`, `src/shared/`, `src/sys/`, `src/tools/`, `src/transcript/`, `src/tui/`.
