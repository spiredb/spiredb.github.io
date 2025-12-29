# Troubleshooting

## Common Issues

### Connection Refused (6379)

- **Check Port**: Ensure `SPIRE_RESP_PORT` is set correctly (default 6379).
- **Check Logs**: Look for "Failed to start Ranch listener".

### Cluster Warning: "Node not running"

- **Check Cookies**: Ensure all nodes share the same Erlang cookie.
- **Check Networking**: Verify connectivity on port 4369 (EPMD) and the distribution port range.

### "No leader elected"

- **Check Quorum**: Ensure at least `(N/2)+1` nodes are online.
- **Check Logs**: Look for Raft election timeouts. Decrease `SPIRE_RAFT_ELECTION_TIMEOUT` if network latency is high.
- **Data Dir**: Ensure `SPIRE_RAFT_DATA_DIR` is writable and persistent.

## Diagnostics

### Check Cluster Membership

Connect to the console:

```bash
./bin/spiredb_store remote_console
```

Run:

```elixir
Node.list()
# [:spiredb@node2, :spiredb@node3]
```

### Check Raft Status

```elixir
:ra.members({:region_1, node()})
```

## Logs

Set log level to debug for more info:

```bash
export SPIRE_LOG_LEVEL=debug
```
