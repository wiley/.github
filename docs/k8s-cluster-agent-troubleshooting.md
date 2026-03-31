# Troubleshooting Kubernetes Clusters with Cursor Agent

This guide explains how to use Cursor's AI agent to troubleshoot Kubernetes clusters
using MCP (Model Context Protocol) servers.

## Quick Start

1. Ensure `kubectl` is configured and can reach your cluster (`kubectl cluster-info`).
2. Copy the provided `.cursor/mcp.json` into your project (already included in this repo).
3. Open Cursor, start an Agent session, and ask it to diagnose your cluster.

The agent will automatically invoke the MCP server tools to inspect pods, read logs,
check events, and more — no manual `kubectl` commands required.

## Available MCP Server Options

There are three well-maintained Kubernetes MCP servers. Each has different trade-offs.

### Option 1 — `kubernetes-mcp-server` (Recommended)

| | |
|---|---|
| **Package** | [containers/kubernetes-mcp-server](https://github.com/containers/kubernetes-mcp-server) |
| **Language** | Go (distributed as a single native binary; also available via npx) |
| **Dependencies** | None — talks directly to the Kubernetes API server |
| **Best for** | Production troubleshooting, multi-cluster setups, OpenShift |

**Key capabilities:**

- Multi-cluster support (switch contexts within a single session)
- Pod operations: list, get, delete, logs, exec
- Generic CRUD on any Kubernetes resource kind
- Helm integration (install, list, uninstall releases)
- OpenTelemetry observability (optional)

**Configuration** (`.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "kubernetes": {
      "command": "npx",
      "args": ["-y", "kubernetes-mcp-server@latest"]
    }
  }
}
```

### Option 2 — `kubectl-mcp-server`

| | |
|---|---|
| **Package** | [rohitg00/kubectl-mcp-server](https://github.com/rohitg00/kubectl-mcp-server) |
| **Language** | Node.js |
| **Dependencies** | Requires `kubectl` on PATH |
| **Best for** | Deep diagnostics — 270+ tools including security scanning, RBAC validation |

**Key capabilities:**

- 270+ tools, 8 resources, 8 prompts
- Security scanning and RBAC validation
- Diagnostic analysis with error-pattern identification
- Deployment scaling, rollouts, event monitoring

**Configuration** (`.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "kubectl": {
      "command": "npx",
      "args": ["-y", "kubectl-mcp-server"],
      "env": {
        "KUBECONFIG": "/path/to/your/kubeconfig"
      }
    }
  }
}
```

### Option 3 — `mcp-server-kubernetes`

| | |
|---|---|
| **Package** | [Flux159/mcp-server-kubernetes](https://github.com/Flux159/mcp-server-kubernetes) |
| **Language** | Node.js |
| **Dependencies** | Requires `kubectl` (and optionally Helm v3) |
| **Best for** | Simple setups, quick one-off inspections |

**Configuration** (`.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "kubernetes": {
      "command": "npx",
      "args": ["-y", "mcp-server-kubernetes"]
    }
  }
}
```

## Prerequisites

All options require:

- **Node.js v20+** (for `npx` execution)
- **A valid kubeconfig** — by default `~/.kube/config`; override with the `KUBECONFIG`
  environment variable in the MCP server config
- **Network access** from your machine to the Kubernetes API server

Option 2 and 3 additionally require `kubectl` to be installed and on your PATH.

## How It Works

```
You (Cursor Agent chat)
  │
  ▼
Cursor Agent  ──MCP protocol──▶  MCP Server  ──K8s API / kubectl──▶  Cluster
  │                                    │
  ◀────── tool results ───────────────┘
```

1. You describe the problem in natural language (e.g. "Pods in namespace `payments` are
   crash-looping — help me figure out why").
2. The Cursor agent selects and calls the appropriate MCP tools (list pods, get events,
   read logs, describe resources, etc.).
3. Results flow back through MCP and the agent synthesizes a diagnosis.

## Example Prompts for Troubleshooting

Once the MCP server is configured, try asking the agent:

- "List all pods that are not in Running state across all namespaces."
- "Show me the logs for the last restart of pod `api-server-xyz` in namespace `backend`."
- "Why is the deployment `checkout-service` not reaching its desired replica count?"
- "Check if there are any pending PersistentVolumeClaims and explain why they're stuck."
- "Run a security scan on namespace `production` and summarize findings."
- "Show recent events in namespace `default` and flag any warnings."

## Configuration Scope

| Scope | File location | Use case |
|-------|---------------|----------|
| **Project** | `.cursor/mcp.json` (committed to repo) | Team-shared cluster config |
| **Global** | `~/.cursor/mcp.json` | Personal clusters across all projects |

Project-level settings override global settings when both define the same server name.

## Troubleshooting the MCP Server Itself

| Symptom | Fix |
|---------|-----|
| Agent doesn't use K8s tools | Restart Cursor after adding/changing `mcp.json` |
| "Server not found" errors | Run the `npx` command manually in a terminal to verify it starts |
| Auth / permission errors | Verify `kubectl cluster-info` works in the same shell environment |
| Tools time out | Check network connectivity to the API server; increase timeout if behind a VPN |
| Logs needed | In Cursor: **Cmd+Shift+U** (macOS) / **Ctrl+Shift+U** (Windows/Linux) → select **MCP Logs** |
