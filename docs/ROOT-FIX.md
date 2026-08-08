# Minimal lifecycle root-fix

Status date: 2026-08-08. This is a temporary experiment against OpenAI Codex commit `618b8e9` (`0.147.0-alpha.6.5`), not a replacement application or permanent fork.

The patch targets the process-heavy V2 runtime lifecycle, which matches the affected local histories:

1. A read-only `thread/resume` restores history and metadata without starting MCP transports or startup tool prewarm.
2. The first real foreground action activates the deferred MCP runtime through the existing refresh boundary.
3. A detached watcher observes a V2 child reaching a terminal status and invokes the existing `shutdown_and_wait` path after a five-second grace period.
4. Only the heavy runtime is removed. Agent metadata, logical ID, terminal status, history, and explicit resume remain available.

The focused tests prove that a completed child runtime disappears while `Completed` remains observable, and that historical resume succeeds even when a configured required MCP executable is deliberately invalid.

This does not claim to repair every stale `Working` badge. UI reconciliation is separate from runtime ownership and is not allowed to materialize MCP or helper processes merely to display history.

The GitHub Actions workflow builds only a testable `codex.exe` from pinned upstream source. It does not rebuild or redistribute the ChatGPT/Codex Desktop shell.
