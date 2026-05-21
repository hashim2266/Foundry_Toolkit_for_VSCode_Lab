# Module 8 - Troubleshooting (Multi-Agent)

This module covers common errors, fixes, and debugging strategies specific to the multi-agent workflow. For general Foundry deployment issues, also refer to the [Lab 01 troubleshooting guide](../../lab01-single-agent/docs/08-troubleshooting.md).

---

## Quick reference: Error → Fix

| Error / Symptom | Likely Cause | Fix |
|----------------|-------------|-----|
| `KeyError: 'AZURE_AI_PROJECT_ENDPOINT'` | `.env` file missing or values not set | Create `.env` with `AZURE_AI_PROJECT_ENDPOINT=<your-endpoint>` and `MODEL_DEPLOYMENT_NAME=<your-model>` |
| `ModuleNotFoundError: No module named 'agent_framework'` | Virtual environment not activated or dependencies not installed | Run `.\.venv\Scripts\Activate.ps1` then `pip install -r requirements.txt` |
| `ModuleNotFoundError: No module named 'mcp'` | MCP package not installed | Add `mcp` to `requirements.txt` then run `pip install -r requirements.txt` |
| Agent starts but returns empty response | `output_executors` mismatch or missing edges | Verify `output_executors=[gap_executor]` and all edges exist in `WorkflowBuilder` |
| Only 1 gap card (rest missing) | GapAnalyzer instructions incomplete | Add the `CRITICAL:` paragraph to `GAP_ANALYZER_INSTRUCTIONS` - see [Module 3](03-configure-agents.md) |
| Fit score is 0 or absent | MatchingAgent didn't receive upstream data | Verify both `add_edge(resume_executor, matching_executor)` and `add_edge(jd_executor, matching_executor)` exist |
| `POST https://learn.microsoft.com/api/mcp → 4xx/5xx` | MCP server rejected the tool call | Check internet connectivity. Try opening `https://learn.microsoft.com/api/mcp` in browser. Retry |
| No Microsoft Learn URLs in output | MCP tool not registered or endpoint wrong | Verify `tools=[search_microsoft_learn_for_plan]` on GapAnalyzer and `MICROSOFT_LEARN_MCP_ENDPOINT` is correct |
| `Address already in use: port 8088` | Another process is using port 8088 | Run `netstat -ano \| findstr :8088` (Windows) or `lsof -i :8088` (macOS/Linux) and stop the conflicting process |
| `Address already in use: port 5679` | Debugpy port conflict | Stop other debug sessions. Run `netstat -ano \| findstr :5679` to find and kill the process |
| Agent Inspector won't open | Server not fully started or port conflict | Wait for "Server running" log. Check port 5679 is free |
| `azure.identity.CredentialUnavailableError` | Not signed into Azure CLI | Run `az login` then restart the server |
| `azure.core.exceptions.ResourceNotFoundError` | Model deployment doesn't exist | Check `MODEL_DEPLOYMENT_NAME` matches a deployed model in your Foundry project |
| Container status "Failed" after deployment | Container crash on startup | Check container logs in Foundry sidebar. Common: missing env var or import error |
| Deployment shows "Pending" for > 5 minutes | Container taking too long to start or resource limits | Wait up to 5 minutes for multi-agent (creates 4 agent instances). If still pending, check logs |
| `ValueError` from `WorkflowBuilder` | Invalid graph configuration | Ensure `start_executor` is set, `output_executors` is a list, and no circular edges |

---

## Environment and configuration issues

### Missing or wrong `.env` values

The `.env` file must be in the `PersonalCareerCopilot/` directory (same level as `main.py`):

```
PersonalCareerCopilot/
├── .env                  ← Must be here
├── main.py
├── agent.yaml
├── Dockerfile
└── requirements.txt
```

Expected `.env` content:

```env
AZURE_AI_PROJECT_ENDPOINT=https://<your-project-name>.services.ai.azure.com/api/projects/<your-project-id>
MODEL_DEPLOYMENT_NAME=gpt-4.1-mini
```

> **Finding your `AZURE_AI_PROJECT_ENDPOINT`:** 
- Open the **Microsoft Foundry** sidebar in VS Code → right-click your project → **Copy Project Endpoint**. 
- Or go to [Azure Portal](https://portal.azure.com) → your Foundry project → **Overview** → **Project endpoint**.

> **Finding your MODEL_DEPLOYMENT_NAME:** In the Foundry sidebar, expand your project → **Models** → find your deployed model name (e.g., `gpt-4.1-mini`).

### Env var precedence

`main.py` uses `load_dotenv(override=True)`, which means:

| Priority | Source | Wins when both are set? |
|----------|--------|------------------------|
| 1 (highest) | `.env` file | Yes (`override=True`) |
| 2 | Shell environment variable | Only if not in `.env` |

In hosted deployment, Foundry sets env vars at the container level (via `agent.yaml`). Set `override=False` if you want container env vars to take precedence.

---

## Version compatibility

### Package version matrix

The multi-agent workflow requires specific package versions. Mismatched versions cause runtime errors.

| Package | Required Version | Check Command |
|---------|-----------------|---------------|
| `agent-framework` | `>=1.1.0` | `pip show agent-framework` |
| `agent-framework-foundry-hosting` | latest | `pip show agent-framework-foundry-hosting` |
| `mcp` | latest | `pip show mcp` |
| `debugpy` | latest | `pip show debugpy` |
| Python | 3.10+ | `python --version` |

### Common version errors

**`ImportError: cannot import name 'WorkflowBuilder' from 'agent_framework'`**

```powershell
# Fix: upgrade agent-framework
pip install "agent-framework>=1.1.0" agent-framework-foundry-hosting
```

**`AttributeError: module 'mcp.client' has no attribute 'streamable_http'`**

```powershell
# Fix: upgrade mcp package
pip install mcp --upgrade
```

### Verify all versions at once

```powershell
pip list | Select-String "agent-framework|mcp|debugpy"
```

Expected output:

```
agent-framework                  1.1.x
agent-framework-foundry-hosting  x.x.x
debugpy                          x.x.x
mcp                              x.x.x
```

---

## MCP tool issues

### MCP tool returns no results

**Symptom:** Gap cards say "No results returned from Microsoft Learn MCP" or "No direct Microsoft Learn results found".

**Possible causes:**

1. **Network issue** - The MCP endpoint (`https://learn.microsoft.com/api/mcp`) is unreachable.
   ```powershell
   # Test connectivity
   Invoke-WebRequest -Uri "https://learn.microsoft.com/api/mcp" -Method POST -UseBasicParsing
   ```
   If this returns `200`, the endpoint is reachable.

2. **Query too specific** - The skill name is too niche for Microsoft Learn search.
   - This is expected for very specialized skills. The tool has a fallback URL in the response.

3. **MCP session timeout** - The Streamable HTTP connection timed out.
   - Retry the request. MCP sessions are ephemeral and may need reconnection.

### MCP logs explained

```
GET https://learn.microsoft.com/api/mcp → 405 (Method Not Allowed)
POST https://learn.microsoft.com/api/mcp → 200
DELETE https://learn.microsoft.com/api/mcp → 405 (Method Not Allowed)
```

| Log | Meaning | Action |
|-----|---------|--------|
| `GET → 405` | MCP client probes during initialization | Normal - ignore |
| `POST → 200` | Tool call succeeded | Expected |
| `DELETE → 405` | MCP client probes during cleanup | Normal - ignore |
| `POST → 400` | Bad request (malformed query) | Check the `query` parameter in `search_microsoft_learn_for_plan()` |
| `POST → 429` | Rate limited | Wait and retry. Reduce `max_results` parameter |
| `POST → 500` | MCP server error | Transient - retry. If persistent, the Microsoft Learn MCP API may be down |
| Connection timeout | Network issue or MCP server unavailable | Check internet. Try `curl https://learn.microsoft.com/api/mcp` |

---

## Deployment issues

### Container fails to start after deployment

1. **Check container logs:**
   - Open the **Microsoft Foundry** sidebar → expand **Hosted Agents (Preview)** → click your agent → expand the version → **Container Details** → **Logs**.
   - Look for Python stack traces or missing module errors.

2. **Common container startup failures:**

   | Error in logs | Cause | Fix |
   |--------------|-------|-----|
   | `ModuleNotFoundError` | `requirements.txt` missing a package | Add the package, redeploy |
   | `KeyError: 'AZURE_AI_PROJECT_ENDPOINT'` | `agent.yaml` or `.env` env vars not set | Update `agent.yaml` → `environment_variables` section (hosted) or `.env` (local) |
   | `azure.identity.CredentialUnavailableError` | Managed Identity not configured | Foundry sets this automatically - ensure you're deploying via the extension |
   | `OSError: port 8088 already in use` | Dockerfile exposes wrong port or port conflict | Verify `EXPOSE 8088` in Dockerfile and `CMD ["python", "main.py"]` |
   | Container exits with code 1 | Unhandled exception in `main()` | Test locally first ([Module 5](05-test-locally.md)) to catch errors before deploying |

3. **Redeploy after fixing:**
   - `Ctrl+Shift+P` → **Microsoft Foundry: Deploy Hosted Agent** → select the same agent → deploy a new version.

### Deployment takes too long

Multi-agent containers take longer to start because they create 4 agent instances on startup. Normal startup times:

| Stage | Expected duration |
|-------|------------------|
| Container image build | 1-3 minutes |
| Image push to ACR | 30-60 seconds |
| Container start (single agent) | 15-30 seconds |
| Container start (multi-agent) | 30-120 seconds |
| Agent available in Playground | 1-2 minutes after "Started" |

> If "Pending" status persists beyond 5 minutes, check container logs for errors.

---

## RBAC and permission issues

### `403 Forbidden` or `AuthorizationFailed`

You need the **[Azure AI User](https://aka.ms/foundry-ext-project-role)** role on your Foundry project:

1. Go to [Azure Portal](https://portal.azure.com) → your Foundry **project** resource.
2. Click **Access control (IAM)** → **Role assignments**.
3. Search for your name → confirm **Azure AI User** is listed.
4. If missing: **Add** → **Add role assignment** → search for **Azure AI User** → assign to your account.

See the [RBAC for Microsoft Foundry](https://learn.microsoft.com/azure/foundry/concepts/rbac-foundry) documentation for details.

### Model deployment not accessible

If the agent returns model-related errors:

1. Verify the model is deployed: Foundry sidebar → expand project → **Models** → check for `gpt-4.1-mini` (or your model) with status **Succeeded**.
2. Verify the deployment name matches: compare `MODEL_DEPLOYMENT_NAME` in `.env` (or `agent.yaml`) with the actual deployment name in the sidebar.
3. If the deployment expired (free tier): redeploy from [Model Catalog](https://learn.microsoft.com/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure) (`Ctrl+Shift+P` → **Microsoft Foundry: Open Model Catalog**).

---

## Agent Inspector issues

### Inspector opens but shows "Disconnected"

1. Verify the server is running: check for "Server running on http://localhost:8088" in the terminal.
2. Check port `5679`: Inspector connects via debugpy on port 5679.
   ```powershell
   netstat -ano | findstr :5679
   ```
3. Restart the server and reopen Inspector.

### Inspector shows partial response

Multi-agent responses are long and stream incrementally. Wait for the full response to complete (may take 30-60 seconds depending on the number of gap cards and MCP tool calls).

If the response is consistently truncated:
- Check the GapAnalyzer instructions have the `CRITICAL:` block that prevents combining gap cards.
- Check your model's token limit - `gpt-4.1-mini` supports up to 32K output tokens, which should be sufficient.

---

## Performance tips

### Slow responses

Multi-agent workflows are inherently slower than single-agent because of sequential dependencies and MCP tool calls.

| Optimization | How | Impact |
|-------------|-----|--------|
| Reduce MCP calls | Lower `max_results` parameter in the tool | Fewer HTTP round-trips |
| Simplify instructions | Shorter, more focused agent prompts | Faster LLM inference |
| Use `gpt-4.1-mini` | Faster than `gpt-4.1` for development | ~2x speed improvement |
| Reduce gap card detail | Simplify the gap card format in GapAnalyzer instructions | Less output to generate |

### Typical response times (local)

| Configuration | Expected time |
|--------------|---------------|
| `gpt-4.1-mini`, 3-5 gap cards | 30-60 seconds |
| `gpt-4.1-mini`, 8+ gap cards | 60-120 seconds |
| `gpt-4.1`, 3-5 gap cards | 60-120 seconds |

---

## Getting help

If you're stuck after trying the fixes above:

1. **Check the server logs** - Most errors produce a Python stack trace in the terminal. Read the full traceback.
2. **Search the error message** - Copy the error text and search in the [Microsoft Q&A for Azure AI](https://learn.microsoft.com/answers/tags/azure-ai-services).
3. **Open an issue** - File an issue on the [workshop repository](https://github.com/ShivamGoyal03/ai-toolkit-hosted-agents-workshop/issues) with:
   - The error message or screenshot
   - Your package versions (`pip list | Select-String "agent-framework"`)
   - Your Python version (`python --version`)
   - Whether the issue is local or after deployment

---

### Checkpoint

- [ ] You can identify and fix the most common multi-agent errors using the quick reference table
- [ ] You know how to check and fix `.env` configuration issues
- [ ] You can verify package versions match the required matrix
- [ ] You understand MCP log entries and can diagnose tool failures
- [ ] You know how to check container logs for deployment failures
- [ ] You can verify RBAC roles in the Azure Portal

---

**Previous:** [07 - Verify in Playground](07-verify-in-playground.md) · **Home:** [Lab 02 README](../README.md) · [Workshop Home](../../../README.md)
