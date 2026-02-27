# Contributing to proto

## Prerequisites

- [buf CLI](https://buf.build/docs/installation) v1.28+
- Familiarity with Protocol Buffers 3 syntax

## Development Setup

```bash
git clone https://github.com/orchestra-mcp/proto.git
cd proto
```

## Making Changes

1. Edit `orchestra/plugin/v1/plugin.proto`.
2. Validate the schema:
   ```bash
   buf lint
   ```
3. Check for breaking changes against the previous version:
   ```bash
   buf breaking --against '.git#branch=main'
   ```
4. Generate code for all target languages:
   ```bash
   buf generate
   ```

## Schema Conventions

- Use field numbers 10-19 for lifecycle, 20-29 for tools, 30-39 for storage.
- All new request types must have a corresponding response type.
- Use `google.protobuf.Struct` for dynamic key-value data (tool arguments, metadata).
- Use `google.protobuf.Timestamp` for time fields.
- Add comments above every message and field.

## Code Style

- Follow the [Protobuf Style Guide](https://protobuf.dev/programming-guides/style/).
- Use `snake_case` for field names.
- Use `PascalCase` for message and enum names.
- Enum values must be prefixed with the enum name in `SCREAMING_SNAKE_CASE` (e.g., `STATUS_HEALTHY`).

## Pull Request Process

1. Fork the repository and create a feature branch.
2. Make your changes and run `buf lint` and `buf breaking`.
3. Describe the motivation for schema changes in the PR description.
4. Breaking changes require a major version bump and must be discussed in an issue first.

## Related Repositories

- [orchestra-mcp/gen-go](https://github.com/orchestra-mcp/gen-go) -- Generated Go code
- [orchestra-mcp/sdk-go](https://github.com/orchestra-mcp/sdk-go) -- Go Plugin SDK
- [orchestra-mcp](https://github.com/orchestra-mcp) -- Organization home
