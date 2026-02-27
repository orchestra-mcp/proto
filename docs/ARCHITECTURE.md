# Architecture

## Overview

This repository defines the canonical Protobuf schema for the Orchestra plugin wire protocol. All Orchestra plugins -- regardless of language (Go, Rust, Swift, Kotlin, C#) -- communicate using these message types over QUIC streams.

This is **not** a gRPC service definition. Messages are exchanged using length-delimited Protobuf framing over raw bidirectional QUIC streams.

## Wire Format

Every message on the wire uses length-delimited framing:

```
[4 bytes: big-endian uint32 length][N bytes: Protobuf payload]
```

- The 4-byte header encodes the payload size as a big-endian unsigned 32-bit integer.
- The payload is a marshaled `PluginRequest` or `PluginResponse` Protobuf message.
- Maximum message size: 16 MB.

## Message Flow

Each RPC is a single bidirectional QUIC stream:

```
Client                         Server
  |                               |
  |--- open stream -------------->|
  |--- [4B len][PluginRequest] -->|
  |                               |  (dispatch to handler)
  |<-- [4B len][PluginResponse] --|
  |--- close stream ------------->|
```

## Request/Response Envelope

All communication uses a `PluginRequest`/`PluginResponse` pair with a shared `request_id` (UUIDv7) for correlation. The `oneof` field determines the operation:

### Request Types

| Field Number | Type | Category |
|---|---|---|
| 10 | `PluginManifest` (register) | Lifecycle |
| 11 | `BootRequest` | Lifecycle |
| 12 | `ShutdownRequest` | Lifecycle |
| 13 | `HealthRequest` | Lifecycle |
| 20 | `ToolRequest` (tool_call) | Tools |
| 21 | `ListToolsRequest` | Tools |
| 30 | `StorageReadRequest` | Storage |
| 31 | `StorageWriteRequest` | Storage |
| 32 | `StorageDeleteRequest` | Storage |
| 33 | `StorageListRequest` | Storage |

### Response Types

Response field numbers mirror request field numbers for each operation type.

## Lifecycle Messages

### Registration

A plugin sends its `PluginManifest` to register with the orchestrator. The manifest declares:

- `id` -- Plugin identifier (e.g., `tools.features`, `storage.markdown`)
- `version` -- Semantic version
- `language` -- Implementation language (`go`, `rust`, `swift`, etc.)
- `provides_tools` / `provides_storage` / `provides_transport` / `provides_ai` -- Capabilities offered
- `needs_storage` / `needs_events` / `needs_ai` / `needs_tools` -- Dependencies on other plugins

### Boot

After registration, the orchestrator sends a `BootRequest` with a `config` map of key-value pairs. The plugin initializes its resources and responds with `BootResult.ready = true`.

### Health

Simple liveness check. The response includes a status enum: `HEALTHY`, `DEGRADED`, or `UNHEALTHY`.

## Tool Messages

- `ToolRequest` -- Invoke a named tool with `google.protobuf.Struct` arguments.
- `ToolResponse` -- Returns `success`, a `result` Struct, or `error_code` + `error_message`.
- `ListToolsRequest/Response` -- Enumerate available tools with name, description, and JSON Schema (`input_schema`).

## Storage Messages

All storage operations include a `storage_type` field (defaults to `"markdown"`) so the orchestrator can route to the correct storage plugin.

- **Read** -- Path-based, returns `content` (bytes), `metadata` (Struct), and `version`.
- **Write** -- Supports optimistic concurrency: `expected_version = 0` means "create new", `> 0` means "update only if version matches".
- **Delete** -- Removes by path.
- **List** -- Enumerate entries by `prefix` and glob `pattern`. Returns path, size, version, and modification timestamp.

## File Structure

```
proto/
  orchestra/plugin/v1/
    plugin.proto          # All message definitions
```

The `.proto` file imports `google/protobuf/struct.proto` and `google/protobuf/timestamp.proto` from the well-known types.
