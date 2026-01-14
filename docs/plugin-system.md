# Plugin System

SpireDB supports 5 plugin types for extending functionality.

## Plugin Types

| Type | Module | Purpose |
|------|--------|---------|
| **Index** | `Store.Plugin.Index` | Custom index implementations |
| **Storage** | `Store.Plugin.StorageEngine` | Storage backends |
| **Function** | `Store.Plugin.Function` | Custom SQL functions |
| **Protocol** | `Store.Plugin.Protocol` | Wire protocol extensions |
| **Auth** | `Store.Plugin.Auth` | Authentication/authorization |

---

## Base Plugin Behaviour

All plugins implement `Store.Plugin`:

```elixir
@callback info() :: %{
  name: String.t(),
  version: String.t(),
  type: :index | :storage | :function | :protocol | :auth,
  description: String.t(),
  has_nif: boolean()
}

@callback init(opts :: keyword()) :: {:ok, state} | {:error, reason}
@callback shutdown(state) :: :ok
```

---

## Index Plugins

Create custom index types (geospatial, full-text, etc).

### Behaviour

```elixir
@callback create(name :: String.t(), opts :: keyword()) :: :ok | {:error, term()}
@callback drop(name :: String.t()) :: :ok | {:error, term()}
@callback insert(name :: String.t(), id :: term(), data :: term()) :: :ok | {:error, term()}
@callback delete(name :: String.t(), id :: term()) :: :ok | {:error, term()}
@callback search(name :: String.t(), query :: term(), opts :: keyword()) ::
            {:ok, [result]} | {:error, term()}
```

### Example: Geospatial Index

```elixir
defmodule MyApp.GeoIndex do
  @behaviour Store.Plugin
  @behaviour Store.Plugin.Index
  
  # Plugin metadata
  def info do
    %{
      name: "geo_index",
      version: "1.0.0",
      type: :index,
      description: "R-tree geospatial index",
      has_nif: false
    }
  end
  
  def init(_opts) do
    {:ok, %{indexes: %{}}}
  end
  
  def shutdown(_state), do: :ok
  
  # Index implementation
  def create(name, opts) do
    # Initialize R-tree structure
    :ok
  end
  
  def drop(name) do
    :ok
  end
  
  def insert(name, id, %{lat: lat, lng: lng}) do
    # Insert into R-tree
    :ok
  end
  
  def delete(name, id) do
    :ok
  end
  
  def search(name, %{lat: lat, lng: lng, radius_km: r}, _opts) do
    # Query R-tree for points within radius
    {:ok, []}
  end
end
```

---

## Storage Plugins

Custom storage backends (encryption, tiered storage, etc).

### Behaviour

```elixir
@callback init(opts :: keyword()) :: {:ok, state} | {:error, term()}
@callback get(key :: binary(), state) :: {:ok, binary()} | {:error, :not_found | term()}
@callback put(key :: binary(), value :: binary(), state) :: {:ok, state} | {:error, term()}
@callback delete(key :: binary(), state) :: {:ok, state} | {:error, term()}
@callback scan(start_key :: binary(), end_key :: binary(), opts :: keyword(), state) ::
            {:ok, Enumerable.t()} | {:error, term()}
@callback flush(state) :: {:ok, state} | {:error, term()}  # optional
@callback shutdown(state) :: :ok
```

### Example: Encrypted Storage

```elixir
defmodule MyApp.EncryptedStorage do
  @behaviour Store.Plugin
  @behaviour Store.Plugin.StorageEngine
  
  def info do
    %{
      name: "encrypted_storage",
      version: "1.0.0",
      type: :storage,
      description: "AES-256-GCM encrypted storage layer",
      has_nif: true
    }
  end
  
  def init(opts) do
    key = Keyword.fetch!(opts, :encryption_key)
    {:ok, %{key: key, inner: Keyword.get(opts, :inner_storage)}}
  end
  
  def get(key, %{key: enc_key, inner: inner} = state) do
    case inner.get(key, inner) do
      {:ok, ciphertext} ->
        {:ok, decrypt(ciphertext, enc_key)}
      error ->
        error
    end
  end
  
  def put(key, value, %{key: enc_key, inner: inner} = state) do
    ciphertext = encrypt(value, enc_key)
    case inner.put(key, ciphertext, inner) do
      {:ok, _} -> {:ok, state}
      error -> error
    end
  end
  
  defp encrypt(data, key), do: :crypto.crypto_one_time_aead(:aes_256_gcm, key, ...)
  defp decrypt(data, key), do: :crypto.crypto_one_time_aead(:aes_256_gcm, key, ...)
end
```

---

## Function Plugins

Custom SQL functions.

### Behaviour

```elixir
@callback functions() :: [{name :: String.t(), arity :: non_neg_integer(), return_type :: atom()}]
@callback execute(name :: String.t(), args :: list()) :: {:ok, term()} | {:error, term()}
```

### Example: Statistical Functions

```elixir
defmodule MyApp.StatsFunctions do
  @behaviour Store.Plugin
  @behaviour Store.Plugin.Function
  
  def info do
    %{
      name: "stats_functions",
      version: "1.0.0",
      type: :function,
      description: "Statistical aggregation functions",
      has_nif: false
    }
  end
  
  def init(_opts), do: {:ok, %{}}
  def shutdown(_state), do: :ok
  
  def functions do
    [
      {"median", 1, :float64},
      {"percentile", 2, :float64},
      {"stddev", 1, :float64}
    ]
  end
  
  def execute("median", [values]) when is_list(values) do
    sorted = Enum.sort(values)
    len = length(sorted)
    mid = div(len, 2)
    
    result = if rem(len, 2) == 0 do
      (Enum.at(sorted, mid - 1) + Enum.at(sorted, mid)) / 2
    else
      Enum.at(sorted, mid)
    end
    
    {:ok, result}
  end
  
  def execute("percentile", [values, p]) when is_list(values) do
    sorted = Enum.sort(values)
    idx = round(p / 100 * (length(sorted) - 1))
    {:ok, Enum.at(sorted, idx)}
  end
  
  def execute("stddev", [values]) when is_list(values) do
    mean = Enum.sum(values) / length(values)
    variance = Enum.map(values, fn v -> (v - mean) ** 2 end) |> Enum.sum() |> Kernel./(length(values))
    {:ok, :math.sqrt(variance)}
  end
end
```

---

## Protocol Plugins

Custom wire protocols (GraphQL, WebSocket, etc).

### Behaviour

```elixir
@callback commands() :: [{name :: String.t(), arity :: non_neg_integer(), description :: String.t()}]
@callback parse(data :: binary(), state) ::
            {:ok, command, remaining :: binary(), state}
            | {:incomplete, state}
            | {:error, reason}
@callback execute(command, state) :: {:ok, response, state} | {:error, reason, state}
@callback encode(response) :: binary()
@callback on_connect(opts :: keyword()) :: {:ok, state} | {:error, term()}  # optional
@callback on_disconnect(state) :: :ok  # optional
```

### Example: GraphQL Protocol

```elixir
defmodule MyApp.GraphQLProtocol do
  @behaviour Store.Plugin
  @behaviour Store.Plugin.Protocol
  
  def info do
    %{
      name: "graphql_protocol",
      version: "1.0.0",
      type: :protocol,
      description: "GraphQL query endpoint",
      has_nif: false
    }
  end
  
  def init(_opts), do: {:ok, %{}}
  def shutdown(_state), do: :ok
  
  def commands do
    [
      {"query", 1, "Execute GraphQL query"},
      {"mutation", 1, "Execute GraphQL mutation"}
    ]
  end
  
  def on_connect(_opts), do: {:ok, %{}}
  def on_disconnect(_state), do: :ok
  
  def parse(data, state) do
    case Jason.decode(data) do
      {:ok, %{"query" => query}} ->
        {:ok, {:query, query}, "", state}
      {:error, _} ->
        {:incomplete, state}
    end
  end
  
  def execute({:query, query}, state) do
    result = Absinthe.run(query, MyApp.Schema)
    {:ok, result, state}
  end
  
  def encode(response) do
    Jason.encode!(response)
  end
end
```

---

## Auth Plugins

Authentication and authorization.

### Behaviour

```elixir
@callback authenticate(credentials :: map()) :: {:ok, user :: map()} | {:error, term()}
@callback authorize(user :: map(), operation :: atom(), resource :: String.t()) ::
            :allow | :deny | {:deny, reason :: String.t()}
@callback has_permission?(user :: map(), permission :: atom()) :: boolean()  # optional
@callback refresh(user :: map()) :: {:ok, user :: map()} | {:error, term()}  # optional
@callback invalidate(user :: map()) :: :ok  # optional
```

### Example: JWT Auth

```elixir
defmodule MyApp.JWTAuth do
  @behaviour Store.Plugin
  @behaviour Store.Plugin.Auth
  
  def info do
    %{
      name: "jwt_auth",
      version: "2.0.0",
      type: :auth,
      description: "JWT-based authentication",
      has_nif: false
    }
  end
  
  def init(opts) do
    secret = Keyword.fetch!(opts, :jwt_secret)
    {:ok, %{secret: secret}}
  end
  
  def shutdown(_state), do: :ok
  
  def authenticate(%{token: token}) do
    case JOSE.JWT.verify(token, @secret) do
      {true, %{fields: claims}, _} ->
        {:ok, %{
          id: claims["sub"],
          roles: claims["roles"] || [],
          exp: claims["exp"]
        }}
      _ ->
        {:error, :invalid_token}
    end
  end
  
  def authorize(%{roles: roles}, :write, "admin:" <> _) do
    if "admin" in roles, do: :allow, else: {:deny, "Admin role required"}
  end
  
  def authorize(%{roles: roles}, :write, _resource) do
    if "writer" in roles or "admin" in roles, do: :allow, else: :deny
  end
  
  def authorize(_user, :read, _resource), do: :allow
  
  def has_permission?(%{roles: roles}, permission) do
    case permission do
      :admin -> "admin" in roles
      :write -> "writer" in roles or "admin" in roles
      :read -> true
    end
  end
  
  def refresh(%{id: id, roles: roles}) do
    new_token = JOSE.JWT.sign(@secret, %{"sub" => id, "roles" => roles})
    {:ok, %{id: id, roles: roles, token: new_token}}
  end
  
  def invalidate(_user), do: :ok
end
```

---

## Installing Plugins

### Via RESP
```bash
# From Hex.pm
SPIRE.PLUGIN.INSTALL hex:my_plugin

# From GitHub
SPIRE.PLUGIN.INSTALL github:user/spiredb-plugin
```

### Via gRPC
```protobuf
rpc InstallPlugin(InstallPluginRequest) returns (InstallPluginResponse);
```

### Via Configuration
```elixir
# config/config.exs
config :spiredb_store, :plugins, [
  {MyApp.GeoIndex, []},
  {MyApp.JWTAuth, [jwt_secret: System.get_env("JWT_SECRET")]}
]
```

---

## Plugin Development Guide

1. **Create Mix project**
   ```bash
   mix new my_plugin
   ```

2. **Add SpireDB dependency**
   ```elixir
   {:spiredb_store, "~> 0.1.0", only: :dev}
   ```

3. **Implement behaviours**
   - `Store.Plugin` (required)
   - Type-specific behaviour

4. **Test locally**
   ```elixir
   {:ok, state} = MyPlugin.init([])
   MyPlugin.search("idx", %{query: "..."}, [])
   ```

5. **Publish to Hex**
   ```bash
   mix hex.publish
   ```
