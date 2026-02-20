<!-- mcp-name: io.github.rhel-lightspeed/linux-mcp-server -->
[![CI](https://github.com/rhel-lightspeed/linux-mcp-server/actions/workflows/ci.yml/badge.svg)](https://github.com/rhel-lightspeed/linux-mcp-server/actions/workflows/ci.yml)
[![Coverage](https://codecov.io/gh/rhel-lightspeed/linux-mcp-server/graph/badge.svg?token=TtUkG1y0rx)](https://codecov.io/gh/rhel-lightspeed/linux-mcp-server)
[![PyPI](https://img.shields.io/pypi/v/linux-mcp-server?label=PyPI)](https://pypi.org/project/linux-mcp-server)
[![Docs](https://img.shields.io/badge/Docs-Linux%20MCP%20Server-red)](https://rhel-lightspeed.github.io/linux-mcp-server/)


# Linux MCP Server

A Model Context Protocol (MCP) server for read-only Linux system administration, diagnostics, and troubleshooting on RHEL-based systems.


## Features

- **Read-Only Operations**: All tools are strictly read-only for safe diagnostics
- **Remote SSH Execution**: Execute commands on remote systems via SSH with key-based authentication
- **Multi-Host Management**: Connect to different remote hosts in the same session
- **Comprehensive Diagnostics**: System info, services, processes, logs, network, and storage
- **Package Insights (DNF)**: Query packages, repositories, provides, groups, and modules
- **Configurable Log Access**: Control which log files can be accessed via environment variables
- **RHEL/systemd Focused**: Optimized for Red Hat Enterprise Linux systems


## Installation and Usage

For detailed instructions on setting up and using the Linux MCP Server, please refer to our official documentation:

- **[Installation Guide]**: Detailed steps for `pip`, `uv`, container-based, and Kubernetes Helm deployments.
- **[Usage Guide]**: Information on running the server, configuring LLM clients, and troubleshooting.
- **[Cheatsheet]**: A reference for what prompts to use to invoke various tools.

## Kubernetes Deployment (Helm)

This repository includes a Helm chart at `charts/linux-mcp-server` for Kubernetes deployments.

```bash
kubectl create namespace linux-mcp-server
kubectl -n linux-mcp-server create secret generic linux-mcp-server-ssh \
  --from-file=id_ed25519="$HOME/.ssh/id_ed25519" \
  --from-file=config="$HOME/.ssh/config"
helm install linux-mcp-server ./charts/linux-mcp-server -n linux-mcp-server
```

Optional Ingress:

```bash
helm upgrade --install linux-mcp-server ./charts/linux-mcp-server -n linux-mcp-server \
  --set ingress.enabled=true \
  --set ingress.className=nginx \
  --set ingress.hosts[0].host=linux-mcp.example.com \
  --set ingress.hosts[0].paths[0].path=/mcp \
  --set ingress.hosts[0].paths[0].pathType=Prefix
```

Optional Gateway API (HTTPRoute):

```bash
helm upgrade --install linux-mcp-server ./charts/linux-mcp-server -n linux-mcp-server \
  --set gatewayApi.enabled=true \
  --set gatewayApi.parentRefs[0].name=shared-gateway \
  --set gatewayApi.parentRefs[0].namespace=infra \
  --set gatewayApi.hostnames[0]=linux-mcp.example.com
```

If `gatewayApi.hostnames` is empty, the chart omits `spec.hostnames` from HTTPRoute (no hostname filtering).

Default chart behavior:

- MCP transport: `streamable-http`
- Service type: `ClusterIP` on port `8000`
- Supported `linuxMcp.transport` values for Kubernetes: `http`, `streamable-http`
- Supported `service.type` values: `ClusterIP`, `NodePort`, `LoadBalancer`
- `linuxMcp.port` controls container/app listen port; `service.port` controls Service exposure port
- Environment variables are managed via a dedicated Kubernetes ConfigMap
- `linuxMcp.extraEnv` adds key/value entries to the dedicated ConfigMap
- `extraEnv` and `extraEnvFrom` add direct container environment configuration
- `extraVolumes` and `extraVolumeMounts` add pass-through Kubernetes volumes and mounts
- PodDisruptionBudget is always created; default `minAvailable` is `0` when `replicaCount<=1`, otherwise `1`
- Ingress is optional and disabled by default
- Gateway API (HTTPRoute) is optional and disabled by default
- `ingress.enabled` and `gatewayApi.enabled` are mutually exclusive
- `ingress.enabled=true` requires at least one `ingress.hosts` entry
- `gatewayApi.enabled=true` requires at least one `gatewayApi.parentRefs` and `gatewayApi.rules` entry
- SSH key/config mount from existing secret `linux-mcp-server-ssh`
- Logs to stdout and `/tmp/linux-mcp-server/logs` inside the container


[Installation Guide]: https://rhel-lightspeed.github.io/linux-mcp-server/install/
[Usage Guide]: https://rhel-lightspeed.github.io/linux-mcp-server/usage/
[Cheatsheet]: https://rhel-lightspeed.github.io/linux-mcp-server/cheatsheet/
