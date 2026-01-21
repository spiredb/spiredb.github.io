# Storage Engine

Durable, persistent storage with LSM-tree architecture.

## Data Layout

```
/var/lib/spiredb/
├── data/           # Persistent data files
├── raft/           # Region Raft logs
│   └── region_*/
└── pd/             # PD metadata
```

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `SPIRE_DATA_PATH` | `/var/lib/spiredb/data` | Data directory |

## Operations

### Point Lookup

1. Check in-memory buffer
2. Check block cache
3. Check bloom filter
4. Read from disk

Latency: 0.05-0.2ms (cached)

### Range Scan

```elixir
Engine.scan_range(engine, "user:", "user:~", batch_size: 1000)
```

Returns batched iterator for efficient large scans.

### Write Path

1. Append to WAL (durability)
2. Insert to memory buffer
3. Return to client
4. Background flush to disk

## Monitoring

Get storage statistics:

```elixir
Store.KV.Engine.get_stats(Store.KV.Engine)
Store.KV.Engine.get_cache_info(Store.KV.Engine)
```
