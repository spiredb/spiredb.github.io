# gRPC API

SpireDB exposes gRPC services for high-performance programmatic access.

## Ports

| Port | Service |
|------|---------|
| **50051** | Placement Driver (PD) - Schema, Cluster, TSO, Plugin |
| **50052** | Store - DataAccess, Transaction, Vector |

---

## DataAccess Service (Port 50052)

High-performance data access with Arrow IPC format.

### Raw KV Operations

#### RawGet
```protobuf
rpc RawGet(RawGetRequest) returns (RawGetResponse);

message RawGetRequest {
  uint64 region_id = 1;
  bytes key = 2;
  uint64 snapshot_ts = 3;  // 0 = latest
  bool read_follower = 4;
}

message RawGetResponse {
  bytes value = 1;
  bool found = 2;
}
```

#### RawPut
```protobuf
rpc RawPut(RawPutRequest) returns (Empty);

message RawPutRequest {
  bytes key = 1;
  bytes value = 2;
}
```

#### RawDelete
```protobuf
rpc RawDelete(RawDeleteRequest) returns (Empty);

message RawDeleteRequest {
  bytes key = 1;
}
```

#### RawScan
Streaming scan returning Arrow RecordBatches.
```protobuf
rpc RawScan(RawScanRequest) returns (stream RawScanResponse);

message RawScanRequest {
  uint64 region_id = 1;
  bytes start_key = 2;
  bytes end_key = 3;
  uint32 batch_size = 4;
  uint32 limit = 5;
  uint64 snapshot_ts = 6;
  bool read_follower = 7;
}

message RawScanResponse {
  bytes arrow_batch = 1;  // Schema: [key: Binary, value: Binary]
  bool has_more = 2;
  ScanStats stats = 3;
}
```

#### RawBatchGet
```protobuf
rpc RawBatchGet(RawBatchGetRequest) returns (RawBatchGetResponse);

message RawBatchGetRequest {
  uint64 region_id = 1;
  repeated bytes keys = 2;
  uint64 snapshot_ts = 3;
}

message RawBatchGetResponse {
  bytes arrow_batch = 1;  // Schema: [key: Binary, value: Binary, found: Bool]
}
```

### Table Operations

#### TableScan
```protobuf
rpc TableScan(TableScanRequest) returns (stream TableScanResponse);

message TableScanRequest {
  string table_name = 1;
  repeated string columns = 2;    // Projection (empty = all)
  bytes filter_expr = 3;          // Serialized DataFusion Expr
  uint32 limit = 4;
  uint64 snapshot_ts = 5;
  bool read_follower = 6;
}

message TableScanResponse {
  bytes arrow_batch = 1;
  bool has_more = 2;
  ScanStats stats = 3;
}
```

#### TableGet / TableInsert / TableUpdate / TableDelete
```protobuf
rpc TableGet(TableGetRequest) returns (TableGetResponse);
rpc TableInsert(TableInsertRequest) returns (TableInsertResponse);
rpc TableUpdate(TableUpdateRequest) returns (TableUpdateResponse);
rpc TableDelete(TableDeleteRequest) returns (TableDeleteResponse);
```

---

## TransactionService (Port 50052)

Percolator-style distributed transactions.

### Prewrite
```protobuf
rpc Prewrite(PrewriteRequest) returns (PrewriteResponse);

message PrewriteRequest {
  repeated Mutation mutations = 1;
  bytes primary_key = 2;
  uint64 start_ts = 3;
  uint64 lock_ttl = 4;
  bool is_pessimistic = 5;
}

message PrewriteResponse {
  bool success = 1;
  repeated KeyError errors = 2;
}
```

### Commit
```protobuf
rpc Commit(CommitRequest) returns (CommitResponse);

message CommitRequest {
  bytes primary_key = 1;
  uint64 start_ts = 2;
  uint64 commit_ts = 3;
  repeated bytes keys = 4;
  repeated bytes read_keys = 10;   // For SSI
  repeated bytes write_keys = 11;
}

message CommitResponse {
  bool success = 1;
  uint64 commit_ts = 2;
  string error = 3;
  ConflictInfo conflict = 4;
}
```

### Rollback
```protobuf
rpc Rollback(RollbackRequest) returns (Empty);
```

### Lock Operations
```protobuf
rpc CheckTxnStatus(CheckTxnStatusRequest) returns (TxnStatus);
rpc ResolveLock(ResolveLockRequest) returns (Empty);
rpc AcquirePessimisticLock(PessimisticLockRequest) returns (PessimisticLockResponse);
```

---

## VectorService (Port 50052)

Vector similarity search.

### CreateIndex / DropIndex
```protobuf
rpc CreateIndex(VectorIndexCreateRequest) returns (Empty);
rpc DropIndex(VectorIndexDropRequest) returns (Empty);

message VectorIndexCreateRequest {
  string name = 1;
  string table_name = 2;
  string column_name = 3;
  string algorithm = 4;         // "ANODE" or "MANODE"
  map<string, string> params = 5;
}
```

### Insert / Delete
```protobuf
rpc Insert(VectorInsertRequest) returns (VectorInsertResponse);
rpc Delete(VectorDeleteRequest) returns (Empty);

message VectorInsertRequest {
  string index_name = 1;
  bytes doc_id = 2;
  bytes vector = 3;             // Float32 array as bytes
  bytes payload = 4;            // Optional JSON metadata
}
```

### Search / BatchSearch
```protobuf
rpc Search(VectorSearchRequest) returns (VectorSearchResponse);
rpc BatchSearch(BatchVectorSearchRequest) returns (BatchVectorSearchResponse);

message VectorSearchRequest {
  string index_name = 1;
  bytes query_vector = 2;
  uint32 k = 3;
  float radius = 4;             // Range search (0 = disabled)
  bytes filter = 5;             // Metadata filter
  bool return_payload = 6;
}

message VectorSearchResponse {
  repeated VectorResult results = 1;
}

message VectorResult {
  bytes id = 1;
  float distance = 2;
  bytes payload = 3;
}
```

---

## TSOService (Port 50051)

Timestamp Oracle for MVCC.

```protobuf
rpc GetTimestamp(GetTimestampRequest) returns (GetTimestampResponse);

message GetTimestampRequest {
  uint32 count = 1;  // Batch size
}

message GetTimestampResponse {
  uint64 start_ts = 1;
  uint32 count = 2;
}
```

---

## SchemaService (Port 50051)

Table and index management.

```protobuf
rpc CreateTable(CreateTableRequest) returns (CreateTableResponse);
rpc DropTable(DropTableRequest) returns (Empty);
rpc GetTable(GetTableRequest) returns (TableSchema);
rpc ListTables(Empty) returns (TableList);

rpc CreateIndex(CreateIndexRequest) returns (CreateIndexResponse);
rpc DropIndex(DropIndexRequest) returns (Empty);
rpc GetIndex(GetIndexRequest) returns (IndexSchema);
rpc ListIndexes(ListIndexesRequest) returns (IndexList);

rpc GetTableStats(GetTableStatsRequest) returns (TableStats);
rpc UpdateTableStats(UpdateTableStatsRequest) returns (Empty);
```

---

## ClusterService (Port 50051)

Region routing and store management.

```protobuf
rpc GetRegion(GetRegionRequest) returns (Region);
rpc GetRegionByKey(GetRegionByKeyRequest) returns (Region);
rpc GetTableRegions(GetTableRegionsRequest) returns (RegionList);

rpc GetStore(GetStoreRequest) returns (Store);
rpc ListStores(Empty) returns (StoreList);
rpc RegisterStore(RegisterStoreRequest) returns (RegisterStoreResponse);
rpc Heartbeat(StoreHeartbeat) returns (StoreHeartbeatResponse);
```

---

## PluginService (Port 50051)

Plugin management.

```protobuf
rpc InstallPlugin(InstallPluginRequest) returns (InstallPluginResponse);
rpc UninstallPlugin(UninstallPluginRequest) returns (Empty);
rpc ListPlugins(Empty) returns (PluginList);
rpc ReloadPlugin(ReloadPluginRequest) returns (Empty);

message InstallPluginRequest {
  oneof source {
    string hex_package = 1;   // e.g., "geo_index"
    string github_repo = 2;   // e.g., "user/spiredb-fulltext"
    bytes tarball = 3;        // Raw plugin archive
  }
}
```

---

## Client Examples

### Rust
```rust
use spiredb::data::{DataAccessClient, RawScanRequest};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut client = DataAccessClient::connect("http://localhost:50052").await?;
    
    let request = RawScanRequest {
        region_id: 1,
        start_key: b"user:".to_vec(),
        end_key: b"user:~".to_vec(),
        batch_size: 1000,
        ..Default::default()
    };
    
    let mut stream = client.raw_scan(request).await?.into_inner();
    while let Some(response) = stream.message().await? {
        let batch = arrow::ipc::reader::read_record_batch(&response.arrow_batch)?;
        println!("Got {} rows", batch.num_rows());
    }
    Ok(())
}
```

### Go
```go
import (
    pb "spiredb/data"
    "google.golang.org/grpc"
)

func main() {
    conn, _ := grpc.Dial("localhost:50052", grpc.WithInsecure())
    client := pb.NewDataAccessClient(conn)
    
    stream, _ := client.RawScan(context.Background(), &pb.RawScanRequest{
        RegionId:  1,
        StartKey:  []byte("user:"),
        EndKey:    []byte("user:~"),
        BatchSize: 1000,
    })
    
    for {
        resp, err := stream.Recv()
        if err == io.EOF {
            break
        }
        // Process resp.ArrowBatch
    }
}
```

### Python
```python
import grpc
from spiredb.data_pb2_grpc import DataAccessStub
from spiredb.data_pb2 import RawScanRequest
import pyarrow as pa

channel = grpc.insecure_channel('localhost:50052')
client = DataAccessStub(channel)

request = RawScanRequest(
    region_id=1,
    start_key=b'user:',
    end_key=b'user:~',
    batch_size=1000
)

for response in client.RawScan(request):
    reader = pa.ipc.open_stream(response.arrow_batch)
    batch = reader.read_all()
    print(f"Got {batch.num_rows} rows")
```
