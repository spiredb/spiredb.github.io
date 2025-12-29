# Monitoring

Observability for SpireDB clusters.

## Metrics

### Prometheus Endpoint

Default port: 9090

```bash
curl http://localhost:9090/metrics
```

### Key Metrics

#### Raft

| Metric | Description |
|--------|-------------|
| `raft_proposals_total` | Proposals submitted |
| `raft_apply_duration_seconds` | Apply latency |
| `raft_log_size_bytes` | Raft log size |

#### RocksDB

| Metric | Description |
|--------|-------------|
| `rocksdb_write_duration_seconds` | Write latency |
| `rocksdb_read_duration_seconds` | Read latency |
| `rocksdb_compaction_pending` | Pending compactions |
| `rocksdb_block_cache_hits` | Cache hit count |
| `rocksdb_block_cache_misses` | Cache miss count |

#### RESP

| Metric | Description |
|--------|-------------|
| `resp_commands_total` | Commands processed |
| `resp_command_duration_seconds` | Command latency |
| `resp_connections_active` | Active connections |

## Logging

### Structured JSON

Production uses JSON logging:

```json
{
  "time": "2024-12-28T10:00:00Z",
  "level": "info",
  "msg": "Region initialized",
  "region_id": 1,
  "node": "spiredb@node1"
}
```

### Log Levels

```bash
SPIRE_LOG_LEVEL=debug  # debug, info, warning, error
```

### Key Log Events

| Event | Level | Meaning |
|-------|-------|---------|
| `Region initialized` | info | Region Raft started |
| `Leader elected` | info | Raft leader change |
| `Compaction started` | debug | RocksDB compaction |
| `Slow query` | warning | Command > 100ms |
| `Connection closed` | debug | Client disconnect |

## Health Checks

### RESP Ping

```bash
redis-cli -p 6379 ping
# PONG
```

### HTTP Health

```bash
curl http://localhost:9090/health
# {"status":"ok"}
```

## Grafana Dashboards

### Overview Dashboard

Panels:
- QPS (commands/second)
- Latency P50/P99
- Active connections
- Raft leader distribution
- RocksDB cache hit rate

### Raft Dashboard

Panels:
- Proposals per region
- Apply latency histogram
- Leader elections timeline
- Log size per region

### Storage Dashboard

Panels:
- Disk usage
- Compaction rate
- Write amplification
- Block cache efficiency

## Alerting

### Critical Alerts

```yaml
# No leader for region
- alert: SpireDBNoLeader
  expr: raft_leader_status == 0
  for: 30s
  labels:
    severity: critical

# Disk > 80%
- alert: SpireDBDiskHigh
  expr: disk_usage_percent > 80
  for: 5m
  labels:
    severity: warning

# Latency spike
- alert: SpireDBHighLatency
  expr: histogram_quantile(0.99, resp_command_duration_seconds) > 0.1
  for: 5m
  labels:
    severity: warning
```

## Elixir Observer

Live system inspection:

```elixir
# Connect to running node
iex --remsh spiredb@hostname

# Process info
:observer.start()

# Memory
:erlang.memory()

# Process count
length(Process.list())
```

## RocksDB Statistics

```elixir
Store.KV.Engine.get_stats(Store.KV.Engine)
# [
#   {:block_cache_hit, 12345},
#   {:block_cache_miss, 123},
#   {:compaction_time, 5000},
#   ...
# ]
```
