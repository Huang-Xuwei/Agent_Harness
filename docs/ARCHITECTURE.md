# Architecture

Agent Harness is organized around five layers.

## 1. Entry Layer

Users can enter through CLI or Gateway adapters. Platform-specific messages are normalized into `MessageEvent` objects with a `SessionSource`, then routed by `GatewayRunner`.

## 2. Agent Loop

The loop sends messages and tool schemas to an OpenAI-compatible model. When the model returns `tool_calls`, the harness dispatches each call, appends `tool` messages, and continues until the model returns a final answer or the iteration budget is exhausted.

## 3. Tool and Intelligence Layer

`ToolRegistry` stores tool metadata and handlers. Memory, skills, permissions, delegation, MCP, browser, voice, and vision capabilities are all exposed as tools or tool-adjacent services.

## 4. Execution Layer

`BaseExecutionEnvironment` defines a common command execution contract. Local, Docker, and SSH backends share command wrapping, timeout handling, environment snapshotting, and CWD tracking.

## 5. Persistence Layer

SQLite stores sessions and messages. Files under `HERMES_HOME` store memory, skills, allowlists, cache artifacts, and profile configuration.

## Self-Evolution Pipeline

Runtime loop:

```text
messages -> background review -> memory/skill write -> prompt injection
```

Offline optimization loop:

```text
skill -> synthetic eval dataset -> judge scoring -> feedback
      -> mutate skill -> validate constraints -> select -> deploy
```

The project evolves behavior by improving context, memory, and skill instructions. It does not modify model weights.
