# src/sys/

## Responsibility
Platform Abstraction Layer (Hardware/OS Abstraction Layer). Isolates all operating system-specific primitives (POSIX/Unix system calls via `nix` and `libc` versus Windows Win32 APIs via `windows-sys`) behind uniform, platform-neutral interfaces for file locking, signal handling, process lifecycle, and socket I/O.

## Design
- **Port & Adapter (Abstraction Barrier)**: Replaces scattered `#[cfg(unix)]` / `#[cfg(windows)]` checks across the codebase with single-boundary modules (`fs.rs`, `process.rs`, `signal.rs`, `net.rs`, `io.rs`).
- **RAII Resource Management**: Implements guards such as `FileLock` to guarantee advisory lock release across POSIX `flock` and Win32 `LockFileEx`.
- **Process Hierarchy Traverser**: `process.rs` traverses PID trees, PGIDs, and Windows Job Objects for robust agent tree termination.

## Flow
1. **Locking**: Callers call `fs::FileLock::acquire(path, nonblocking)`, which applies `flock` (Unix) or `LockFileEx` (Windows) and returns a scoped RAII guard.
2. **Process Lifecycle**: When agents terminate or are killed via `hcom kill`, `process::kill_process_tree(pid)` recursively discovers child processes and sends escalation signals (SIGTERM → SIGKILL or Win32 `TerminateProcess`).
3. **Signals**: `signal::setup_signal_handler` registers signal listeners for terminal resize (SIGWINCH) and termination interrupts.

## Integration
- **Consumed by**: `src/pty/`, `src/launcher.rs`, `src/instance_lifecycle.rs`, `src/pidtrack.rs`, `src/notify/`, and `src/commands/kill.rs`.
- **Depends on**: Unix (`nix`, `libc`, `signal-hook`), Windows (`windows-sys`), Rust standard library `std::process`.

