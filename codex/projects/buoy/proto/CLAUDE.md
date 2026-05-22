<!-- GENERATED FROM claude/projects/buoy/proto/CLAUDE.md. Do not edit directly. -->

# Protobuf Conventions (proto/)

- Syntax: `proto3`; one package per file matching directory name
- Field names: `snake_case`; message/service/enum names: `PascalCase`
- RPC methods: `Request`/`Response` suffix pairs (e.g., `SignInRequest`, `SignInResponse`)
- Enums: always include `UNSPECIFIED = 0` value; values use `UPPER_SNAKE_CASE` with enum name prefix
- Use `optional` for nullable fields, `oneof` for mutually exclusive choices, `repeated` for collections
- Go package option format: `option go_package = "buoy/gen/rpc/{package};{package}";`
- Code generation: separate `buf.gen.yaml` per project; plugin versions pinned for compatibility
- Buf linting enabled; backwards compatibility enforced with `FILE` strategy
