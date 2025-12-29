# Quick Start

Get SpireDB running in under 5 minutes.

## Prerequisites

- Docker (recommended) or Elixir 1.14+
- redis-cli for testing

## Option 1: Docker (Recommended)

```bash
# Pull and run
docker run -d \
  --name spiredb \
  -p 6379:6379 \
  -v spiredb_data:/var/lib/spiredb \
  spiredb/spiredb:latest

# Verify
redis-cli -p 6379 PING
# PONG
```

## Option 2: From Source

```bash
# Clone
git clone https://github.com/spiredb/spiredb.git
cd spiredb

# Install dependencies
make setup

# Build
make build

# Run
make run
```

## Basic Usage

Connect with any Redis client:

```bash
redis-cli -p 6379
```

### String Operations

```redis
SET user:1:name "Alice"
GET user:1:name
# "Alice"

SET counter 0
INCR counter
# (integer) 1
INCRBY counter 10
# (integer) 11
```

### Multi-Key Operations

```redis
MSET key1 "val1" key2 "val2" key3 "val3"
MGET key1 key2 key3
# 1) "val1"
# 2) "val2"
# 3) "val3"
```

### Key Management

```redis
EXISTS key1
# (integer) 1

DEL key1 key2
# (integer) 2

EXISTS key1
# (integer) 0
```

## Configuration

Key environment variables:

```bash
# Network
SPIRE_RESP_PORT=6379

# Storage
SPIRE_ROCKSDB_PATH=/var/lib/spiredb/data

# Raft
SPIRE_RAFT_DATA_DIR=/var/lib/spiredb/raft

# Logging
SPIRE_LOG_LEVEL=info
```

See [Configuration](configuration.md) for full reference.

## Next Steps

- [Architecture Overview](architecture.md)
- [Redis API Reference](redis-api.md)
- [Docker Deployment](docker.md)
- [Kubernetes Deployment](kubernetes.md)
