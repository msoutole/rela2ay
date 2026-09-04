# src/core/

## Responsibility
Core Domain Logic and Query Processing Engine. Encapsulates business validation, query filter generation (compiling CLI filter expressions to SQL clauses), batch launch tracking state machines, context bundle exchange protocols, and detail level definitions.

## Design
- **Query Builder / Interpreter Pattern**: `filters.rs` compiles parsed CLI filter arguments (e.g. `--from`, `--to`, `--action`, `--since`, `--unseen`) into dynamic SQL expressions and bound parameter vectors.
- **Specification Pattern**: `helpers.rs` enforces domain rules regarding message scope, agent intent, mentions format, and group routing.
- **Data Transfer Object (DTO) & Bundling**: `bundles.rs` structures multi-file context exchanges between agents, including bundle validation and checksum verification.
- **State Machine**: `launch_status.rs` coordinates multi-agent launch status polling (`Launching` → `Listening` → `Ready` / `Failed`).

## Flow
1. **Filtering**: User or agent invokes `hcom events` or `hcom listen` with filter flags → `filters::EventFilter::from_args` parses flags → `to_sql_clause` produces SQL predicate → executed against `HcomDb`.
2. **Launch Coordination**: Batch agent launcher invokes `launch_status::wait_for_launch`, polling instance status and timeout deadlines.
3. **Bundling**: Agents call bundle utilities to package files and git diffs into standardized context payloads for other agents.

## Integration
- **Consumed by**: `src/commands/events.rs`, `src/commands/listen.rs`, `src/commands/launch.rs`, `src/commands/bundle.rs`, `src/tui/`.
- **Depends on**: `src/db/`, `src/shared/`, `rusqlite`, `serde_json`, `regex`.

