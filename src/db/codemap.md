# src/db/

## Responsibility
Persistence Layer and Data Access Object (DAO) Engine. Manages the embedded SQLite database (`db.sqlite3`) housing three distinct operational state planes: live agent instances (`instances`), immutable audit and message event logs (`events`), and control plane state (`process_bindings`, `session_bindings`, `notify_endpoints`, `kv`, `subscriptions`, `claude_actor_capabilities`).

## Design
- **Repository / DAO Pattern**: Specific data access logic is modularized across submodules:
  - `instances.rs`: Agent status, heartbeat, PID bindings, terminal presets.
  - `events.rs`: Append-only event store and message querying.
  - `sessions.rs`: Session and PID-to-instance mappings.
  - `notify.rs`: Registered wake notification endpoints.
  - `subscriptions.rs`: Topic-based pub/sub routing.
  - `kv.rs`: Key-value configuration and cluster state.
  - `claude_actors.rs`: Ephemeral sub-agent capability tokens.
- **Append-Only Event Sourcing**: Inter-agent messages and status changes are recorded as immutable event rows with sequential monotonic IDs and cursor tracking.
- **Automated Migration Pipeline**: Self-healing schema management executes linear version migrations up to schema version 18.
- **WAL Concurrency**: Utilizes SQLite Write-Ahead Logging (WAL) and busy timeout configurations for multi-process concurrent access.

## Flow
1. **Connection Initialization**: `HcomDb::open` initializes connection, enforces WAL mode and foreign keys, checks version pragma, and executes outstanding migrations.
2. **Message Ingestion**: When `hcom send` or an agent hook executes, `HcomDb::insert_event` persists the message payload to `events`.
3. **Cursor-based Retrieval**: Target agent's delivery loop calls `HcomDb::get_unread_messages(instance_name)`, filtering events greater than the instance's `last_event_id`.
4. **Cursor Advance**: After successful injection/dispatch, `HcomDb::advance_instance_cursor` atomically updates `instances.last_event_id`.

## Integration
- **Consumed by**: `src/commands/`, `src/delivery.rs`, `src/hooks/`, `src/relay/`, `src/tui/`, `src/pty/`.
- **Depends on**: `rusqlite`, `serde_json`, `chrono`, `anyhow`.

