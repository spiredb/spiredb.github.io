# Stream Commands

Redis Streams for event streaming and message queues.

---

## XADD

Append an entry to a stream.

**Syntax**
```
XADD key [NOMKSTREAM] [MAXLEN [~] count] *|id field value [field value ...]
```

**Options**
| Option | Description |
|--------|-------------|
| `*` | Auto-generate ID (timestamp-sequence) |
| `MAXLEN count` | Cap stream length |
| `MAXLEN ~ count` | Approximate cap (more efficient) |
| `NOMKSTREAM` | Don't create stream if missing |

**Returns**
- The generated/provided entry ID

**Example**
```bash
XADD mystream * sensor_id 1234 temperature 19.8
# "1704067200000-0"

XADD mystream MAXLEN ~ 1000 * event "login" user "alice"
# "1704067200001-0"
```

---

## XREAD

Read entries from streams.

**Syntax**
```
XREAD [COUNT count] [BLOCK ms] STREAMS key [key ...] id [id ...]
```

**Options**
| Option | Description |
|--------|-------------|
| `COUNT count` | Max entries per stream |
| `BLOCK ms` | Block for new entries (0 = forever) |

**Returns**
- Array of [stream, entries] pairs

**Example**
```bash
# Read all new entries from stream
XREAD STREAMS mystream 0

# Read up to 10 entries after ID 1704067200000-0
XREAD COUNT 10 STREAMS mystream 1704067200000-0

# Block and wait for new entries
XREAD BLOCK 5000 STREAMS mystream $
```

---

## XRANGE

Get entries in ID range.

**Syntax**
```
XRANGE key start end [COUNT count]
```

**Special IDs**
| ID | Meaning |
|----|---------|
| `-` | Minimum ID |
| `+` | Maximum ID |

**Returns**
- Array of entries

**Example**
```bash
# Get all entries
XRANGE mystream - +

# Get first 10 entries
XRANGE mystream - + COUNT 10

# Get entries in time range
XRANGE mystream 1704067200000 1704153600000
```

---

## XREVRANGE

Get entries in reverse order.

**Syntax**
```
XREVRANGE key end start [COUNT count]
```

**Example**
```bash
# Get last 10 entries
XREVRANGE mystream + - COUNT 10
```

---

## XLEN

Get stream length.

**Syntax**
```
XLEN key
```

**Returns**
- Number of entries

**Example**
```bash
XLEN mystream
# 1523
```

---

## XINFO STREAM

Get stream information.

**Syntax**
```
XINFO STREAM key
```

**Returns**
- Stream metadata array

**Example**
```bash
XINFO STREAM mystream
# 1) "length"
# 2) (integer) 1523
# 3) "first-entry"
# 4) 1) "1704067200000-0"
#    2) 1) "sensor_id"
#       2) "1234"
# ...
```

---

## XTRIM

Trim stream to a maximum length.

**Syntax**
```
XTRIM key MAXLEN [~] count
```

**Returns**
- Number of entries removed

**Example**
```bash
XTRIM mystream MAXLEN 1000
# 523

XTRIM mystream MAXLEN ~ 1000
# (approximate trim, faster)
```

---

## XDEL

Delete entries by ID.

**Syntax**
```
XDEL key id [id ...]
```

**Returns**
- Number of entries deleted

**Example**
```bash
XDEL mystream 1704067200000-0 1704067200001-0
# 2
```
