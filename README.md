# Orchestra Plugin Protocol

Protobuf schema defining the Orchestra plugin wire protocol.

## Overview

This package contains the canonical `.proto` definitions for all Orchestra plugin communication. Messages are sent over QUIC streams using length-delimited Protobuf framing (not gRPC).

## Messages

The protocol defines a request/response envelope (`PluginRequest`/`PluginResponse`) covering:

- **Lifecycle** -- Register, Boot, Shutdown, Health
- **Tools** -- ToolCall, ListTools
- **Storage** -- StorageRead, StorageWrite, StorageDelete, StorageList

## Wire Format

Each QUIC stream carries one request/response pair:

```
[4 bytes: big-endian uint32 length][N bytes: Protobuf message]
```

## Code Generation

Install [buf](https://buf.build/docs/installation) and run from the monorepo root:

```bash
make proto
```

This generates typed code for Go (in `gen-go/`). Future targets include Rust, Swift, Kotlin, and C#.

## File Structure

```
proto/
└── orchestra/plugin/v1/
    └── plugin.proto          # All message and enum definitions
```

## Related Packages

| Package | Description |
|---------|-------------|
| [gen-go](https://github.com/orchestra-mcp/gen-go) | Generated Go types |
| [sdk-go](https://github.com/orchestra-mcp/sdk-go) | Go plugin SDK |
| [orchestrator](https://github.com/orchestra-mcp/orchestrator) | Central hub |

## License

[MIT](LICENSE)
