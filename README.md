# GPT_CODEX_subagentBUG_antipatch

## Codex Desktop Subagent Lifecycle — Experimental Checkpoint

Public, sanitized checkpoint for investigation of a Codex Desktop lifecycle defect: completed subagents from an old task can appear as `Working` again and can trigger unnecessary local runtime processes when that task is reopened.

This repository is **not an official OpenAI fix** and is not a distributable Codex build. It records a small experimental anti-patch and the evidence-based next steps so the work can be reviewed and continued without sharing personal configuration, logs, binaries, or credentials.

## Current status

- A local experimental build can defer MCP startup while an old task is merely being restored.
- A local experiment can unload a completed child runtime while retaining its terminal status for the UI.
- The main legacy defect is still unresolved: persisted child spawn records may remain open, and reopening a parent task can recursively rehydrate those children.
- Plugin and skill isolation for lightweight child workers remains a separate follow-up item.

See [the minimal root-fix](docs/ROOT-FIX.md), [the compact checkpoint](CHECKPOINT.md), and [patch notes](docs/PATCH-NOTES.md).

## Safety

Do not copy local Codex state, session databases, logs, environment files, binaries, or user configuration into this repository. The patch is experimental and must be rebuilt and tested against the exact upstream source version before use.
