# String Commands

Basic key-value operations for string data.

---

## GET

Get the value of a key.

**Syntax**
```
GET key
```

**Returns**
- The value if key exists
- `nil` if key does not exist

**Example**
```bash
SET user:1 '{"name":"alice"}'
GET user:1
# '{"name":"alice"}'

GET nonexistent
# (nil)
```

---

## SET

Set the value of a key.

**Syntax**
```
SET key value [EX seconds] [PX milliseconds] [NX|XX] [KEEPTTL]
```

**Options**
| Option | Description |
|--------|-------------|
| `EX seconds` | Set expiry in seconds |
| `PX milliseconds` | Set expiry in milliseconds |
| `NX` | Only set if key does not exist |
| `XX` | Only set if key exists |
| `KEEPTTL` | Retain existing TTL |

**Returns**
- `OK` on success
- `nil` if condition not met (NX/XX)

**Example**
```bash
SET mykey "Hello"
# OK

SET mykey "World" EX 60
# OK (expires in 60 seconds)

SET mykey "New" NX
# (nil) - key already exists

SET newkey "Value" NX
# OK - created new key
```

---

## MGET

Get values of multiple keys.

**Syntax**
```
MGET key [key ...]
```

**Returns**
- Array of values (nil for missing keys)

**Example**
```bash
SET k1 "v1"
SET k2 "v2"
MGET k1 k2 k3
# 1) "v1"
# 2) "v2"
# 3) (nil)
```

---

## MSET

Set multiple keys atomically.

**Syntax**
```
MSET key value [key value ...]
```

**Returns**
- `OK`

**Example**
```bash
MSET k1 "v1" k2 "v2" k3 "v3"
# OK
```

---

## INCR

Increment the integer value of a key by 1.

**Syntax**
```
INCR key
```

**Returns**
- The new value after increment

**Example**
```bash
SET counter 10
INCR counter
# 11

INCR newcounter
# 1 (initializes to 0, then increments)
```

---

## DECR

Decrement the integer value of a key by 1.

**Syntax**
```
DECR key
```

**Returns**
- The new value after decrement

**Example**
```bash
SET counter 10
DECR counter
# 9
```

---

## INCRBY

Increment the integer value of a key by N.

**Syntax**
```
INCRBY key increment
```

**Returns**
- The new value after increment

**Example**
```bash
SET counter 10
INCRBY counter 5
# 15

INCRBY counter -3
# 12
```

---

## DECRBY

Decrement the integer value of a key by N.

**Syntax**
```
DECRBY key decrement
```

**Returns**
- The new value after decrement

**Example**
```bash
SET counter 10
DECRBY counter 3
# 7
```

---

## APPEND

Append a value to an existing string.

**Syntax**
```
APPEND key value
```

**Returns**
- Length of the string after append

**Example**
```bash
SET greeting "Hello"
APPEND greeting " World"
# 11

GET greeting
# "Hello World"

APPEND newkey "Start"
# 5 (creates key if not exists)
```

---

## STRLEN

Get the length of a string value.

**Syntax**
```
STRLEN key
```

**Returns**
- Length of the string
- `0` if key does not exist

**Example**
```bash
SET mykey "Hello World"
STRLEN mykey
# 11

STRLEN nonexistent
# 0
```
