# Checkpoint — 2026-08-07

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

Focused source tests now pass for both behaviors. A real Desktop A/B against historical tasks is still required before calling the patch operationally verified.

## What remains

- Build the pinned Windows executable and run the real Desktop A/B.
- Verify that UI `Working`/`Done` state does not reactivate runtime.
- Measure process trees and model-request logs separately; a badge or local process alone is not proof of token use.
- Verify deny-by-default child profiles and narrow document, memory, knowledge, browser, and computer-use specialist profiles without reducing the primary orchestrator's capabilities.
- Rebuild and run UI end-to-end tests against historical task data.

## Scope of this repository

This is a public engineering checkpoint only. It intentionally excludes personal paths, Codex state databases, MCP configuration, credentials, logs, screenshots, compiled binaries, and user task history.
