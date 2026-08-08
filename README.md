# GPT_CODEX_subagentBUG_antipatch

## Codex Desktop subagent lifecycle — validated experimental checkpoint

This public, sanitized checkpoint records a temporary source-level repair for a
Codex Desktop lifecycle defect: reopening a historical task with completed
subagents must not recreate child runtimes, MCP transports, Node helpers, or
model work.

This is **not an official OpenAI fix**, a distributable Desktop build, or a
permanent Codex fork.

## Current status

- Validated against upstream commit `618b8e9111da9f57fe380b09d0f6516e3f343536`
  and Codex `0.147.0-alpha.6.5` on Windows.
- Read-only history hydration defers MCP/tool startup until a real foreground
  action.
- Completed V2 child runtimes unload while terminal status and explicit resume
  remain available.
- Terminal rollout state repairs stale legacy V1 open edges without recreating
  the child runtime.
- Four controlled historical-task opens produced zero new Python, Node,
  `node_repl`, MCP-start, child-spawn, or observed model-request events.
- A stale `Working` badge can remain a separate UI projection issue; it must not
  own or recreate runtime resources.

The patch does not disable MCP, plugins, browser control, computer use, or Node
tools. The primary orchestrator keeps its configured capabilities. Lightweight
children use deny-by-default profiles, while specialist children receive only
the tool family required by their assignment.

See [the validation record](docs/VALIDATION-2026-08-08.md),
[the compact checkpoint](CHECKPOINT.md), and
[the patch notes](docs/PATCH-NOTES.md).

## Safety

Do not commit local Codex state, task histories, databases, logs, screenshots,
credentials, user configuration, or compiled binaries. Rebuild and revalidate
after every upstream Codex update.
