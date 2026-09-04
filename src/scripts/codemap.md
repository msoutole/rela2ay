# src/scripts/

## Responsibility
Embedded Script Repository and Packaging Directory. Contains custom and pre-packaged orchestration scripts that are compiled into the binary or distributed alongside `hcom`.

## Design
- **Compile-Time Asset Embedding**: Scripts located in subdirectories (such as `bundled/`) are embedded into the executable binary via `include_str!` in `src/scripts.rs`.
- **Recipe Management**: Provides out-of-the-box workflows for multi-agent tasks, debate sessions, and codebase verification.

## Flow
1. Scripts residing here are referenced by `src/scripts.rs`.
2. The runtime looks up bundled scripts when the user invokes `hcom run <name>`.
3. The selected script is written to an ephemeral path or piped directly into a subshell for execution.

## Integration
- **Consumed by**: `src/scripts.rs`, `src/commands/run.rs`.
- **Depends on**: `src/scripts/bundled/`.

