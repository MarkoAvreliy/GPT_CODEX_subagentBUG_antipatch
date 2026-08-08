# Local verification - 2026-08-08

This document records a sanitized local verification of the experimental Codex Desktop lifecycle patch. It is evidence for a temporary workaround, not an official OpenAI release or a claim that every UI defect is fixed.

## Target

- Upstream source commit: `618b8e9`
- Codex version: `0.147.0-alpha.6.5`
- Patched Windows executable SHA-256: `6D9EC41990406B99850C1E417563C49993A002A3F5D4607F8A1D419136C23067`
- Experimental flags: lazy MCP activation on historical resume and automatic unload of completed child runtimes after a five-second grace period

OpenAI's current subagent documentation says that Codex owns spawning, waiting for, and closing agent threads, and that the UI should separate Active and Done children. It also documents per-agent configuration files and inherited configuration. See [OpenAI: Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents).

## Result

The tested build passed the process-lifecycle acceptance criteria:

| Scenario | Observation | Result |
| --- | --- | --- |
| Open one completed heavy historical task three times | No new Python, Node, `node_repl`, MCP startup, child spawn, or model turn for the historical task | Pass |
| Open a second completed heavy historical task | No new runtime process; only history/shell-state reconstruction was observed | Pass |
| Manually open four old tasks and wait for each UI view to load | 13 samples over about 35 seconds; maximum Python, Node, and `node_repl` descendants was zero | Pass |
| Keep the shared HTTP MCP pool enabled during the old-task test | The same eight Windows wrapper/worker PIDs and the same listeners on ports 8101-8103 remained stable | Pass |
| Spawn a general lightweight V2 child | Explicit `agent_type="default"` plus `fork_turns="none"`; zero tool calls and zero MCP events | Pass |
| Spawn a memory specialist V2 child | Explicit `agent_type="memory"` plus `fork_turns="none"`; exactly one `brain-memory.whereami` call and no unrelated MCP call | Pass |
| Observe both real children through completion | 52 process samples over about 68 seconds; no Python, Node, or `node_repl` descendants and no duplicated shared MCP server | Pass |

The old UI can still initially paint some completed children as `Working` and flip them to `Done` when selected. In the tested build this stale presentation did not materialize a runtime or send a model turn for the old task. UI reconciliation remains a separate defect.

## Child-context finding

MultiAgent V2 defaults `fork_turns` to full history. A custom `agent_type` cannot enforce its child profile on that full-history path because the child inherits the parent agent type/configuration. The reliable lightweight contract for this version is therefore:

```text
spawn_agent(
  agent_type = "default|worker|explorer|documents|memory|knowledge|browser|computer",
  fork_turns = "none",
  task = "a bounded assignment containing only the required context slice"
)
```

For the legacy V1 surface, the equivalent is an explicit `agent_type` with `fork_context=false`.

This removed the parent transcript from the child start. The first measured initial model input fell from 204,499 tokens in the full orchestrator turn (181,248 cached) to about 25,000 tokens in each clean child start (11,008 cached).

A second child-only configuration pass disabled Codex's built-in memory injection and automatic global skills-instructions catalog through `features.memories=false`, `memories.use_memories=false`, and `skills.include_instructions=false`. It did not alter the primary orchestrator. A general child then started with 14,712 input tokens, and a memory specialist started with 14,693 before its one MCP result. This is approximately 93% below the measured orchestrator turn. Mandatory permissions, team-agent, project `AGENTS.md`, environment, and tool schemas remain. Cached input is reported separately and these measurements must not be interpreted as exact subscription billing.

## Source-level lifecycle checks

Two focused `codex-core --lib` tests passed:

1. `completed_agent_runtime_unloads_but_terminal_status_remains_observable`
2. `residency_slot_reservation_unloads_oldest_idle_v2_agent`

The first proves that terminal child runtime is removed from the live thread manager while `Completed` remains queryable. The second proves that the residency cap can evict the oldest idle child rather than rejecting useful new work. On this Windows host the second test required a larger test-thread stack through `RUST_MIN_STACK`; its logic then passed.

## Operational boundary

- The primary orchestrator may retain its full configured toolset.
- Ordinary children use deny-by-default profiles and receive no heavy optional MCP/plugin bundle.
- Specialist children receive only the one tool family required by the assignment.
- Child profiles omit the global skills catalog and built-in Codex memory injection; the primary orchestrator retains both.
- Initial concurrency remains capped at three during soak testing.
- The patch does not disable MCP, plugins, browser control, computer use, or Node tools.
- Opening history is read-only and must not activate deferred MCP; a real new action may activate the tools normally.

## Rollback and revalidation

The patched runtime is a separate executable and does not replace the installed Desktop binary. Roll back by closing the patched Desktop runtime and starting the official application normally.

Re-enable the subagent circuit breaker and repeat the short old-history plus two-child smoke test after any Codex update, source-commit change, executable-hash change, or when running the stock binary. Do not assume this patch applies to a newer internal lifecycle implementation.

## Evidence handling

Raw process captures, account logs, session rollouts, user paths, and local configuration remain private and are intentionally not committed. The table above is the sanitized result derived from those artifacts.
