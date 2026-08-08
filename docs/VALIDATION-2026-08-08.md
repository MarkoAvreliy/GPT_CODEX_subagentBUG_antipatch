# Validated lifecycle checkpoint — 2026-08-08

This is a temporary, source-built repair for a Codex Desktop subagent lifecycle
defect. It is not an official OpenAI release or a permanent fork.

## Pinned runtime

- Upstream source commit: `618b8e9111da9f57fe380b09d0f6516e3f343536`
- Codex version: `0.147.0-alpha.6.5`
- Patched executable SHA-256:
  `06BDE4920EAF39EE5424DC6EEB0E54E347EAA5A877F60C8D92CBC04E2CFB2588`
- Platform: Windows Desktop

Any upstream update invalidates this validation until the same smoke tests pass
again.

## What is repaired

- Read-only history resume does not initialize MCP transports or startup tool
  prewarm.
- The first real foreground action activates deferred MCP through the normal
  runtime boundary.
- Terminal V2 child runtimes unload through the existing shutdown path while
  logical identity and terminal status remain observable.
- Durable terminal rollout state repairs a stale legacy V1 open edge without
  resuming the child runtime.
- V2 descendants are not blindly recreated merely because an old parent task
  is displayed.

MCP, plugins, browser control, computer use, and Node tools are not removed. The
orchestrator retains its configured capabilities; lightweight children use
deny-by-default profiles and specialist children receive only the required tool
family.

## Controlled validation result

Four historical parent tasks were opened and allowed to load:

- new Python processes: **0**
- new Node processes: **0**
- new `node_repl` processes: **0**
- model requests caused by navigation: **0 observed**
- MCP starts caused by navigation: **0 observed**
- child spawn/resume events caused by navigation: **0 observed**

A separate runtime audit confirmed the exact patched app-server path and hash
with no Python, Node, or `node_repl` descendants.

Focused tests passed:

- `completed_agent_runtime_unloads_but_terminal_status_remains_observable`
- `residency_slot_reservation_unloads_oldest_idle_v2_agent`
- `resume_agent_from_rollout_does_not_reopen_v2_descendants`
- `completed_legacy_child_is_repaired_without_resuming_runtime`

The three `lazy_thread_resume_` integration tests also passed. They verify that
historical resume tolerates an unavailable required MCP executable, that the
first real turn starts the deferred stdio MCP normally, and that hydrating many
histories materializes only the runtime that receives real work.

Formatting, diff-integrity, and PowerShell parser checks also passed.

The consolidated source patch contains 14 files, applies cleanly to the pinned
upstream commit, and has SHA-256
`93813F18E93E26BBB291E563D891108B889089DFD05540BC28971FE5EA7C9BE8`.

## Lightweight children

Lifecycle repair and tool routing are separate controls:

- the primary orchestrator can keep its full MCP/plugin surface;
- ordinary children use deny-by-default profiles and bounded assignments;
- specialist children receive only the matching document, memory, knowledge,
  browser, computer-use, or other tool family;
- initial soak concurrency remains capped at three workers.

For MultiAgent V2, select the intended `agent_type` and use
`fork_turns = "none"`. For legacy V1, select the intended `agent_type` and use
`fork_context = false`.

## Remaining boundary

Some completed historical children can still render as `Working` until selected
in the UI. This is a separate state-projection defect. During the controlled
test, those badges did not materialize runtimes or trigger model work.

Automatic self-restart from inside the Desktop process/job tree was unreliable
and is not supported. Launch the patched runtime from an independent shortcut
or terminal.

## References

- [Official Codex subagent documentation](https://learn.chatgpt.com/docs/agent-configuration/subagents)
- [OpenAI Codex issue #37042](https://github.com/openai/codex/issues/37042)
- [OpenAI Codex issue #33700](https://github.com/openai/codex/issues/33700)
- [OpenAI Codex issue #37299](https://github.com/openai/codex/issues/37299)

Do not publish local Codex databases, histories, logs, credentials,
configuration, screenshots, or compiled binaries.
