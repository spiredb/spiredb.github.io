# Deployment

Below you can see the different ways to deploy SpireDB and also discovery methods.

## Release Build

Generate a self-contained release using Mix:

```bash
MIX_ENV=prod mix release
```

Start the node:

```bash
_build/prod/rel/spiredb_store/bin/spiredb_store start
```

## Docker

Use the official image:

```bash
docker run -d \
  --name spiredb \
  -p 6379:6379 \
  -v spiredb_data:/var/lib/spiredb/data \
  ghcr.io/spiredb/spiredb:latest
```

## Kubernetes (Helm)

For production Kubernetes deployments, use our official Helm chart.

- [**Helm Deployment Guide**](kubernetes.md) - Full instructions, configuration values, and architecture.
- [**Helm Charts Repository**](https://github.com/spiredb/charts) - Source code for charts.

Quick Command:
```bash
helm repo add spiredb https://charts.spire.zone
helm install spire spiredb/spire \
  --create-namespace \
  --namespace spire \
  --set spiresql.replicaCount=3 \
  --set spiresql.autoscaling.maxReplicas=20
```

## Cluster Discovery

SpireDB determines how to find peer nodes and form a cluster based on the `SPIRE_DISCOVERY` environment variable.

### 1. Kubernetes DNS (Recommended)
Automatically discovers pods backing a Kubernetes Headless Service. This is the default strategy for K8s deployments.

- **Env Var**: `SPIRE_DISCOVERY=k8sdns`
- **Configuration**:
  - `SPIRE_SERVICE_NAME` - Name of the headless service (Default: `spiredb-headless`)
  - `SPIRE_NAMESPACE` - Namespace of the pods (Default: `default`)

### 2. Gossip (Multicast UDP)
Uses UDP multicast to broadcast presence and discover other nodes on the same subnet. Useful for local Docker networks or bare-metal LANs where multicast is enabled.

- **Env Var**: `SPIRE_DISCOVERY=gossip`
- **Configuration**:
  - `SPIRE_MULTICAST_ADDR` - Multicast address (Default: `230.1.1.251`)
  - **Port**: Fixed at `45892`

### 3. DNS Polling (AWS/Cloud)
Polls a specific DNS A-record to retrieve a list of peer IP addresses. Useful for AWS ECS service discovery or other non-k8s cloud environments.

- **Env Var**: `SPIRE_DISCOVERY=dnspoll`
- **Configuration**:
  - `SPIRE_DNS_QUERY` - The DNS name to query (Default: `spire.zone`)

### 4. Static (EPMD)
Relies on a static list of node names. Suitable for local development or static IPs.

- **Env Var**: `SPIRE_DISCOVERY=epmd`
- **Configuration**:
  - `SPIRE_CLUSTER_NODES` - Comma-separated list of node names (e.g., `spiredb@10.0.1.2,spiredb@10.0.1.3`)
