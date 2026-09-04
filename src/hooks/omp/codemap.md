# src/hooks/omp/

## Responsibility
Specialized Hook Adapter and Extension Provisioner for Oh My Pi (`omp`). Manages the installation lifecycle of the `omp` TypeScript extension and dispatches argv-based lifecycle events (`start`, `status`, `stop`).

## Design
- **Asset Provisioning Pattern**: `plugin.rs` embeds `src/omp_plugin/hcom.ts` at compile time and ensures dynamic installation/updates in the target `omp` extensions directory.
- **Argv Dispatcher**: `handlers.rs` decodes positional command-line arguments emitted by `omp` hook triggers into domain actions.

## Flow
1. **Provisioning**: During agent launcher setup, `ensure_omp_plugin_installed` verifies and writes the latest extension script to disk.
2. **Execution**: `omp` executes `hcom hook omp <action> [args]` on lifecycle events.
3. **Dispatch**: `dispatch_omp_hook` handles `start` (binds instance/session), `status` (updates active/idle state and TCP notify port), or `stop` (marks instance inactive).

## Integration
- **Consumed by**: `src/hooks/mod.rs`, `src/launcher.rs`.
- **Depends on**: `src/omp_plugin/`, `src/db/`, `src/shared/`, `src/notify/`.

