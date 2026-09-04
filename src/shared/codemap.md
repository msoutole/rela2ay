# src/shared/

## Responsibility
Cross-cutting Foundation Library and Domain Primitive Layer. Provides common types, domain constants, context holders, cross-platform environment detectors, terminal preset definitions, and error taxonomies used across the entire binary.

## Design
- **Value Object Pattern**: Domain-specific primitives such as `SenderIdentity`, `SenderKind`, `TitleMode`, and status identifiers (`ST_ACTIVE`, `ST_BLOCKED`, etc.) are modeled with strong typing and serialization support.
- **Context Object Pattern**: `HcomContext` acts as an encapsulation unit bundling database access (`HcomDb`), configuration (`Config`), and caller identity for CLI and hook operations.
- **Facade & Strategy Patterns**: `platform.rs` and `tool_detection.rs` abstract platform and environment checks (WSL, Termux, active AI tool heuristics) behind unified query methods.
- **Unified Error Hierarchy**: `errors.rs` defines typed domain errors (`HcomError`, `HookError`, `CLIError`) using `thiserror`.

## Flow
1. **Tool & Platform Detection**: Upon command startup or hook invocation, `tool_detection::detect_current_tool_from_env` or `platform::detect_current_tool_from_env` inspects parent process trees, environment variables, and markers.
2. **Context Resolution**: Commands construct an `HcomContext` using active credentials, environment paths, and database handles.
3. **Mention & Message Normalization**: When processing agent messages, `constants::extract_mentions` parses `@target` tokens using `MENTION_PATTERN`, enforcing `MAX_MESSAGE_SIZE` bounds.
4. **Terminal Formatting**: `terminal_presets.rs` and `ansi.rs` supply styling rules and window title templates depending on terminal emulator type.

## Integration
- **Consumed by**: `src/commands/`, `src/hooks/`, `src/delivery.rs`, `src/launcher.rs`, `src/router.rs`, `src/tui/`, and `src/pty/`.
- **Depends on**: Standard library, `rusqlite`, `serde`, `regex`, `chrono`.

