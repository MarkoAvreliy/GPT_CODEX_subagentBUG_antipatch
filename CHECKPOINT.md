# Checkpoint — 2026-08-07

## Observed problem

On Codex Desktop, reopening an older task that previously used many subagents may show already-finished children as `Working`. Opening the parent task, or selecting individual stale child entries, can coincide with local process activity such as MCP-backed Python workers or Node-based helper processes. This is undesirable even if the model-side task had already produced a final response.

The behavior is reproducible enough to treat it as a lifecycle/state-restoration defect, not a workflow preference issue.

## Evidence-backed hypothesis

The legacy multi-agent resume path can recursively restore child agents whose persisted spawn records are still marked open. That creates two related but distinct failures:

1. stale UI lifecycle state (`Working` instead of terminal state);
2. runtime rehydration when a historical parent task is opened.

MCP process multiplication is a consequence of runtime rehydration when heavy local tools are available; it is not, by itself, the root cause.

## Experimental anti-patch currently tested

- **Lazy resume of MCP:** do not start MCP servers solely because a historical task is being restored; initialize them after a real new action needs them.
- **Completed-child cleanup:** after a child reaches a terminal result, unload its active runtime after a short grace period while keeping the terminal status available to the UI.

In controlled smoke tests, this prevented residual Python/Node helper processes after lightweight child completion. It does **not** yet close or migrate old persisted spawn records, so it is not a permanent repair.

## What remains

- Persist or infer terminal closure for completed legacy children.
- Prevent recursive rehydration of terminal descendants during parent-task resume.
- Verify that UI `Working`/`Done` state is reconciled from durable lifecycle data.
- Isolate plugins and skills for lightweight child workers, not only heavy MCP servers.
- Rebuild and run UI end-to-end tests against historical task data.

## Scope of this repository

This is a public engineering checkpoint only. It intentionally excludes personal paths, Codex state databases, MCP configuration, credentials, logs, screenshots, compiled binaries, and user task history.
