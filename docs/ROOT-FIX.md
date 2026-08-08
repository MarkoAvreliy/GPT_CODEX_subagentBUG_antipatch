# Minimal lifecycle root-fix

Status date: 2026-08-08. This is a temporary experiment against OpenAI Codex commit `618b8e9` (`0.147.0-alpha.6.5`), not a replacement application or permanent fork.

The patch changes only the legacy V1 subagent lifecycle:

1. When a child reaches a terminal state, its persisted `thread_spawn_edge` is changed from `Open` to `Closed`.
2. When an old parent task is restored, Codex inspects an `Open` child's durable rollout before constructing the child runtime.
3. If the rollout already ends in a terminal lifecycle event, Codex repairs the stale edge to `Closed` and does not resume that child.
4. If the rollout cannot be read, Codex preserves the original resume behavior instead of guessing that the child is complete.

The focused regression test simulates the historical defect by reopening an edge after the child's terminal event, shutting down the tree, and resuming the parent. The expected result is a repaired `Closed` edge and no loaded child runtime.

This patch deliberately excludes the earlier lazy-MCP and delayed runtime-unload experiments. Those mitigations can hide symptoms; this checkpoint tests the smaller root-cause repair first.

The GitHub Actions workflow builds only a testable `codex.exe` from pinned upstream source. It does not rebuild or redistribute the ChatGPT/Codex Desktop shell.
