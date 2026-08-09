# Related startup-host hypothesis (unverified locally)

## Status

This is a **related external observation**, not a confirmed regression in this
checkpoint's validated environment. Do not change the lifecycle patch solely
because of this note.

On 2026-08-09, [openai/codex#37672](https://github.com/openai/codex/issues/37672)
reported a Windows Codex Desktop startup burst of Node and `node_repl` helpers.
The report attributes the burst to code-mode host plugin probing and says that
disabled plugin entries were still enumerated in that environment.

## Relationship to this checkpoint

The symptoms overlap, but the reported trigger differs:

| Case | Trigger | Current repair coverage |
| --- | --- | --- |
| This checkpoint | Opening historical parents/children could rehydrate completed child runtime | Covered: read-only history hydration stays passive |
| #37672 | Starting Desktop itself creates a large helper burst | Not covered unless reproduced locally |

A plausible shared downstream component is Code Mode / plugin helper startup:
historical child rehydration may cause an additional runtime initialization,
while #37672 reports that initial startup alone can initialize helpers in bulk.
This is a hypothesis, not proof of a common root cause.

## Evidence boundary

Do not infer model-token or subscription usage from helper-process counts alone.
Process growth establishes local resource use. Claim agent/model usage only when
the correlated logs show a model transport, agent task, or other request.

## Controlled check before any patch extension

1. Start the patched Desktop runtime and wait 2-3 minutes without opening a
   historical task or sending a message.
2. Capture the Codex process tree: parent PID, command line, exact executable,
   helper count, and working-set memory.
3. Open historical tasks separately and capture the same measurements.
4. Run one new lightweight child, then 2-3 independent lightweight children.
5. Compare deltas by trigger. Do not test the concurrency cap of six until the
   smaller fan-out remains stable.

Interpretation:

- Clean idle baseline plus clean historical navigation: keep the current patch
  and profile routing; #37672 is not reproduced locally.
- Large helper burst before any interaction: record it as a distinct startup
  defect and investigate code-mode host/plugin enumeration separately.
- Growth only after historical navigation: investigate lifecycle hydration and
  child-runtime ownership first.

## Related public reports

- [#37426](https://github.com/openai/codex/issues/37426): stale Working child
  state and inherited local MCP suite.
- [#37453](https://github.com/openai/codex/issues/37453): historical
  subagent-thread resume and duplicate MCP / `node_repl` stacks.
- [#37672](https://github.com/openai/codex/issues/37672): reported startup-host
  helper burst and plugin-enable handling.
