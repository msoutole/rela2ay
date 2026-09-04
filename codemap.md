# Repository Atlas: hcom (rela2ay)

## Project Responsibility
`hcom` is a high-performance inter-agent communication, coordination, and synchronization system for AI coding tools. It connects Claude Code, Gemini CLI, OpenAI Codex, OpenCode, Kilo Code, Pi, Oh My Pi, Antigravity, Cursor, Kimi, and GitHub Copilot, enabling disparate agent processes to send direct messages, subscribe to events, monitor terminal states, and spawn each other across local terminals and distributed machines.

## System Entry Points
- `src/main.rs`: Native Rust binary entry point; initializes configuration and dispatches commands via `router.rs`.
- `Cargo.toml`: Crate definition, feature flags, dependencies (`rusqlite`, `ratatui`, `crossterm`, `rumqttc`, `vt100`, `nix`, `windows-sys`).
- `package.json`: TypeScript tool dependencies for in-process extensions (`@oh-my-pi/pi-coding-agent`, `@opencode-ai/plugin`, `@earendil-works/pi-coding-agent`).
- `Justfile`: Development workflow recipes (build, check, test, lint, format).
- `install.sh`: Shell installer script for cross-platform distribution.
- `gemini-extension.json`: Extension descriptor for Gemini CLI integration.
- `plugin/hcom/.claude-plugin/plugin.json`: Plugin descriptor for Claude Code integration.
- `skills/hcom-agent-messaging/SKILL.md`: Agent self-discovery skill teaching LLMs how to use `hcom`.

## Repository Directory Map
| Directory | Responsibility Summary | Detailed Map |
| :--- | :--- | :--- |
| `src/` | Core application kernel, configuration, launcher engine, prompt bootstrapping, and process coordination. | [View Map](src/codemap.md) |
| `src/commands/` | CLI command controllers for messaging (`send`, `listen`), lifecycle (`launch`, `kill`), diagnostics, and config. | [View Map](src/commands/codemap.md) |
| `src/core/` | Core domain query compiler (`filters`), validation rules, batch launch state machines, and context bundles. | [View Map](src/core/codemap.md) |
| `src/db/` | Embedded SQLite DAO engine managing agent instances, append-only event logs, sessions, and subscriptions. | [View Map](src/db/codemap.md) |
| `src/delivery/` | Tool-specific delivery adapters and teardown handlers (e.g. Antigravity session-end reconciliation). | [View Map](src/delivery/codemap.md) |
| `src/hooks/` | Tool lifecycle interception middleware capturing events from Claude, Gemini, Codex, Cursor, and other agent CLIs. | [View Map](src/hooks/codemap.md) |
| `src/hooks/omp/` | Dedicated hook adapter and extension script provisioner for Oh My Pi (`omp`). | [View Map](src/hooks/omp/codemap.md) |
| `src/notify/` | Inter-process signaling subsystem using ephemeral TCP connect-and-drop pings to wake sleeping poll loops. | [View Map](src/notify/codemap.md) |
| `src/omp_plugin/` | In-process TypeScript extension running natively inside Oh My Pi for prompt and tool event interception. | [View Map](src/omp_plugin/codemap.md) |
| `src/opencode_plugin/` | In-process TypeScript plugin integrating with OpenCode's SDK for session observation and message injection. | [View Map](src/opencode_plugin/codemap.md) |
| `src/pi_plugin/` | In-process TypeScript extension for Pi coding agent providing prompt hooks and TCP wake listeners. | [View Map](src/pi_plugin/codemap.md) |
| `src/pty/` | Cross-platform PTY wrapper (Unix `openpty` / Windows ConPTY), `vt100` virtual screen tracking, and TCP injection. | [View Map](src/pty/codemap.md) |
| `src/relay/` | Distributed MQTT event replication mesh with ChaCha20-Poly1305 end-to-end encryption for multi-device sync. | [View Map](src/relay/codemap.md) |
| `src/scripts/` | Embedded script catalog and launcher integration for multi-agent recipes. | [View Map](src/scripts/codemap.md) |
| `src/scripts/bundled/` | Production multi-agent orchestration recipes (`confess.sh`, `debate.sh`, `fatcow.sh`). | [View Map](src/scripts/bundled/codemap.md) |
| `src/shared/` | Shared domain primitives, constants, context structures, environment detection, and error definitions. | [View Map](src/shared/codemap.md) |
| `src/sys/` | Platform abstraction layer isolating POSIX/Unix (`nix`, `libc`) and Windows (`windows-sys`) system APIs. | [View Map](src/sys/codemap.md) |
| `src/tools/` | Tool-specific CLI argument validation, flag translation, and preprocessing for target agent CLIs. | [View Map](src/tools/codemap.md) |
| `src/transcript/` | Multi-vendor transcript parser extracting dialogue turns, tool executions, and diffs across agent log formats. | [View Map](src/transcript/codemap.md) |
| `src/tui/` | Real-time interactive terminal monitoring dashboard built with Ratatui and Crossterm. | [View Map](src/tui/codemap.md) |
| `src/tui/inline/` | Non-destructive inline viewport manager that streams new events into terminal scrollback history. | [View Map](src/tui/inline/codemap.md) |
| `src/tui/input/` | Keyboard and mouse controller with modal dialog handlers and an interactive message composition buffer. | [View Map](src/tui/input/codemap.md) |
| `src/tui/render/` | Layout partitioner and widget rendering engine for agent tables, message logs, and themes. | [View Map](src/tui/render/codemap.md) |

## Core Data & Control Flows

### 1. Inter-Agent Message Delivery Flow
```text
[Agent A]
   │  hcom send @agent-b "Review PR"
   ▼
[SQLite DB: events table] ──── (Record unread message event)
   │
   ▼
[notify::wake] ─────────────── (Ephemeral TCP ping to Agent B's wake port)
   │
   ▼
[Agent B Delivery Loop / Hook]
   ├─► Check Screen Readiness (vt100 screen state != busy)
   ├─► Inject message into PTY stream / prompt context
   └─► Advance DB Cursor (instances.last_event_id)
```

### 2. Multi-Machine Relay Replication Flow
```text
[Local Machine]
   │  Local DB Event appended
   ▼
[relay::worker push] ────────► Encrypt (ChaCha20-Poly1305) ──► MQTT Broker (TLS)
                                                                     │
                                                                     ▼
[Remote Machine] ◄──────────── Decrypt & Deduplicate (LRU) ◄─────────┘
   │
   ▼
[Remote DB: events table] ────► notify::wake ──► Remote Agent Delivery
```

