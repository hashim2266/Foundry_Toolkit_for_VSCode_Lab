# Known Issues

This document tracks known issues with the current repository state.

> Last updated: 2026-05-21. Tested against Python 3.13 / Windows. Foundry Toolkit v1.2.1.

---

## Current Package Pins

### Lab 01 (`workshop/lab01-single-agent/agent/`)

| Package | Constraint |
|---------|------------|
| `agent-framework` | `>=1.1.0` |
| `agent-framework-foundry-hosting` | latest |
| `debugpy` | latest |

### Lab 02 (`workshop/lab02-multi-agent/PersonalCareerCopilot/`)

| Package | Constraint |
|---------|------------|
| `agent-framework` | `>=1.1.0` |
| `agent-framework-foundry-hosting` | latest |
| `debugpy` | latest |
| `mcp` | latest |

---

## KI-001 - Teams/M365 Publish Succeeds but Agent Unreachable (BotId/ClientId Mismatch)

**Status:** Open | **Severity:** 🔴 High | **Type:** Integration bug | **Ref:** [vscode-ai-toolkit#381](https://github.com/microsoft/vscode-ai-toolkit/issues/381)

### Description

Publishing a Hosted Agent to Teams/M365 Copilot via Foundry Toolkit completes successfully but the agent is **not reachable** on those channel surfaces.

Root cause: **Publish** (registers bot channel artifacts) and **Deploy** (starts live hosted runtime) are two separate operations. Without Deploy, no running backend exists behind the endpoint.

Error observed when Bot Service attempts to route traffic:

```
Failed to publish agent BotId '<id>' does not match the application's default instance identity ClientId '<id>'.
[Status: 400, Code: UserError]
```

Azure Bot Service sends Bot Framework JWTs (`iss=https://api.botframework.com`). The current Foundry gateway auth path does not fully accept this token shape in this configuration.

**Deploy blocker:** Deploy Hosted Agent requires a local Docker build. No cloud-side build fallback is currently offered.

### Workaround

Test using the **web chat** channel in Azure Bot Service - this path works correctly. Teams/M365 channel integration is blocked until upstream auth and UX issues are resolved.

---

## KI-002 - No Dedicated Multi-Agent Template in Foundry Toolkit v1.2.1

**Status:** Open | **Severity:** 🟡 Medium | **Type:** Feature gap

### Description

The Foundry Toolkit v1.2.1 wizard flow is identical for both Lab 01 and Lab 02: **Language → API Type → Template → Model → Workspace/Folder**. The template step offers:

- Echo (Streaming)
- Multi-Turn Chat
- Note Taking
- **Basic - Agent Framework** ← used in both Lab 01 and Lab 02

There is no dedicated multi-agent template. The scaffold always generates a **single-agent stub** (`Agent` + `ResponsesHostServer`, no `WorkflowBuilder` code).

### Workshop impact

Lab 02 learners scaffold with **Basic - Agent Framework** (same as Lab 01), then replace the generated `main.py` stub with the full `WorkflowBuilder` graph (four agents, MCP tool, fan-out/fan-in edges). Lab 02 therefore points learners at the pre-built **`PersonalCareerCopilot/`** folder as the complete reference implementation.

---

## References

- [Azure/azure-sdk-for-python#46324](https://github.com/Azure/azure-sdk-for-python/issues/46324) - SDK fix pending (open)
- [microsoft/vscode-ai-toolkit#381](https://github.com/microsoft/vscode-ai-toolkit/issues/381) - Teams/M365 publish + auth (open)
