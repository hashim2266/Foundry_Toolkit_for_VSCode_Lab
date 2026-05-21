# Module 0 - Prerequisites

Lab 02 builds directly on Lab 01. Complete Lab 01 first before starting here.

---

## 1. Complete Lab 01

You should have already:

- Finished all 8 modules of [Lab 01 - Single Agent](../../lab01-single-agent/README.md)
- Deployed a single agent to Foundry Agent Service
- Verified it works in Agent Inspector and Foundry Playground

Haven't done Lab 01 yet? Start here: [Lab 01 Prerequisites](../../lab01-single-agent/docs/00-prerequisites.md)

---

## 2. Quick verification

Run these checks to confirm everything from Lab 01 is still working.

### Azure CLI

```powershell
az account show --query "{name:name, id:id}" --output table
```

If this fails, run `az login`.

### Foundry extension

Press `Ctrl+Shift+P` → type **Microsoft Foundry** → you should see commands like **Create a New Hosted Agent**.

### Your Foundry project and model

1. Click the **Microsoft Foundry** icon in the VS Code Activity Bar.
2. Your project is listed and your deployed model shows status **Succeeded**.

![Foundry sidebar showing project and deployed model with Succeeded status](images/00-foundry-sidebar-project-model.png)

If your model expired, redeploy it: `Ctrl+Shift+P` → **Microsoft Foundry: Open Model Catalog**.

### RBAC

You already assigned **Azure AI User** in Lab 01. If you need to verify or re-assign it, see [Lab 01, Module 2](../../lab01-single-agent/docs/02-create-foundry-project.md).

---

## 3. What's new in Lab 02

| | Lab 01 | Lab 02 |
|--|--------|--------|
| Agents | 1 | 4 (chained with WorkflowBuilder) |
| Scaffold wizard | Basic - Agent Framework | Same - identical wizard flow |
| New package | - | `mcp` |
| Key concept | Single conversational agent | Parallel + sequential multi-agent workflow |

The scaffold wizard steps in Module 2 are **exactly the same** as Lab 01. The difference is what you put in `main.py` after scaffolding.

---

**Next:** [01 - Understand the Architecture →](01-understand-multi-agent.md)
