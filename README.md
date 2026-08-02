# CogOS Charts

> **Experimental**: Minimal Helm templates for Kubernetes deployment. Not production-tested. These charts define the deployment structure but have not been validated in a real cluster.
>
> No container images are published for these components yet. See "Container Images" below before running `helm install` or `docker compose up`.

Helm charts and deployment manifests for deploying CogOS nodes to Kubernetes.

## Container Images

No `ghcr.io/myrgic/*` images are published yet. The registry paths in these charts and in `docker-compose.yml` are placeholders, not a working pull target, and `helm install` or `docker compose up` against the defaults below will fail with an image-pull error.

Until images are published, there are two ways to run these charts:

- **Build from source and point the chart at your own image.** `myrgic/cogos` and `myrgic/constellation` ship Dockerfiles; `myrgic/mod3` does not yet.

  ```sh
  docker build -t your-registry/cogos:dev ../cogos
  docker push your-registry/cogos:dev
  helm install my-node charts/cogos-node \
    --set cogos-kernel.image.repository=your-registry/cogos \
    --set cogos-kernel.image.tag=dev
  ```

- **Use Docker Compose for local development**, which builds the kernel image from source directly (see below).

Treat `image.repository` and `image.tag` in each chart's `values.yaml` as fields you fill in, not defaults that already resolve.

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

The node chart pins specific versions of each component. Upgrading the chart upgrades the whole node, like a Kubernetes release bundling component versions.

### Component charts

Each component has its own chart for standalone deployment or custom compositions:

```sh
# Kernel only
helm install kernel charts/cogos-kernel --set workspace.path=/data/workspace

# Mod³ only (connects to existing kernel)
helm install mod3 charts/cogos-mod3 --set kernel.endpoint=http://cogos-kernel:6931
```

## Docker Compose (local development)

For local development without Kubernetes. The `cogos` service builds from a sibling `../cogos` checkout, so no image is pulled:

```sh
docker compose build     # Build the kernel image from ../cogos
docker compose up        # Start the full node
docker compose up cogos  # Kernel only
```

Mod³ has no Dockerfile yet (see the commented-out block in `docker-compose.yml`), so it isn't part of this containerized path yet. Run it on the host per the macOS note below.

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

myrgic/cogos              ← kernel source, has a Dockerfile
myrgic/mod3               ← voice server source, no Dockerfile yet
myrgic/constellation      ← identity/trust source, has a Dockerfile
```

This repo composes those components into deployable units. None of them currently publish a container image (see "Container Images" above).

## Local macOS Note

On macOS, the kernel can run in a container but Mod³ typically runs on bare metal (Docker containers can't access host audio devices). The compose file handles this:

```yaml
services:
  cogos:
    image: ghcr.io/myrgic/cogos:latest    # built from ../cogos, not pulled
    ports: ["6931:6931"]

  # mod3 runs on host, connects to kernel via network
  # Start separately: mod3 serve --kernel http://localhost:6931
```

Port 6931 is the kernel default. Override via `kernel.yaml` or `--port` flag.

For headless servers or Linux, the kernel can run fully containerized once you have an image (see "Container Images" above). Mod³ still needs to run on the host until it has a Dockerfile.

## License

MIT
