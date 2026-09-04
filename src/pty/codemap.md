# src/pty/

## Responsibility
Pseudo-Terminal Emulation, Screen State Virtualization, and Input Injection Subsystem. Wraps child agent processes inside a PTY (POSIX `openpty` on Unix or ConPTY on Windows), maintains a live `vt100` virtual screen model to detect prompt readiness, and exposes a TCP injection server for remote keystroke and text insertion.

## Design
- **Proxy Pattern**: The `Proxy` intercepts I/O between the host terminal and the child agent process, transparently forwarding output while updating internal terminal state.
- **Virtual Terminal Tracker**: `screen::ScreenTracker` leverages `vt100` to mirror cursor location, line buffers, alternate screen modes, and terminal prompts.
- **TCP Injection Gateway**: `inject::InjectServer` runs an internal TCP server translating received network commands into PTY master input writes.
- **Cross-Platform Driver**: Isolates POSIX termios/ioctl operations (`terminal.rs`) from Windows pseudo-console handles (`win.rs`).

## Flow
1. **Spawn**: `hcom pty <tool>` creates a PTY pair, configures window size, and forks the child agent process.
2. **I/O Forwarding**: The PTY proxy multiplexes user keyboard input, child stdout/stderr, and injection socket connections.
3. **Screen Analysis**: Child output streams through `ScreenTracker` to maintain live screen state and detect prompt readiness heuristics.
4. **Text Injection**: External callers connect to the injection port; `InjectServer` validates state and writes UTF-8 input directly into the PTY stream.
5. **Teardown**: On process termination, `TerminalGuard` restores original terminal settings.

## Integration
- **Consumed by**: `src/main.rs::run_pty`, `src/launcher.rs`, `src/delivery.rs`, `src/commands/term.rs`.
- **Depends on**: `vt100`, `nix`, `crossterm`, `src/sys/`.

