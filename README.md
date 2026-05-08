# CogOS Charts

> **Experimental** — Minimal Helm templates for Kubernetes deployment. Not production-tested. These charts define the deployment structure but have not been validated in a real cluster.

Helm charts and deployment manifests for deploying CogOS nodes to Kubernetes.

## Charts

| Chart | What it deploys | Status |
|-------|----------------|--------|
| **cogos-node** | Complete node (kernel + optional services) | Primary -- start here |
| **cogos-kernel** | Kernel only (the daemon) | Standalone component |
| **cogos-mod3** | Mod³ voice server (TTS, VAD) | Standalone component |

### cogos-node (the umbrella chart)

Deploys a complete CogOS node as a single unit. Includes the kernel by default and optionally enables additional services:

```sh
# Default: kernel only
helm install my-node charts/cogos-node

# With Mod³ modality server
helm install my-node charts/cogos-node --set mod3.enabled=true

# With custom workspace path
helm install my-node charts/cogos-node --set workspace.path=/data/my-workspace
```

The node chart pins specific versions of each component. Upgrading the chart upgrades the whole node — like a Kubernetes release bundling component versions.

### Component charts

Each component has its own chart for standalone deployment or custom compositions:

```sh
# Kernel only
helm install kernel charts/cogos-kernel --set workspace.path=/data/workspace

# Mod³ only (connects to existing kernel)
helm install mod3 charts/cogos-mod3 --set kernel.endpoint=http://cogos-kernel:6931
```

## Docker Compose (local development)

For local development without Kubernetes:

```sh
docker compose up        # Start the full node
docker compose up cogos  # Kernel only
```

## Version Pinning

The node chart defines the tested combination of component versions:

```yaml
# charts/cogos-node/Chart.yaml
dependencies:
  - name: cogos-kernel
    version: "0.1.x"
  - name: cogos-mod3
    version: "0.2.x"
    condition: mod3.enabled
```

Upgrade the node chart → upgrade all components together, tested as a unit.

## Architecture

```
myrgic/charts             ← this repo (orchestration layer)
  charts/cogos-node       ← umbrella chart
  charts/cogos-kernel     ← the daemon
  charts/cogos-mod3       ← voice server
  docker-compose.yml      ← local dev alternative

myrgic/cogos              ← kernel source + Dockerfile
myrgic/mod3               ← voice server source + Dockerfile
myrgic/constellation      ← identity/trust source + Dockerfile
```

Each component repo builds and publishes its own container image. This repo composes them into deployable units.

## Local macOS Note

On macOS, the kernel can run in a container but Mod³ typically runs on bare metal (Docker containers can't access host audio devices). The compose file handles this:

```yaml
services:
  cogos:
    image: ghcr.io/myrgic/cogos:latest    # containerized
    ports: ["6931:6931"]

  # mod3 runs on host, connects to kernel via network
  # Start separately: mod3 serve --kernel http://localhost:6931
```

Port 6931 is the kernel default. Override via `kernel.yaml` or `--port` flag.

For headless servers or Linux, everything can be fully containerized.

## License

MIT
