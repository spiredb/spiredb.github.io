# Transaction Commands

Distributed transaction support with Percolator-style 2PC.

SpireDB provides ACID transactions with configurable isolation levels and savepoints.

---

## MULTI

Start a transaction.

**Syntax**
```
MULTI [ISOLATION level] [LOCK PESSIMISTIC]
```

**Isolation Levels**
| Level | Description |
|-------|-------------|
| `READ_COMMITTED` | See committed data only |
| `REPEATABLE_READ` | Snapshot at transaction start (default) |
| `SERIALIZABLE` | Full serializability via SSI |

**Options**
| Option | Description |
|--------|-------------|
| `LOCK PESSIMISTIC` | Acquire locks upfront |

**Returns**
- `OK`

**Example**
```bash
# Default transaction
MULTI
# OK

# Serializable isolation
MULTI ISOLATION SERIALIZABLE
# OK

# Pessimistic locking
MULTI LOCK PESSIMISTIC
# OK
```

---

## EXEC

Commit a transaction.

**Syntax**
```
EXEC
```

**Returns**
- `["OK", commit_timestamp]` on success
- Error on conflict

**Example**
```bash
MULTI
# OK
SET account:1 "900"
# QUEUED
SET account:2 "1100"
# QUEUED
EXEC
# ["OK", 1704067200000]
```

---

## DISCARD

Rollback a transaction.

**Syntax**
```
DISCARD
```

**Returns**
- `OK`

**Example**
```bash
MULTI
# OK
SET key "value"
# QUEUED
DISCARD
# OK
# (nothing committed)
```

---

## SAVEPOINT

Create a named savepoint within a transaction.

**Syntax**
```
SAVEPOINT name
```

**Returns**
- `OK`

**Example**
```bash
MULTI
# OK
SET k1 "v1"
# QUEUED
SAVEPOINT sp1
# OK
SET k2 "v2"
# QUEUED
# ... can rollback to sp1 if needed
```

---

## ROLLBACK TO

Rollback to a savepoint.

**Syntax**
```
ROLLBACK TO savepoint_name
```

**Returns**
- `OK`
- Error if savepoint not found

**Example**
```bash
MULTI
# OK
SET balance "1000"
# QUEUED
SAVEPOINT before_update
# OK
SET balance "500"
# QUEUED
# Oops, wrong amount
ROLLBACK TO before_update
# OK
SET balance "750"
# QUEUED
EXEC
# ["OK", 1704067200000]
# balance is now "750"
```

---

## Transaction Behavior

### Within Transactions

Inside a `MULTI` block, commands behave differently:

| Command | Behavior |
|---------|----------|
| `GET` | Reads from snapshot |
| `SET` | Returns `QUEUED`, buffered |
| `DEL` | Returns `QUEUED`, buffered |

### Conflict Detection

SpireDB uses optimistic concurrency control:

1. **START_TS**: Assigned when `MULTI` is called
2. **WRITES**: Buffered until `EXEC`
3. **COMMIT_TS**: Assigned during `EXEC`
4. **VALIDATION**: Check for write-write conflicts

If another transaction modified keys you wrote, you'll get:
```
ERR write conflict on key 'key_name'
```

### Example: Bank Transfer

```bash
# Transfer $100 from account 1 to account 2
MULTI
# OK
GET account:1
# "1000"
GET account:2
# "500"
SET account:1 "900"
# QUEUED
SET account:2 "600"
# QUEUED
EXEC
# ["OK", 1704067200000]
```

### Example: Serializable Isolation

```bash
# Ensure no phantom reads
MULTI ISOLATION SERIALIZABLE
# OK
GET inventory:item1
# "10"
# ... complex logic ...
SET inventory:item1 "9"
# QUEUED
EXEC
# If another txn modified inventory:item1, this fails
```
