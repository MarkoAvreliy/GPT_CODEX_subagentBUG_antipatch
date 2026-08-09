# Experimental patch notes — validated checkpoint

## Scope

Prevent a read-only open of historical Codex tasks from recreating completed
child runtimes, MCP transports, helper processes, or model work, while preserving
all normal tool capabilities for real new actions.

## Implemented behavior

1. Historical `thread/resume` defers MCP startup and startup tool prewarm.
2. The first real foreground action activates deferred MCP through the existing
   refresh boundary.
3. Terminal V2 children unload through the existing `shutdown_and_wait` path
   after a short grace period.
4. Logical child identity and terminal status remain observable after unload.
5. Durable terminal rollout state repairs stale legacy V1 open edges without
   resuming the child runtime.
6. V2 descendants are not blindly recreated while a historical parent is only
   being displayed.

## Acceptance results

- Repeated historical navigation: no Python, Node, `node_repl`, MCP-start,
  child-spawn/resume, or observed model-request events.
- Completed runtime unload keeps terminal status observable: passed.
- Residency pressure unloads the oldest idle V2 runtime: passed.
- V2 rollout resume does not reopen descendants: passed.
- Terminal legacy child repair does not resume runtime: passed.
- Patch applies cleanly to the pinned source and passes diff-integrity checks.

## Architecture boundary

The lifecycle patch does not remove MCP, plugins, browser control, computer use,
or Node tools. Tool routing is a separate configuration layer:

- the primary orchestrator retains its configured capabilities;
- ordinary children use a lightweight deny-by-default profile;
- specialist children receive only their matching tool family;
- bounded context is passed explicitly instead of full parent-history forks.

## Known limitation

Stale `Working` rendering can remain in the packaged UI. The validated invariant
is that this stale display state does not materialize a child runtime during
read-only historical navigation.

This patch is pinned to one upstream version. Re-run the old-history,
completed-child, tool-activation, and process-tree tests after every update.
