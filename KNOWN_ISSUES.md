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

## KI-005 - Teams/M365 Publish Succeeds but Agent Unreachable (BotId/ClientId Mismatch)

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

## KI-006 - No Multi-Agent Wizard Template in AITK v1.2.1

**Status:** Open | **Severity:** 🟡 Medium | **Type:** Feature gap

### Description

The AITK wizard (Foundry Toolkit v1.2.1) provides the following templates under the **Response API** path:

- Echo (Streaming)
- Multi-Turn Chat
- Note Taking
- **Basic - Agent Framework** ← used in Lab 01

There is no **Multi-Agent Workflow** template. Lab 02 uses the pre-built **`PersonalCareerCopilot/`** folder rather than a wizard-generated scaffold.

### Workshop impact

Learners cannot use the wizard to scaffold a new multi-agent project using MAF GA. Lab 02 proceeds using the pre-existing example code directly.

---

## References

- [agent-framework-core on PyPI](https://pypi.org/project/agent-framework-core/)
- [agent-framework-foundry on PyPI](https://pypi.org/project/agent-framework-foundry/)
- [microsoft/agent-framework#5273](https://github.com/microsoft/agent-framework/issues/5273) - agentserver incompatibility with MAF GA (closed)
- [Azure/azure-sdk-for-python#46324](https://github.com/Azure/azure-sdk-for-python/issues/46324) - SDK fix pending (open)
- [microsoft/vscode-ai-toolkit#381](https://github.com/microsoft/vscode-ai-toolkit/issues/381) - Teams/M365 publish + auth (open)
