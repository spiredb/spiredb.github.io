# Kubernetes

Production deployment using the SpireDB Helm chart.

## Prerequisites

- Kubernetes 1.28+
- Helm 3.x
- Persistent volume provisioner
- `kubectl` configured

## Helm Installation

```bash
# Add repository
helm repo add spiredb https://spiredb.github.io/helm-charts
helm repo update

# Install with defaults (3 replicas, gossip discovery)
helm install spiredb spiredb/spiredb -n spiredb --create-namespace

# Install with custom values
helm install spiredb spiredb/spiredb -n spiredb --create-namespace -f values.yaml
```

## Configuration

### values.yaml Reference

```yaml
replicaCount: 3

image:
  repository: ghcr.io/spiredb/spiredb
  tag: "latest"
  pullPolicy: IfNotPresent

# Cluster discovery method: gossip, k8sdns, dnspoll, epmd
discovery: gossip

# Persistent storage
persistence:
  enabled: true
  storageClass: ""      # Use default if empty
  size: 100Gi
  accessModes:
    - ReadWriteOnce

# Resource allocation
resources:
  requests:
    cpu: 2
    memory: 4Gi
  limits:
    cpu: 4
    memory: 8Gi

# SpireDB configuration
spiredb:
  pd:
    numRegions: 16
    heartbeatInterval: 10000

  ra:
    dataDir: /var/lib/spiredb/raft

  store:
    respPort: 6379
    respMaxConnections: 10000
    respConnectionTimeout: 60000

  rocksdb:
    path: /var/lib/spiredb/data
    compression: lz4
    blockCacheSize: 1073741824
    writeBufferSize: 134217728
    maxOpenFiles: 10000
    bloomBitsPerKey: 10

  raft:
    dataDir: /var/lib/spiredb/regions
    electionTimeout: 1000
    heartbeatInterval: 150
    walMaxSize: 67108864
    snapshotInterval: 4096

  logLevel: info
```

## Architecture

The Helm chart deploys:

| Component | Description |
|-----------|-------------|
| StatefulSet | 3+ SpireDB pods with stable network identity |
| Headless Service | Internal cluster discovery (EPMD, gossip) |
| ClusterIP Service | Client access (RESP, gRPC) |
| ServiceAccount + RBAC | Pod/endpoint access for discovery |
| PodDisruptionBudget | Maintains quorum during upgrades |
| NetworkPolicy | Optional network isolation |

### Services

**Client Service** (`spiredb`):
- Port 6379: RESP/Redis protocol
- Port 50052: DataAccess gRPC

**Headless Service** (`spiredb-headless`):
- Port 50051: PD Metadata gRPC
- Port 4369: EPMD
- Port 9001: Erlang distribution

## Discovery Methods

### Gossip (Default)

Uses UDP multicast for automatic peer discovery. Works in most environments.

```yaml
discovery: gossip
```

### Kubernetes DNS

Queries headless service DNS for peer discovery. Requires DNS addon.

```yaml
discovery: k8sdns
```

### DNS Polling

Polls a DNS record periodically. For external DNS-based discovery.

```yaml
discovery: dnspoll
spiredb:
  dnsQuery: "spiredb.example.com"
```

## Operations

### Upgrade

```bash
helm upgrade spiredb spiredb/spiredb -n spiredb -f values.yaml
```

### Scale

```bash
# Via Helm
helm upgrade spiredb spiredb/spiredb -n spiredb --set replicaCount=5

# Via kubectl (not recommended - use Helm)
kubectl scale statefulset spiredb -n spiredb --replicas=5
```

### Uninstall

```bash
helm uninstall spiredb -n spiredb

# PVCs are retained by default - delete manually if needed
kubectl delete pvc -l app.kubernetes.io/name=spiredb -n spiredb
```

## Access

### In-Cluster

```bash
# Redis CLI from any pod
redis-cli -h spiredb.spiredb.svc.cluster.local -p 6379

# Port forward for local access
kubectl port-forward svc/spiredb 6379:6379 -n spiredb
```

### Connection String

```
redis://spiredb.spiredb.svc.cluster.local:6379
```

## Monitoring

### Logs

```bash
# All pods
kubectl logs -l app.kubernetes.io/name=spiredb -n spiredb

# Specific pod
kubectl logs spiredb-0 -n spiredb -f

# Previous container (after crash)
kubectl logs spiredb-0 -n spiredb --previous
```

### Shell Access

```bash
kubectl exec -it spiredb-0 -n spiredb -- /bin/bash
```

### Health Check

```bash
# Check pod status
kubectl get pods -n spiredb

# Check RESP connectivity
kubectl exec spiredb-0 -n spiredb -- redis-cli ping
```

## High Availability

### PodDisruptionBudget

The chart includes a PDB maintaining minimum 2 available pods during voluntary disruptions:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: spiredb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: spiredb
```

### Pod Anti-Affinity

Enable to spread pods across nodes:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app.kubernetes.io/name: spiredb
        topologyKey: kubernetes.io/hostname
```

## Network Policy

Optional network isolation (disabled by default):

```yaml
networkPolicy:
  enabled: true
```

When enabled, allows:
- Inbound: RESP (6379), gRPC (50051, 50052), EPMD (4369), Dist (9001), Gossip UDP (45892)
- Outbound: DNS (UDP 53)
- Inter-pod: All traffic between SpireDB pods

## Storage

### Storage Class

For production, use SSD-backed storage:

```yaml
# GKE
persistence:
  storageClass: premium-rwo

# AWS EKS  
persistence:
  storageClass: gp3

# Azure AKS
persistence:
  storageClass: managed-premium
```

### Data Retention

PVCs are retained after uninstall. To delete:

```bash
kubectl delete pvc -l app.kubernetes.io/name=spiredb -n spiredb
```
