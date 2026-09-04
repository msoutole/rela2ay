# src/relay/

## Responsibility
Distributed Mesh and Cross-Device Synchronization Subsystem. Implements an end-to-end encrypted (E2EE) event replication engine over MQTT (using `rumqttc` with TLS), allowing agents across different machines and networks to exchange messages, sync instance states, and dispatch remote control directives.

## Design
- **Pub/Sub Replication Model**: Synchronizes state across MQTT topics (`{relay_id}/{device_uuid}` for retained state, `{relay_id}/control` for non-retained signals).
- **Authenticated Symmetric Encryption**: `crypto.rs` encrypts payloads with ChaCha20-Poly1305 using key derivation from cluster pairing tokens.
- **Autonomous Worker Daemon**: `worker.rs` manages the MQTT client lifecycle, automatic reconnects with jittered backoff, and heartbeat tracking in SQLite KV.
- **Replay Defense & Deduplication**: `replay.rs` utilizes an LRU cache and monotonic sequence counters to eliminate retransmitted or replayed network packets.
- **Pairing Token Protocol**: `token.rs` handles base64 serialization of cluster IDs, cryptographic keys, and pinned MQTT broker endpoints.

## Flow
1. **Cluster Pairing**: Initiator runs `hcom relay link`, generating an encrypted pairing token; joiner runs `hcom relay join <token>`.
2. **Worker Daemon Startup**: Spawns `relay_worker` background process, establishing mutual TLS connection to configured broker (`rumqttc`).
3. **Outbound Push**: `push::push_events` queries un-replicated local events from SQLite, encrypts the payload, and publishes to the relay topic.
4. **Inbound Pull**: `pull::pull_events` receives remote MQTT messages, verifies authenticity, decrypts content, and writes to local SQLite `events` and `instances`.
5. **Heartbeat & Failover**: Periodically refreshes `relay_worker_heartbeat`; marks remote instances stale if silent for >90s.

## Integration
- **Consumed by**: `src/commands/relay.rs`, `src/commands/daemon.rs`, `src/tui/`.
- **Depends on**: `rumqttc`, `chacha20poly1305`, `rustls`, `lru`, `src/db/`, `src/config.rs`.

