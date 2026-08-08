# Experimental Patch Notes

## Goal

Reduce the expensive side effect of opening historical Codex tasks: local MCP/runtime processes should not start merely because old child-agent state is being displayed or restored.

## Patch behavior under evaluation

### 1. Lazy MCP initialization on resume

The experimental resume path records that MCP may be needed, but delays starting the MCP manager until a new action actually requires it. This separates **history hydration** from **tool process startup**.

### 2. Cleanup of completed active child runtimes

An experimental watcher identifies a child whose terminal response is available, waits through a short grace period, then unloads the active runtime. Terminal status is cached so the UI can still render the child as completed.

## Why this is not the full repair

The legacy controller may persist child spawn records as open. The resume code can walk those open records and recreate descendants even when a child previously returned a final answer. Lazy MCP startup reduces process impact, but it does not correct that persisted lifecycle state or guarantee correct UI status.

## Required acceptance tests

1. Reopen a completed historical parent task repeatedly: no child runtime, MCP, or helper process should start until a new user action needs it.
2. Open a parent task with completed children: each child should render as terminal without rehydrating its runtime.
3. Run lightweight child workers: after completion and the cleanup grace period, no residual runtime processes remain.
4. Run an MCP-specialist child: its server starts only for its requested operation and exits when the child lifecycle ends.
5. Confirm that a newly created task with no child activity has no behavioral regression.

## Compatibility and safety

- This work targets a source-built experimental Codex Desktop runtime only.
- Do not patch installed application binaries in place.
- Revalidate after every upstream Codex update; internal APIs and feature flags may change.
- Do not enable unfinished multi-agent variants globally as a workaround.
