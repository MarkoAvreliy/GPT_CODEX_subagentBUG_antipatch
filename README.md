# GPT_CODEX_subagentBUG_antipatch

## Codex Desktop Subagent Lifecycle — Experimental Checkpoint

Public, sanitized checkpoint for investigation of a Codex Desktop lifecycle defect: completed subagents from an old task can appear as `Working` again and can trigger unnecessary local runtime processes when that task is reopened.

This repository is **not an official OpenAI fix** and is not a distributable Codex build. It records a small experimental anti-patch and the evidence-based next steps so the work can be reviewed and continued without sharing personal configuration, logs, binaries, or credentials.

## Current status

- The affected local histories are predominantly `multi_agent_version: v2`, even when their storage mode is named `legacy`; the earlier V1-only diagnosis was insufficient.
- The current patch defers MCP startup while an old task is merely displayed and restores it on the first real foreground action.
- The current patch unloads a completed V2 child runtime through the normal shutdown path while retaining its logical identity and terminal status.
- Stale `Working` rendering remains a separate UI/state-reconciliation defect; it must not force runtime startup.
- The patch does **not** disable or remove MCP, plugins, browser control, computer use, or Node-based tools. They remain available and start on a real action that needs them.
- Lightweight children are a routing/configuration concern: ordinary children receive a minimal profile, while specialist children receive only the tool family required by their assignment. The primary orchestrator can retain its full configured toolset.

See [the minimal root-fix](docs/ROOT-FIX.md), [the compact checkpoint](CHECKPOINT.md), and [patch notes](docs/PATCH-NOTES.md).

## Safety

Do not copy local Codex state, session databases, logs, environment files, binaries, or user configuration into this repository. The patch is experimental and must be rebuilt and tested against the exact upstream source version before use.
