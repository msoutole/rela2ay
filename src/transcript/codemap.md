# src/transcript/

## Responsibility
Agent Session Transcript Parsing and Normalization Engine. Parses, reconstructs, and formats conversation histories, tool execution outputs, prompt turns, and diffs from heterogeneous tool storage formats (JSONL logs, JSON trees, SQLite databases) across all supported AI coding agents.

## Design
- **Parser Strategy Pattern**: Concrete parsers per agent (`claude.rs`, `codex.rs`, `copilot.rs`, `cursor.rs`, `gemini.rs`, `kimi.rs`, `opencode.rs`, `pi.rs`) accommodate disparate vendor log schemas.
- **Canonical Transcript Schema**: `shared.rs` and `mod.rs` normalize disparate events into a common hierarchy of turns, tool calls, model thoughts, and user prompts.
- **Multilevel Projection**: Employs detail level projections (`concise`, `normal`, `full`, `raw`) to scale output volume for human review or LLM context consumption.

## Flow
1. **Resolution**: `hcom transcript <instance>` identifies the instance's tool type, session ID, and log directory on disk.
2. **Parsing**: The tool's transcript parser streams and decodes local transcript artifacts (e.g. Claude's `transcript.jsonl` or Codex session logs).
3. **Turn Reconstruction**: Collates streaming tokens, tool inputs, and tool results into coherent chronological dialogue turns.
4. **Formatting**: Applies detail filters and terminal/markdown rendering for display or bundling.

## Integration
- **Consumed by**: `src/commands/transcript.rs`, `src/commands/resume.rs`, `src/core/bundles.rs`, `src/tui/render/messages.rs`.
- **Depends on**: `src/core/detail_levels.rs`, `src/shared/`, `serde_json`, `regex`.

