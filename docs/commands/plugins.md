# Plugin Commands

Plugin management via RESP.

---

## SPIRE.PLUGIN.LIST

List all loaded plugins.

**Syntax**
```
SPIRE.PLUGIN.LIST
```

**Returns**
- Array of `[name, type, version]` tuples

**Example**
```bash
SPIRE.PLUGIN.LIST
# 1) 1) "geo_index"
#    2) "index"
#    3) "1.0.0"
# 2) 1) "jwt_auth"
#    2) "auth"
#    3) "2.1.0"
```

---

## SPIRE.PLUGIN.INFO

Get detailed plugin information.

**Syntax**
```
SPIRE.PLUGIN.INFO plugin_name
```

**Returns**
- Plugin metadata array

**Example**
```bash
SPIRE.PLUGIN.INFO geo_index
# 1) "name"
# 2) "geo_index"
# 3) "type"
# 4) "index"
# 5) "version"
# 6) "1.0.0"
# 7) "description"
# 8) "Geospatial index plugin for location queries"
# 9) "has_nif"
# 10) "true"
```

---

## SPIRE.PLUGIN.RELOAD

Hot-reload a plugin.

**Syntax**
```
SPIRE.PLUGIN.RELOAD plugin_name
```

**Returns**
- `OK` on success

**Example**
```bash
SPIRE.PLUGIN.RELOAD geo_index
# OK
```

---

## Plugin Types

SpireDB supports 5 plugin types:

| Type | Description |
|------|-------------|
| `index` | Custom index implementations |
| `storage` | Storage engine extensions |
| `function` | Custom SQL functions |
| `protocol` | Wire protocol extensions |
| `auth` | Authentication/authorization |

See [Plugin System](/plugin-system.md) for writing plugins.
