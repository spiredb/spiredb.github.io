# Docker

Container deployment for SpireDB.

## Quick Start

```bash
docker run -d \
  --name spiredb \
  -p 6379:6379 \
  -v spiredb-data:/var/lib/spiredb \
  spiredb:latest
```

## Docker Compose

```yaml
version: '3.8'

services:
  spiredb:
    image: spiredb:latest
    ports:
      - "6379:6379"
      - "50051:50051"
    volumes:
      - spiredb-data:/var/lib/spiredb
    environment:
      - SPIRE_RESP_PORT=6379
      - SPIRE_LOG_LEVEL=info
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "-p", "6379", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  spiredb-data:
```

## Build Image

```bash
cd spiredb
docker build -t spiredb:latest -f docker/Dockerfile .
```

## Environment Variables

Pass via `-e` or `environment:` in compose:

```bash
docker run -d \
  -e SPIRE_RESP_PORT=6379 \
  -e SPIRE_LOG_LEVEL=info \
  spiredb:latest
```

## Volumes

| Path | Purpose |
|------|---------|
| `/var/lib/spiredb/data` | RocksDB data |
| `/var/lib/spiredb/raft` | Raft logs |
| `/var/lib/spiredb/pd` | PD metadata |

Mount all:

```bash
-v /mnt/ssd/spiredb:/var/lib/spiredb
```

## Ports

| Port | Protocol |
|------|----------|
| 6379 | RESP (Redis) |
| 50051 | gRPC (PD) |
| 50052 | gRPC (Store) |

## Resource Limits

```yaml
deploy:
  resources:
    limits:
      cpus: '4'
      memory: 8G
    reservations:
      cpus: '2'
      memory: 4G
```

## Multi-Node Cluster

```yaml
version: '3.8'

services:
  spiredb-1:
    image: spiredb:latest
    hostname: spiredb-1
    environment:
      - RELEASE_NODE=spiredb@spiredb-1
      - SPIRE_DISCOVERY=epmd
      - SPIRE_CLUSTER_NODES=spiredb@spiredb-2,spiredb@spiredb-3
    volumes:
      - node1-data:/var/lib/spiredb
    networks:
      - spiredb-net

  spiredb-2:
    image: spiredb:latest
    hostname: spiredb-2
    environment:
      - RELEASE_NODE=spiredb@spiredb-2
      - SPIRE_DISCOVERY=epmd
      - SPIRE_CLUSTER_NODES=spiredb@spiredb-1,spiredb@spiredb-3
    volumes:
      - node2-data:/var/lib/spiredb
    networks:
      - spiredb-net

  spiredb-3:
    image: spiredb:latest
    hostname: spiredb-3
    environment:
      - RELEASE_NODE=spiredb@spiredb-3
      - SPIRE_DISCOVERY=epmd
      - SPIRE_CLUSTER_NODES=spiredb@spiredb-1,spiredb@spiredb-2
    volumes:
      - node3-data:/var/lib/spiredb
    networks:
      - spiredb-net

networks:
  spiredb-net:
    driver: bridge

volumes:
  node1-data:
  node2-data:
  node3-data:
```

## Health Check

```bash
docker exec spiredb redis-cli -p 6379 ping
# PONG
```

## Logs

```bash
docker logs -f spiredb
```
