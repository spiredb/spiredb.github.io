# Redis API

SpireDB implements RESP (Redis Serialization Protocol) for client compatibility.

## Connection

```bash
redis-cli -h localhost -p 6379
```

## Supported Commands

### String Operations

| Command | Syntax | Description |
|---------|--------|-------------|
| `GET` | `GET key` | Get value |
| `SET` | `SET key value [EX s] [PX ms]` | Set value |
| `DEL` | `DEL key [key ...]` | Delete keys |
| `EXISTS` | `EXISTS key [key ...]` | Check existence |
| `MGET` | `MGET key [key ...]` | Get multiple |
| `MSET` | `MSET key value [key value ...]` | Set multiple |

### Counters

| Command | Syntax | Description |
|---------|--------|-------------|
| `INCR` | `INCR key` | Increment by 1 |
| `DECR` | `DECR key` | Decrement by 1 |
| `INCRBY` | `INCRBY key increment` | Increment by N |
| `DECRBY` | `DECRBY key decrement` | Decrement by N |

### String Manipulation

| Command | Syntax | Description |
|---------|--------|-------------|
| `APPEND` | `APPEND key value` | Append to value |
| `STRLEN` | `STRLEN key` | Get string length |

### Server

| Command | Syntax | Description |
|---------|--------|-------------|
| `PING` | `PING [message]` | Health check |
| `COMMAND` | `COMMAND` | List commands |
| `FLUSHALL` | `FLUSHALL` | Clear all data |

## Examples

```bash
# Basic operations
SET user:1 '{"name":"alice"}'
GET user:1

# Counters
SET counter 0
INCR counter
INCRBY counter 10

# Multi-key
MSET k1 v1 k2 v2 k3 v3
MGET k1 k2 k3

# Check existence
EXISTS user:1 user:2
DEL user:1 user:2
```

## Client Libraries

Any Redis client works:

```python
# Python
import redis
r = redis.Redis(host='localhost', port=6379)
r.set('key', 'value')
```

```javascript
// Node.js
const Redis = require('ioredis');
const redis = new Redis(6379, 'localhost');
await redis.set('key', 'value');
```

```elixir
# Elixir
{:ok, conn} = Redix.start_link(host: "localhost", port: 6379)
Redix.command(conn, ["SET", "key", "value"])
```

## Limitations

- No TTL persistence (EX/PX parsed but not enforced)
- No pub/sub
- No Lua scripting
- No transactions (MULTI/EXEC)
- No cluster mode commands
