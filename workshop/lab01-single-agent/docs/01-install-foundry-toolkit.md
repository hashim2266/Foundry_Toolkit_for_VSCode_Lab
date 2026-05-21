# Module 1 - Install and Verify Foundry Toolkit

This module walks you through installing and verifying Foundry Toolkit (which includes Microsoft Foundry capabilities used throughout this workshop). If you already installed it during [Module 0](00-prerequisites.md), use this module to verify commands and views are working correctly.

---

## Step 1: Install Foundry Toolkit for VS Code

**Foundry Toolkit for VS Code** is the primary extension used in this workshop. It includes the Foundry workflows for creating projects, deploying models, scaffolding hosted agents, and deployment.

1. Open VS Code.
2. Press `Ctrl+Shift+X` to open the **Extensions** panel.
3. In the search box at the top, type: **Foundry Toolkit**
4. Look for the result titled **Foundry Toolkit for VS Code**.
   - Publisher: **Microsoft**
   - Extension ID: `ms-windows-ai-studio.windows-ai-studio`
5. Click the **Install** button.
6. Wait for the installation to complete (you'll see a small progress indicator).
7. After installation, look at the **Activity Bar** (the vertical icon bar on the left side of VS Code). You should see a **Foundry Toolkit** icon.
8. Open the toolkit views. You should see sections for:
   - **Resources** (or Projects)
   - **Agents**
   - **Models**

![Microsoft Foundry icon in VS Code Activity Bar with sidebar view showing Resources, Agents, and Models sections](images/01-foundry-sidebar-view.png)

> **If the icon doesn't appear:** Try reloading VS Code (`Ctrl+Shift+P` → `Developer: Reload Window`).

---

## Step 2: Verify Agent Inspector availability

Foundry Toolkit provides the [**Agent Inspector**](https://learn.microsoft.com/azure/foundry/agents/how-to/vs-code-agents-workflow-pro-code) - a visual interface for testing and debugging agents locally - plus playground, model management, and evaluation tools.

1. Open Command Palette (`Ctrl+Shift+P`) and search for **Foundry Toolkit**.
2. Confirm commands appear.
3. Open the Foundry Toolkit views and confirm the welcome screen loads with options for:
   - **Models**
   - **Playground**
   - **Agents**

---

## Step 3: Verify commands are working

### 3.1 Verify Foundry commands

1. Open Command Palette (`Ctrl+Shift+P`) and type **Microsoft Foundry**.
2. If you're signed into Azure (from Module 0), you should see your projects listed under **Resources**.
3. If prompted to sign in, click **Sign in** and follow the authentication flow.
4. Confirm you can see the sidebar without errors.

### 3.2 Verify Foundry Toolkit views

1. Click the **Foundry Toolkit** icon in the Activity Bar.
2. Confirm the welcome view or main panel loads without errors.
3. You don't need to configure anything yet - we'll use the Agent Inspector in [Module 5](05-test-locally.md).

### 3.3 Verify via Command Palette

1. Press `Ctrl+Shift+P` to open the Command Palette.
2. Type **"Microsoft Foundry"** - you should see commands like:
   - `Microsoft Foundry: Create a New Hosted Agent`
   - `Microsoft Foundry: Deploy Hosted Agent`
   - `Microsoft Foundry: Open Model Catalog`
3. Press `Escape` to close the Command Palette.
4. Open the Command Palette again and type **"Foundry Toolkit"** - you should see commands like:
   - `Foundry Toolkit: Open Agent Inspector`

![VS Code Command Palette showing Microsoft Foundry commands like Create new Hosted Agent and Deploy Hosted Agent](images/01-command-palette-foundry-commands.png)

> If you don't see these commands, the extensions may not be installed correctly. Try uninstalling and reinstalling them.

---

## What these extensions do in this workshop

| Capability | What it does | When you'll use it |
|-----------|-------------|-------------------|
| **Foundry Toolkit (incl. Foundry commands)** | Create Foundry projects, deploy models, **scaffold [hosted agents](https://learn.microsoft.com/azure/foundry/agents/concepts/hosted-agents)** (auto-generates `agent.yaml`, `main.py`, `Dockerfile`, `requirements.txt`), deploy to [Foundry Agent Service](https://learn.microsoft.com/azure/foundry/agents/overview) | Modules 2, 3, 6, 7 |
| **Agent Inspector** | Local testing/debugging, streaming output and workflow inspection | Modules 5, 7 |

> Foundry Toolkit provides the core end-to-end lifecycle in this workshop: scaffold → configure → deploy → verify.

---

### Checkpoint

- [ ] Foundry Toolkit icon is visible in the Activity Bar
- [ ] Clicking it opens the sidebar without errors
- [ ] `Ctrl+Shift+P` → typing "Microsoft Foundry" shows available commands
- [ ] `Ctrl+Shift+P` → typing "Foundry Toolkit" shows available commands

---

**Previous:** [00 - Prerequisites](00-prerequisites.md) · **Next:** [02 - Create Foundry Project →](02-create-foundry-project.md)