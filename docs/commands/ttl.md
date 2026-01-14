# TTL Commands

Time-to-live (expiry) operations.

---

## TTL

Get remaining time-to-live in seconds.

**Syntax**
```
TTL key
```

**Returns**
- TTL in seconds
- `-1` if key exists but has no TTL
- `-2` if key does not exist

**Example**
```bash
SET mykey "value" EX 60
TTL mykey
# 58

SET permanent "value"
TTL permanent
# -1

TTL nonexistent
# -2
```

---

## PTTL

Get remaining time-to-live in milliseconds.

**Syntax**
```
PTTL key
```

**Returns**
- TTL in milliseconds
- `-1` if key exists but has no TTL
- `-2` if key does not exist

**Example**
```bash
SET mykey "value" PX 5000
PTTL mykey
# 4832
```

---

## EXPIRE

Set a key's TTL in seconds.

**Syntax**
```
EXPIRE key seconds
```

**Returns**
- `1` if timeout was set
- `0` if key does not exist

**Example**
```bash
SET mykey "value"
EXPIRE mykey 60
# 1

TTL mykey
# 60

EXPIRE nonexistent 60
# 0
```

---

## PEXPIRE

Set a key's TTL in milliseconds.

**Syntax**
```
PEXPIRE key milliseconds
```

**Returns**
- `1` if timeout was set
- `0` if key does not exist

**Example**
```bash
SET mykey "value"
PEXPIRE mykey 5000
# 1

PTTL mykey
# 5000
```

---

## EXPIREAT

Set expiry as Unix timestamp (seconds).

**Syntax**
```
EXPIREAT key unix-time-seconds
```

**Returns**
- `1` if timeout was set
- `0` if key does not exist

**Example**
```bash
SET mykey "value"
EXPIREAT mykey 1893456000
# 1 (expires Jan 1, 2030)
```

---

## PERSIST

Remove the TTL from a key.

**Syntax**
```
PERSIST key
```

**Returns**
- `1` if TTL was removed
- `0` if key does not exist or has no TTL

**Example**
```bash
SET mykey "value" EX 60
TTL mykey
# 60

PERSIST mykey
# 1

TTL mykey
# -1 (no expiry)
```
