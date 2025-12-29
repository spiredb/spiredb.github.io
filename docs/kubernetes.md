# Kubernetes

Production deployment on Kubernetes.

## Prerequisites

- Kubernetes 1.28+
- Persistent volume provisioner
- `kubectl` configured

## Quick Deploy

```bash
kubectl create namespace spiredb
kubectl apply -f deploy/kubernetes/
```

## StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: spiredb
  namespace: spiredb
spec:
  serviceName: spiredb-headless
  replicas: 3
  selector:
    matchLabels:
      app: spiredb
  template:
    metadata:
      labels:
        app: spiredb
    spec:
      containers:
      - name: spiredb
        image: spiredb:latest
        ports:
        - containerPort: 6379
          name: redis
        - containerPort: 50051
          name: grpc
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: RELEASE_NODE
          value: "spiredb@$(POD_NAME).spiredb-headless.spiredb.svc.cluster.local"
        - name: SPIRE_DISCOVERY
          value: "k8sdns"
        - name: SPIRE_SERVICE_NAME
          value: "spiredb-headless"
        - name: SPIRE_NAMESPACE
          value: "spiredb"
        - name: SPIRE_ROCKSDB_COMPRESSION
          value: "lz4"
        volumeMounts:
        - name: data
          mountPath: /var/lib/spiredb
        resources:
          requests:
            cpu: 2
            memory: 4Gi
          limits:
            cpu: 4
            memory: 8Gi
        readinessProbe:
          exec:
            command: ["redis-cli", "-p", "6379", "ping"]
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          exec:
            command: ["redis-cli", "-p", "6379", "ping"]
          initialDelaySeconds: 30
          periodSeconds: 10
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 100Gi
```

## Services

### Headless (Cluster Discovery)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: spiredb-headless
  namespace: spiredb
spec:
  clusterIP: None
  ports:
  - port: 6379
    name: redis
  - port: 50051
    name: grpc
  selector:
    app: spiredb
```

### Client Access

```yaml
apiVersion: v1
kind: Service
metadata:
  name: spiredb
  namespace: spiredb
spec:
  type: ClusterIP
  ports:
  - port: 6379
    targetPort: 6379
    name: redis
  selector:
    app: spiredb
```

## ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: spiredb-config
  namespace: spiredb
data:
  SPIRE_LOG_LEVEL: "info"
  SPIRE_NUM_REGIONS: "16"
  SPIRE_ROCKSDB_COMPRESSION: "lz4"
  SPIRE_ROCKSDB_BLOCK_CACHE_SIZE: "1073741824"
```

## Pod Anti-Affinity

Spread across nodes:

```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: spiredb
        topologyKey: kubernetes.io/hostname
```

## Storage Class

For SSD storage:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

## Scaling

```bash
# Scale replicas
kubectl scale statefulset spiredb -n spiredb --replicas=5

# PD auto-discovers and rebalances regions
```

## Monitoring

### ServiceMonitor (Prometheus)

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: spiredb
  namespace: spiredb
spec:
  selector:
    matchLabels:
      app: spiredb
  endpoints:
  - port: metrics
    interval: 30s
```

## Operations

```bash
# Check pods
kubectl get pods -n spiredb

# Logs
kubectl logs -f spiredb-0 -n spiredb

# Shell access
kubectl exec -it spiredb-0 -n spiredb -- /bin/bash

# Redis CLI
kubectl exec -it spiredb-0 -n spiredb -- redis-cli -p 6379
```
