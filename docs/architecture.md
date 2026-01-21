# Architecture

SpireDB is built on three core subsystems: the Placement Driver, Store Nodes, and the RESP protocol handler.

## System Overview

```mermaid
graph TB
    subgraph Cluster[SpireDB Cluster]
        direction TB
        
        subgraph PDLayer[Placement Driver Layer]
            direction TB
            PDS[PD.Server<br/>Metadata]
            subgraph PDComponents
                direction LR
                PDR[PD.Router<br/>Routing Table]
                PDSched[PD.Scheduler<br/>Rebalancing]
            end
            PDS --- PDComponents
        end

        subgraph StoreLayer[Store Node Layer]
            direction LR
            
            subgraph Node1[Store Node 1]
                direction TB
                subgraph N1Regions[Regions]
                    direction LR
                    N1R1[Region 1]
                    N1R2[Region ...]
                    N1R16[Region 16]
                end
                N1Store[(Storage)]
                N1Regions --> N1Store
            end

            subgraph NodeN[Store Node N]
                direction TB
                subgraph NNRegions[Regions]
                    direction LR
                    NNR1[Region 1]
                    NNR2[Region ...]
                    NNR16[Region 16]
                end
                NNStore[(Storage)]
                NNRegions --> NNStore
            end
        end

        PDLayer -->|Heartbeats & Instruction| StoreLayer
    end
```

## Components

### Placement Driver (PD)

Central metadata service managing:

| Component | Function |
|-----------|----------|
| PD.Server | Raft-backed state machine for metadata |
| PD.Router | Key-to-region routing |
| PD.Scheduler | Automatic region rebalancing |
| PD.API.GRPC | gRPC interface for region discovery |

PD maintains:
- Store registry (alive nodes)
- Region assignments
- Cluster topology

### Store Nodes

Data storage and serving nodes:

| Component | Function |
|-----------|----------|
| Store.Server | Request coordination |
| Store.Region.Raft | Per-region Raft consensus |
| Store.KV.Engine | Persistent storage engine |
| Store.API.RESP | Redis protocol handler |

### Region

A region is a contiguous range of the key space. Each region:
- Has its own Raft group
- Replicates across multiple stores
- Splits when too large

Default: 16 regions distributed across nodes.

## Data Flow

### Write Path

```mermaid
sequenceDiagram
    participant C as Client
    participant RESP as RESP Handler
    participant S as Store.Server
    participant R as Region.Raft
    participant Q as Quorum
    participant DB as Storage

    C->>RESP: SET key value
    RESP->>S: Parse Command
    S->>R: Route to Region
    R->>Q: Propose Write via Raft
    Q-->>R: Consensus Reached
    R->>DB: Apply (StateMachine)
    R-->>C: OK
```

Latency: 3-7ms (Raft consensus overhead)

### Read Path

```mermaid
sequenceDiagram
    participant C as Client
    participant RESP as RESP Handler
    participant S as Store.Server
    participant DB as Storage

    C->>RESP: GET key
    RESP->>S: Parse Command
    S->>DB: Route & Direct Read
    DB-->>C: Return Value
```

Latency: 0.3-0.5ms (bypasses Raft)

## Key Encoding

Keys are hashed to determine region assignment:

```
region_id = :erlang.phash2(key, num_regions) + 1
```

Region 1 handles keys where `hash(key) % 16 == 0`, etc.

## Storage Layout

```
/var/lib/spiredb/
├── data/              # Persistent data
│   ├── CURRENT
│   ├── MANIFEST-*
│   ├── *.sst
│   └── *.log
├── raft/              # Raft logs
│   └── region_*/
│       ├── wal/
│       └── snapshot/
└── pd/                # PD metadata
```

## Supervision Tree

```mermaid
graph TD
    subgraph StoreApp[SpiredbStore.App]
        StoreSup[Store.Supervisor]
        ClusterSup[Cluster.Supervisor]
        RESPSup[Store.API.RESP.Supervisor]
        
        StoreSup --> Engine[Store.KV.Engine]
        StoreSup --> Server[Store.Server]
        RESPSup --> Ranch[Ranch Listener]
    end

    subgraph PDApp[SpiredbPd.App]
        PDSup[PD.Supervisor]
        PDSup --> PDRaft[PD.Server Raft]
        PDSup --> PDSched[PD.Scheduler]
        GRPCSup[PD.API.GRPC.Supervisor]
    end
```

## Technology Stack

| Component | Technology |
|-----------|------------|
| Language | Elixir/OTP |
| Consensus | Ra (Raft) |
| Storage | LSM-Tree Engine |
| Protocol | RESP (Redis) |
| Clustering | libcluster |
| Observability | Telemetry, Prometheus |
