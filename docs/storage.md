# Storage Engine

RocksDB-based persistent storage.

## Architecture

```
┌──────────────────────────────────┐
│         Store.KV.Engine          │
├──────────────────────────────────┤
│  Block Cache (512MB default)     │
├──────────────────────────────────┤
│  Write Buffer (MemTable)         │
├──────────────────────────────────┤
│         RocksDB LSM Tree         │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐    │
│  │ L0 │ │ L1 │ │ L2 │ │ L3 │    │
│  └────┘ └────┘ └────┘ └────┘    │
├──────────────────────────────────┤
│           SSD/NVMe               │
└──────────────────────────────────┘
```

## Data Layout

```
/var/lib/spiredb/
├── data/           # RocksDB files
│   ├── CURRENT
│   ├── MANIFEST-*
│   ├── *.sst       # Sorted String Tables
│   └── *.log       # WAL
├── raft/           # Region Raft logs
│   └── region_*/
└── pd/             # PD metadata
```

## Key Features

### Block Cache

LRU cache for read performance.

```bash
SPIRE_ROCKSDB_BLOCK_CACHE_SIZE=536870912  # 512MB
```

### Bloom Filters

10 bits per key, ~1% false positive rate.

```bash
SPIRE_ROCKSDB_BLOOM_BITS_PER_KEY=10
```

### Compression

| Level | Use Case |
|-------|----------|
| `none` | Low CPU, max speed |
| `lz4` | Balanced (recommended) |
| `zstd` | Max compression |

```bash
SPIRE_ROCKSDB_COMPRESSION=lz4
```

### Write Buffer

MemTable before flush to SST.

```bash
SPIRE_ROCKSDB_WRITE_BUFFER_SIZE=134217728  # 128MB
SPIRE_ROCKSDB_MAX_WRITE_BUFFER_NUMBER=4
```

## Operations

### Point Lookup

1. Check MemTable
2. Check Block Cache
3. Check Bloom Filter
4. Read from SST file

Latency: 0.05-0.2ms (cached)

### Range Scan

```elixir
Engine.scan_range(engine, "user:", "user:~", batch_size: 1000)
```

Returns batched iterator. Uses RocksDB iterator internally.

### Write Path

1. Append to WAL
2. Insert to MemTable
3. Return to client
4. Background flush to SST

## Compaction

### Leveled Compaction

Default strategy. Limits space amplification.

```
L0: 4 files trigger compaction
L1: 256MB
L2: 2.5GB
L3: 25GB
...
```

### Rate Limiting

Prevent I/O storms:

```bash
SPIRE_ROCKSDB_RATE_LIMIT_BYTES_PER_SEC=104857600  # 100MB/s
```

## Tuning Profiles

### Write-Heavy

```bash
SPIRE_ROCKSDB_WRITE_BUFFER_SIZE=268435456     # 256MB
SPIRE_ROCKSDB_MAX_WRITE_BUFFER_NUMBER=6
SPIRE_ROCKSDB_MAX_BACKGROUND_JOBS=8
```

### Read-Heavy

```bash
SPIRE_ROCKSDB_BLOCK_CACHE_SIZE=2147483648     # 2GB
SPIRE_ROCKSDB_COMPRESSION=lz4
```

### Mixed Workload

```bash
SPIRE_ROCKSDB_BLOCK_CACHE_SIZE=1073741824     # 1GB
SPIRE_ROCKSDB_WRITE_BUFFER_SIZE=134217728     # 128MB
SPIRE_ROCKSDB_COMPRESSION=lz4
```

## Monitoring

Get RocksDB statistics:

```elixir
Store.KV.Engine.get_stats(Store.KV.Engine)
Store.KV.Engine.get_cache_info(Store.KV.Engine)
```
