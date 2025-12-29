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
| `SPIRE_ROCKSDB_PATH` | `/var/lib/spiredb/data` | Data directory |
| `SPIRE_ROCKSDB_MAX_OPEN_FILES` | `10000` | Max open file handles |
| `SPIRE_ROCKSDB_BLOCK_CACHE_SIZE` | `536870912` | Block cache (512MB) |
| `SPIRE_ROCKSDB_WRITE_BUFFER_SIZE` | `134217728` | Write buffer (128MB) |
| `SPIRE_ROCKSDB_COMPRESSION` | `none` | Compression: `none`, `lz4`, `zstd` |

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
export SPIRE_ROCKSDB_PATH=/tmp/spiredb/data
export SPIRE_RAFT_DATA_DIR=/tmp/spiredb/raft
export SPIRE_LOG_LEVEL=debug
export SPIRE_NUM_REGIONS=4
```

### Production (Single Node)

```bash
export SPIRE_RESP_PORT=6379
export SPIRE_RESP_MAX_CONNECTIONS=50000
export SPIRE_ROCKSDB_PATH=/mnt/data/spiredb
export SPIRE_ROCKSDB_COMPRESSION=lz4
export SPIRE_ROCKSDB_BLOCK_CACHE_SIZE=1073741824  # 1GB
export SPIRE_LOG_LEVEL=info
```

### Production (Kubernetes)

```bash
export SPIRE_DISCOVERY=k8sdns
export SPIRE_SERVICE_NAME=spiredb-headless
export SPIRE_NAMESPACE=spiredb
export SPIRE_RESP_MAX_CONNECTIONS=50000
export SPIRE_ROCKSDB_COMPRESSION=lz4
```

## RocksDB Tuning

### Write-Heavy Workload

```bash
export SPIRE_ROCKSDB_WRITE_BUFFER_SIZE=268435456     # 256MB
export SPIRE_ROCKSDB_MAX_WRITE_BUFFER_NUMBER=6
export SPIRE_ROCKSDB_MAX_BACKGROUND_JOBS=8
```

### Read-Heavy Workload

```bash
export SPIRE_ROCKSDB_BLOCK_CACHE_SIZE=2147483648    # 2GB
export SPIRE_ROCKSDB_BLOOM_BITS_PER_KEY=10
export SPIRE_ROCKSDB_COMPRESSION=lz4
```

### SSD Storage

```bash
export SPIRE_ROCKSDB_COMPRESSION=lz4
export SPIRE_ROCKSDB_TARGET_FILE_SIZE_BASE=134217728  # 128MB
export SPIRE_ROCKSDB_RATE_LIMIT_BYTES_PER_SEC=209715200  # 200MB/s
```

## Elixir Configuration

For source builds, edit `config/prod.exs`:

```elixir
config :spiredb_store,
  resp_port: 6379,
  rocksdb_path: "/var/lib/spiredb/data",
  rocksdb_compression: :lz4,
  rocksdb_block_cache_size: 1_073_741_824

config :spiredb_pd,
  num_regions: 16,
  heartbeat_interval: 10_000
```
