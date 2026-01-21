# Configuration

All configuration via environment variables with `SPIRE_` prefix.

## Environment Variables

### Network

| Variable | Default | Description |
|----------|---------|-------------|
| `SPIRE_RESP_PORT` | `6379` | Redis protocol port |
| `SPIRE_RESP_MAX_CONNECTIONS` | `10000` | Max concurrent connections |
| `SPIRE_RESP_CONNECTION_TIMEOUT` | `60000` | Connection timeout (ms) |

### Storage

| Variable | Default | Description |
|----------|---------|-------------|
| `SPIRE_DATA_PATH` | `/var/lib/spiredb/data` | Data directory |

### Raft

| Variable | Default | Description |
|----------|---------|-------------|
| `SPIRE_RAFT_DATA_DIR` | `/var/lib/spiredb/raft` | Raft log directory |
| `SPIRE_RAFT_ELECTION_TIMEOUT` | `1000` | Election timeout (ms) |
| `SPIRE_RAFT_HEARTBEAT_INTERVAL` | `150` | Heartbeat interval (ms) |
| `SPIRE_RAFT_WAL_MAX_SIZE` | `67108864` | WAL max size (64MB) |

### Placement Driver

| Variable | Default | Description |
|----------|---------|-------------|
| `SPIRE_NUM_REGIONS` | `16` | Number of regions |
| `SPIRE_PD_DATA_DIR` | `/var/lib/spiredb/pd` | PD data directory |
| `SPIRE_PD_START_RAFT` | `true` | Enable PD Raft |

### Clustering

| Variable | Default | Description |
|----------|---------|-------------|
| `SPIRE_DISCOVERY` | `epmd` | Discovery: `epmd`, `k8sdns`, `gossip` |
| `SPIRE_CLUSTER_NODES` | `` | Comma-separated node list for epmd |
| `SPIRE_SERVICE_NAME` | `spiredb` | K8s service name |
| `SPIRE_NAMESPACE` | `default` | K8s namespace |

### Logging

| Variable | Default | Description |
|----------|---------|-------------|
| `SPIRE_LOG_LEVEL` | `info` | Log level: `debug`, `info`, `warning`, `error` |

## Configuration Examples

### Development

```bash
export SPIRE_RESP_PORT=6379
export SPIRE_DATA_PATH=/tmp/spiredb/data
export SPIRE_RAFT_DATA_DIR=/tmp/spiredb/raft
export SPIRE_LOG_LEVEL=debug
export SPIRE_NUM_REGIONS=4
```

### Production (Single Node)

```bash
export SPIRE_RESP_PORT=6379
export SPIRE_RESP_MAX_CONNECTIONS=50000
export SPIRE_DATA_PATH=/mnt/data/spiredb
export SPIRE_LOG_LEVEL=info
```

### Production (Kubernetes)

```bash
export SPIRE_DISCOVERY=k8sdns
export SPIRE_SERVICE_NAME=spiredb-headless
export SPIRE_NAMESPACE=spiredb
export SPIRE_RESP_MAX_CONNECTIONS=50000
```



## Elixir Configuration

For source builds, edit `config/prod.exs`:

```elixir
config :spiredb_store,
  resp_port: 6379,
  data_path: "/var/lib/spiredb/data"

config :spiredb_pd,
  num_regions: 16,
  heartbeat_interval: 10_000
```
