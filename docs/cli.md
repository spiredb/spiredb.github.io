# Spire CLI

The `spire` command-line tool provides cluster management, plugin operations, and data access utilities.

## Installation

`spire-cli` is included in the standard SpireDB distribution.

```bash
cargo install --path spire-cli
```

## Global Options

| Option | Default | Description |
|--------|---------|-------------|
| `--pd-addr` | `http://127.0.0.1:50051` | Address of the Placement Driver (PD) server |
| `--data-addr` | `http://127.0.0.1:50052` | Address of the Data Access server |
| `--spiresql-addr` | `127.0.0.1:5432` | Address of the SpireSQL Postgres interface |

## Commands

### Cluster Management

Manage cluster membership and status.

```bash
# Show cluster status
spire cluster status

# List cluster members
spire cluster members
```

### Plugin Management

Manage SpireDB plugins (extensions).

#### List Plugins

List all installed plugins.

```bash
spire plugin list
```

#### Install Plugin

Install a plugin from a registry, Git repository, or local path.

```bash
# From registry
spire plugin install hex:my_plugin

# From Git
spire plugin install git+https://github.com/user/repo.git

# From local tarball
spire plugin install ./my-plugin.tar.gz
```

#### Uninstall Plugin

Remove an installed plugin.

```bash
spire plugin uninstall my_plugin
```

#### Create Plugin

Generate a new plugin project scaffold.

```bash
spire plugin new my_plugin
```

#### Build Plugin

Build the plugin in the current directory.

```bash
cd my_plugin
spire plugin build
```

### Schema Management

Inspect the database schema.

```bash
# List all tables
spire schema list-tables

# List all indexes
spire schema list-indexes
```

### Data Access

Basic key-value operations.

```bash
# Get value
spire data get user:1001

# Set value
spire data put user:1001 '{"name": "Alice"}'
```

### SQL Shell

Start an interactive SQL shell (PostgreSQL-compatible) connected to SpireSQL.

```bash
spire sql
```
