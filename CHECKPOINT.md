# Checkpoint — 2026-08-08

## Observed problem

On Codex Desktop, reopening an older task that previously used many subagents may show already-finished children as `Working`. Opening the parent task, or selecting individual stale child entries, can coincide with local process activity such as MCP-backed Python workers or Node-based helper processes. This is undesirable even if the model-side task had already produced a final response.

The behavior is reproducible enough to treat it as a lifecycle/state-restoration defect, not a workflow preference issue.

## Evidence-backed hypothesis

Inspection of the affected rollouts corrected the earlier diagnosis: their storage mode is called `legacy`, but their actual agent lifecycle is V2. In V2, logical child identity remains resident after completion while heavy per-session runtime ownership is coupled to that identity. This creates two related but distinct failures:

1. stale UI lifecycle state (`Working` instead of terminal state);
2. unnecessary runtime/MCP rehydration when history is opened, plus completed V2 runtimes remaining resident.

MCP process multiplication is a consequence of runtime rehydration when heavy local tools are available; it is not, by itself, the root cause.

## Experimental anti-patch currently tested

- **Lazy resume of MCP:** do not start MCP servers solely because a historical task is being restored; initialize them after a real new action needs them.
- **V2 completed-child cleanup:** after a child reaches a terminal result, call the normal session shutdown path after a short grace period while keeping identity, history, and terminal status available.

Focused source tests pass for both behaviors. The patched Desktop runtime has now also passed repeated automated historical-task opens, a manual four-task UI load, and a real two-child profile smoke test without creating new Python, Node, `node_repl`, or duplicated shared MCP processes. See [the sanitized verification record](docs/VERIFICATION-2026-08-08.md).

## What remains

- Treat stale `Working`/`Done` rendering as a separate UI/state-reconciliation issue; it did not reactivate runtime in the verified patched build.
- Soak the reduced approximately 14.7k-token clean child baseline; mandatory safety, project, environment, and tool-schema context remains intentionally intact.
- Soak the bounded three-child workflow in normal work and repeat the smoke test after any Codex update.
- Upstream the minimal lifecycle findings and tests; retire this workaround when an official build passes the same acceptance criteria.

## Scope of this repository

This is a public engineering checkpoint only. It intentionally excludes personal paths, Codex state databases, MCP configuration, credentials, logs, screenshots, compiled binaries, and user task history.
