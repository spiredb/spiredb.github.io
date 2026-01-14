# Vector Commands

Redis Search compatible vector similarity search using [Anodex](https://github.com/spiredb/anodex).

SpireDB supports both **ANODE** (single-shard) and **MANODE** (multi-shard) algorithms for approximate nearest neighbor (ANN) search.

---

## FT.CREATE

Create a vector index.

**Syntax**
```
FT.CREATE index_name ON table_name SCHEMA column_name VECTOR algorithm shards DIM dimensions
```

**Parameters**
| Parameter | Description |
|-----------|-------------|
| `index_name` | Unique index identifier |
| `table_name` | Source table name |
| `column_name` | Column containing vectors |
| `algorithm` | `ANODE` or `MANODE` |
| `shards` | Number of shards (MANODE) |
| `dimensions` | Vector dimensionality |

**Example**
```bash
# Create 128-dimensional index with MANODE
FT.CREATE embeddings_idx ON documents SCHEMA embedding VECTOR MANODE 4 DIM 128
# OK

# Create 768-dimensional index for sentence embeddings
FT.CREATE sentence_idx ON articles SCHEMA vector VECTOR ANODE 1 DIM 768
# OK
```

---

## FT.DROPINDEX

Drop a vector index.

**Syntax**
```
FT.DROPINDEX index_name
```

**Example**
```bash
FT.DROPINDEX embeddings_idx
# OK
```

---

## FT.ADD

Add a vector to an index.

**Syntax**
```
FT.ADD index_name doc_id vector_bytes [PAYLOAD json]
```

**Parameters**
| Parameter | Description |
|-----------|-------------|
| `index_name` | Target index |
| `doc_id` | Unique document identifier |
| `vector_bytes` | Float32 vector as binary |
| `PAYLOAD json` | Optional metadata JSON |

**Example**
```bash
# Add vector with payload
FT.ADD embeddings_idx doc:1 "\x00\x00\x80?\x00\x00\x00@..." PAYLOAD '{"title":"Hello World"}'
# 1

# Python example for encoding vectors
# import struct
# vector = [1.0, 2.0, 3.0, ...]
# vector_bytes = struct.pack(f'{len(vector)}f', *vector)
```

---

## FT.DEL

Delete a vector from an index.

**Syntax**
```
FT.DEL index_name doc_id
```

**Example**
```bash
FT.DEL embeddings_idx doc:1
# 1
```

---

## FT.SEARCH

Search for nearest neighbors.

**Syntax**
```
FT.SEARCH index_name "*=>[KNN k @column $BLOB]" BLOB vector_bytes [RETURN_PAYLOAD]
```

**Parameters**
| Parameter | Description |
|-----------|-------------|
| `k` | Number of results |
| `BLOB vector_bytes` | Query vector |
| `RETURN_PAYLOAD` | Include stored payloads |

**Returns**
- `[count, doc_id, [fields...], ...]`

**Example**
```bash
# Find 10 nearest neighbors
FT.SEARCH embeddings_idx "*=>[KNN 10 @embedding $BLOB]" BLOB "\x00\x00\x80?..."
# 1) (integer) 10
# 2) "doc:42"
# 3) 1) "__distance"
#    2) "0.123"
# 4) "doc:17"
# ...

# With payloads
FT.SEARCH embeddings_idx "*=>[KNN 5 @embedding $BLOB]" BLOB "..." RETURN_PAYLOAD
# 1) (integer) 5
# 2) "doc:42"
# 3) 1) "__distance"
#    2) "0.123"
#    3) "__payload"
#    4) '{"title":"Hello World"}'
```

---

## FT.INFO

Get index information.

**Syntax**
```
FT.INFO index_name
```

**Returns**
- Index metadata array

**Example**
```bash
FT.INFO embeddings_idx
# 1) "index_name"
# 2) "embeddings_idx"
# 3) "table_name"
# 4) "documents"
# 5) "column_name"
# 6) "embedding"
# 7) "algorithm"
# 8) "manode"
# 9) "dimensions"
# 10) (integer) 128
```

---

## FT._LIST

List all vector indexes.

**Syntax**
```
FT._LIST
```

**Returns**
- Array of index names

**Example**
```bash
FT._LIST
# 1) "embeddings_idx"
# 2) "sentence_idx"
```

---

## Client Examples

### Python
```python
import redis
import struct
import numpy as np

r = redis.Redis()

# Create index
r.execute_command('FT.CREATE', 'my_idx', 'ON', 'docs', 
                  'SCHEMA', 'vec', 'VECTOR', 'MANODE', '4', 'DIM', '128')

# Insert vector
vector = np.random.rand(128).astype(np.float32)
vector_bytes = vector.tobytes()
r.execute_command('FT.ADD', 'my_idx', 'doc:1', vector_bytes, 
                  'PAYLOAD', '{"title": "Example"}')

# Search
query = np.random.rand(128).astype(np.float32)
results = r.execute_command('FT.SEARCH', 'my_idx', 
                            '*=>[KNN 10 @vec $BLOB]', 
                            'BLOB', query.tobytes())
```

### Elixir
```elixir
{:ok, conn} = Redix.start_link()

# Create index
Redix.command(conn, ["FT.CREATE", "my_idx", "ON", "docs", 
              "SCHEMA", "vec", "VECTOR", "MANODE", "4", "DIM", "128"])

# Insert vector
vector = :rand.uniform(128) |> Enum.map(fn _ -> :rand.uniform() end)
vector_bytes = vector |> Enum.map(&<<&1::float-32-native>>) |> IO.iodata_to_binary()
Redix.command(conn, ["FT.ADD", "my_idx", "doc:1", vector_bytes])

# Search
Redix.command(conn, ["FT.SEARCH", "my_idx", "*=>[KNN 10 @vec $BLOB]", 
              "BLOB", query_bytes])
```
