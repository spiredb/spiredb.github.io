# Deployment

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

## Clustering

SpireDB uses Erlang distribution.

### DNS Discovery (Kubernetes)

Set `SPIRE_DISCOVERY=k8sdns` and configure service details:

```yaml
env:
  - name: SPIRE_DISCOVERY
    value: "k8sdns"
  - name: SPIRE_SERVICE_NAME
    value: "spiredb-headless"
  - name: SPIRE_NAMESPACE
    valueFrom:
      fieldRef:
        fieldPath: metadata.namespace
```

### Static List (Bare Metal)

Set `SPIRE_DISCOVERY=epmd` and list nodes:

```bash
export SPIRE_DISCOVERY=epmd
export SPIRE_CLUSTER_NODES=spiredb@10.0.0.1,spiredb@10.0.0.2
```
