# gRPC API

Internal APIs for SpireSQL integration and cluster management.

## Services

### DataAccess Service

High-performance data access for SpireSQL.

```protobuf
service DataAccess {
  rpc Scan(ScanRequest) returns (stream ScanResponse);
  rpc Get(GetRequest) returns (GetResponse);
  rpc BatchGet(BatchGetRequest) returns (BatchGetResponse);
}
```

#### Scan

Range scan returning Arrow RecordBatches.

```protobuf
message ScanRequest {
  uint32 region_id = 1;
  bytes start_key = 2;
  bytes end_key = 3;
  uint32 batch_size = 4;  // default: 1000
  uint32 limit = 5;       // 0 = unlimited
  bool read_follower = 6;
}

message ScanResponse {
  bytes arrow_batch = 1;  // Arrow IPC format
  bool has_more = 2;
  ScanStats stats = 3;
}
```

#### Get

Point lookup.

```protobuf
message GetRequest {
  uint32 region_id = 1;
  bytes key = 2;
  bool read_follower = 3;
}

message GetResponse {
  bytes value = 1;
  bool found = 2;
}
```

#### BatchGet

Multi-key retrieval.

```protobuf
message BatchGetRequest {
  uint32 region_id = 1;
  repeated bytes keys = 2;
}

message BatchGetResponse {
  bytes arrow_batch = 1;  // [key, value, found]
}
```

### PlacementDriver Service

Region discovery for distributed scans.

```protobuf
service PlacementDriver {
  rpc GetTableRegions(GetTableRegionsRequest) returns (GetTableRegionsResponse);
  rpc GetRegion(GetRegionRequest) returns (Region);
  rpc RegisterStore(RegisterStoreRequest) returns (RegisterStoreResponse);
  rpc Heartbeat(HeartbeatRequest) returns (HeartbeatResponse);
}
```

#### Region Discovery

```protobuf
message GetTableRegionsRequest {
  string table_name = 1;
}

message GetTableRegionsResponse {
  repeated Region regions = 1;
}

message Region {
  uint32 region_id = 1;
  bytes start_key = 2;
  bytes end_key = 3;
  string leader_node = 4;
  repeated string followers = 5;
  RegionState state = 6;
}
```

## Ports

| Service | Default Port |
|---------|--------------|
| PD gRPC | 50051 |
| Store gRPC | 50052 |

## Arrow Format

Scan results use Apache Arrow IPC stream format:

```
Schema: [key: Binary, value: Binary]
```

BatchGet results:

```
Schema: [key: Binary, value: Binary, found: Boolean]
```

## Usage Example

```rust
// Rust client
let mut client = DataAccessClient::connect("http://localhost:50052").await?;

let request = ScanRequest {
    region_id: 1,
    start_key: b"user:".to_vec(),
    end_key: b"user:~".to_vec(),
    batch_size: 1000,
    limit: 0,
    read_follower: false,
};

let mut stream = client.scan(request).await?.into_inner();
while let Some(response) = stream.message().await? {
    let batch = RecordBatch::try_from(&response.arrow_batch)?;
    // Process batch
}
```
