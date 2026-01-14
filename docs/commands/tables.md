# Table Commands

SpireDB-specific DDL commands for schema management.

---

## SPIRE.TABLE.CREATE

Create a new table.

**Syntax**
```
SPIRE.TABLE.CREATE table_name (column_name type [NOT NULL] [DEFAULT value], ...) PRIMARY KEY (pk_columns)
```

**Supported Types**
| Type | Description |
|------|-------------|
| `INT8`, `INT16`, `INT32`, `INT64` | Signed integers |
| `UINT8`, `UINT16`, `UINT32`, `UINT64` | Unsigned integers |
| `FLOAT32`, `FLOAT64` | Floating point |
| `BOOL` | Boolean |
| `STRING` | Variable-length string |
| `BYTES` | Binary data |
| `DATE` | Date (days since epoch) |
| `TIMESTAMP` | Timestamp (microseconds) |
| `DECIMAL(precision, scale)` | Exact decimal |
| `VECTOR(dim)` | Float32 vector |
| `LIST(element_type)` | Typed array |

**Example**
```bash
# Simple table
SPIRE.TABLE.CREATE users (
  id INT64 NOT NULL,
  name STRING NOT NULL,
  email STRING,
  created_at TIMESTAMP DEFAULT NOW()
) PRIMARY KEY (id)
# ["OK", 1]

# Table with composite primary key
SPIRE.TABLE.CREATE order_items (
  order_id INT64 NOT NULL,
  line_num INT32 NOT NULL,
  product_id INT64,
  quantity INT32 DEFAULT 1,
  price DECIMAL(10,2)
) PRIMARY KEY (order_id, line_num)
# ["OK", 2]

# Table with vector column
SPIRE.TABLE.CREATE documents (
  id INT64 NOT NULL,
  content STRING,
  embedding VECTOR(768)
) PRIMARY KEY (id)
# ["OK", 3]
```

---

## SPIRE.TABLE.DROP

Drop a table.

**Syntax**
```
SPIRE.TABLE.DROP table_name
```

**Example**
```bash
SPIRE.TABLE.DROP users
# OK
```

---

## SPIRE.TABLE.LIST

List all tables.

**Syntax**
```
SPIRE.TABLE.LIST
```

**Returns**
- Array of table names

**Example**
```bash
SPIRE.TABLE.LIST
# 1) "users"
# 2) "order_items"
# 3) "documents"
```

---

## SPIRE.TABLE.DESCRIBE

Get table schema.

**Syntax**
```
SPIRE.TABLE.DESCRIBE table_name
```

**Returns**
- Table with columns: Column, Type, Nullable, Default, Extra

**Example**
```bash
SPIRE.TABLE.DESCRIBE users
# 1) 1) "Column"
#    2) "Type"
#    3) "Nullable"
#    4) "Default"
#    5) "Extra"
# 2) 1) "id"
#    2) "INT64"
#    3) "NO"
#    4) ""
#    5) ""
# 3) 1) "name"
#    2) "STRING"
#    3) "NO"
#    4) ""
#    5) ""
# ...
# N) 1) "PRIMARY KEY"
#    2) "id"
#    3) ""
#    4) ""
#    5) ""
```

---

## SPIRE.INDEX.CREATE

Create an index on a table.

**Syntax**
```
SPIRE.INDEX.CREATE index_name ON table_name (columns) [USING BTREE|ANODE|MANODE]
```

**Index Types**
| Type | Description |
|------|-------------|
| `BTREE` | B-tree index (default) |
| `ANODE` | Single-shard vector index |
| `MANODE` | Multi-shard vector index |

**Example**
```bash
# B-tree index on email
SPIRE.INDEX.CREATE idx_users_email ON users (email)
# ["OK", 1]

# Composite index
SPIRE.INDEX.CREATE idx_orders_date ON orders (customer_id, order_date)
# ["OK", 2]

# Vector index
SPIRE.INDEX.CREATE idx_doc_embedding ON documents (embedding) USING MANODE
# ["OK", 3]
```

---

## SPIRE.INDEX.DROP

Drop an index.

**Syntax**
```
SPIRE.INDEX.DROP index_name
```

**Example**
```bash
SPIRE.INDEX.DROP idx_users_email
# OK
```
