# src/tools/

## Responsibility
Tool-Specific Argument Preprocessing and Validation Layer. Normalizes, sanitizes, and adapts command-line arguments and configuration switches when launching target AI coding agents (`codex`, `copilot`, `cursor`, `opencode`, etc.) before process execution.

## Design
- **Strategy Pattern**: Implements dedicated preprocessors per tool (`codex_preprocessing.rs`, `copilot_preprocessing.rs`, `cursor_preprocessing.rs`, `opencode_preprocessing.rs`) to accommodate divergent CLI parameter syntaxes.
- **Validation Pipeline**: `launch_arg_validation.rs` verifies that requested model configurations, agent names, and execution modes are valid and non-conflicting.
- **Command Rewriter**: Translates high-level flags (e.g., `--subagent`, `--profile`, `--model`) into underlying native tool invocation parameters.

## Flow
1. **Invocation**: Agent or user runs `hcom <tool> [args...]` or `hcom launch`.
2. **Validation**: `launch_arg_validation` ensures valid instance naming and parameter constraints.
3. **Tool Preprocessing**: The matching tool preprocessor manipulates the argument vector, injecting required hook configs, authentication files, and system parameters.
4. **Execution Handoff**: Sanitized argument vector is forwarded to `src/launcher.rs` for PTY wrapper or process spawning.

## Integration
- **Consumed by**: `src/launcher.rs`, `src/commands/launch.rs`, `src/router.rs`.
- **Depends on**: `src/shared/`, `src/integration_spec.rs`, `clap`.

