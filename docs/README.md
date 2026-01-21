# SpireDB

All-in-one Streaming OLTP distributed database where **Elixir** governs the cluster and **Rust** optimizes the compute.

## Architecture

SpireDB is built on a dual-engine architecture:

| Component | Technology | Purpose |
|-----------|------------|---------|
| **SpireDB** | Elixir/OTP | Cluster coordination, Raft consensus, fault tolerance |
| **SpireSQL** | Rust | JIT query execution, vector search, zero-copy performance |
| **SpireWire** | RESP Protocol | Redis-compatible client gateway |

## Features

- **BEAM Resilience** - Built with OTP for unmatched uptime
- **Raft Consensus** - Linearizable writes, strong consistency
- **Redis Compatible** - Drop-in replacement using existing clients
- **Vector Search** - Native ANN with ANODE/MANODE algorithms
- **Change Streams** - Real-time CDC for event sourcing
- **Geo-Distributed** - Multi-region with automatic failover
- **Transactions** - ACID with multiple isolation levels
- **Plugin System** - Extend with custom indexes, storage, functions

## Quick Example

```bash
# Start SpireDB
docker run -p 6379:6379 ghcr.io/spiredb/spiredb:latest

# Connect with redis-cli
redis-cli -p 6379

> SET user:1 "alice"
OK
> GET user:1
"alice"
> SPIRE.VECTOR.CREATE embeddings DIM 384
OK
```

## Next Steps

- [Quick Start Guide](quickstart.md) - Get running in 5 minutes
- [Architecture](architecture.md) - Understand the dual-engine design
- [Kubernetes Deployment](kubernetes.md) - Production setup
- [Command Reference](commands/index.md) - Full RESP command list
- [Spire CLI](cli.md) - Cluster management tool
- [gRPC API](grpc-api.md) - High-performance binary protocol
