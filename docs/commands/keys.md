# Key Commands

Key management operations.

---

## DEL

Delete one or more keys.

**Syntax**
```
DEL key [key ...]
```

**Returns**
- Number of keys deleted

**Example**
```bash
SET k1 "v1"
SET k2 "v2"
SET k3 "v3"

DEL k1 k2 k4
# 2 (k1 and k2 deleted, k4 didn't exist)
```

---

## EXISTS

Check if keys exist.

**Syntax**
```
EXISTS key [key ...]
```

**Returns**
- Number of keys that exist

**Example**
```bash
SET k1 "v1"
SET k2 "v2"

EXISTS k1
# 1

EXISTS k1 k2 k3
# 2 (k1 and k2 exist)

EXISTS nonexistent
# 0
```
