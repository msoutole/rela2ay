# src/scripts/bundled/

## Responsibility
Bundled Multi-Agent Orchestration Recipes. Contains production-ready multi-agent collaboration and evaluation shell scripts embedded directly into the `hcom` binary via `include_str!`.

## Design
- **Autonomous Multi-Agent Patterns**:
  - `confess.sh`: Implements an agent honesty verification protocol (Confessor, Calibrator, Judge) based on introspection research.
  - `debate.sh`: Implements structured dialectical debate rounds between competing agents (PRO, CON) with a presiding Judge.
  - `fatcow.sh`: Establishes a long-running "codebase oracle" agent that memorizes a directory tree and answers queries from other peer agents.
- **Self-Cleaning Lifecycle**: Scripts maintain traps (`trap cleanup EXIT`) to reliably terminate spawned child instances and release system resources.

## Flow
1. User or agent executes `hcom run <recipe>` (e.g. `hcom run debate "Rust vs Go" --spawn`).
2. `src/commands/run.rs` extracts the embedded script, sets up execution environment and pipe descriptors.
3. The script coordinates agent lifecycles via `hcom launch`, `hcom send`, and `hcom listen`.
4. On exit or interrupt, the script terminates child agents via `hcom stop <agent>`.

## Integration
- **Consumed by**: `src/commands/run.rs`, `src/scripts.rs`.
- **Depends on**: `bash`, `hcom` CLI binary.

