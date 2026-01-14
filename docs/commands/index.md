# Command Reference

SpireDB supports 50+ commands via the Redis protocol (RESP). Connect with any Redis client.

```bash
redis-cli -h localhost -p 6379
```

## Command Categories

### [String Commands](strings.md)
Basic key-value operations.
- `GET`, `SET`, `MGET`, `MSET`
- `INCR`, `DECR`, `INCRBY`, `DECRBY`
- `APPEND`, `STRLEN`

### [Key Commands](keys.md)
Key management operations.
- `DEL`, `EXISTS`

### [TTL Commands](ttl.md)
Time-to-live operations.
- `TTL`, `PTTL`, `EXPIRE`, `PEXPIRE`, `EXPIREAT`, `PERSIST`

### [Stream Commands](streams.md)
Redis Streams for event streaming.
- `XADD`, `XREAD`, `XRANGE`, `XREVRANGE`
- `XLEN`, `XINFO`, `XTRIM`, `XDEL`

### [Vector Commands](vectors.md)
Redis Search compatible vector search.
- `FT.CREATE`, `FT.DROPINDEX`, `FT.ADD`, `FT.DEL`
- `FT.SEARCH`, `FT.INFO`, `FT._LIST`

### [Table Commands](tables.md)
SpireDB-specific DDL commands.
- `SPIRE.TABLE.CREATE`, `SPIRE.TABLE.DROP`
- `SPIRE.TABLE.LIST`, `SPIRE.TABLE.DESCRIBE`
- `SPIRE.INDEX.CREATE`, `SPIRE.INDEX.DROP`

### [Transaction Commands](transactions.md)
Distributed transaction support.
- `MULTI`, `EXEC`, `DISCARD`
- `SAVEPOINT`, `ROLLBACK TO`

### [Plugin Commands](plugins.md)
Plugin management.
- `SPIRE.PLUGIN.LIST`, `SPIRE.PLUGIN.INFO`, `SPIRE.PLUGIN.RELOAD`

### Server Commands

| Command | Description |
|---------|-------------|
| `PING [message]` | Health check, returns `PONG` or message |
| `COMMAND` | List available commands |
| `FLUSHALL` | Clear all data (dev only) |
