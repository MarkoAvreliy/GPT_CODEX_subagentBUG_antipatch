# Checkpoint — 2026-08-08

## Result

The temporary lifecycle repair is operationally validated for one pinned
Windows build. Opening the tested completed historical parent tasks no longer
materialized child MCP/helper runtimes or caused observed model work.

Pinned inputs:

- upstream source: `618b8e9111da9f57fe380b09d0f6516e3f343536`;
- Codex version: `0.147.0-alpha.6.5`;
- patched executable SHA-256:
  `06BDE4920EAF39EE5424DC6EEB0E54E347EAA5A877F60C8D92CBC04E2CFB2588`.

## Root failure and repair

The investigation separated two defects:

1. a completed child can be projected as `Working` from persisted/UI state;
2. history restoration can incorrectly recreate or retain the runtime behind
   that stale logical state.

The patch prevents read-only history hydration from initializing MCP/tool
runtime, hydrates durable terminal child state without loading the child,
unloads completed V2 runtimes through the normal shutdown path, and repairs
terminal legacy V1 edges without eager resume. Explicit future resume and the
first real tool-requiring action remain supported.

## Validation

Four historical parent tasks were opened and allowed to hydrate. New Python,
Node, and `node_repl` processes were all zero. Monitored MCP starts, child
spawn/resume events, and model requests caused by navigation were also zero.

The consolidated 14-file source patch applies cleanly to the pinned upstream
commit. Its SHA-256 is
`93813F18E93E26BBB291E563D891108B889089DFD05540BC28971FE5EA7C9BE8`.

## Remaining boundary

The packaged UI can still show a stale `Working` badge until the row is selected
and reconciled. That presentation defect is not considered proof of active
runtime or token use. Revalidate using process-tree and model-transport evidence.

Automatic self-restart from inside the Desktop process/job tree is not
supported. Start the patched runtime from an independent shortcut or terminal.

This checkpoint excludes personal paths, task history, state databases, logs,
credentials, screenshots, and binaries.
