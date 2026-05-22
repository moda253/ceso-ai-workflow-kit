# Go Conventions (buoy/)

- Package structure: `cmd/` (CLI entry), `app/` (business logic), `app/services/` (gRPC service implementations), `db/` (migrations + SQL queries), `config/` (global config)
- Each service in `app/services/{name}/main.go`; tests in `main_test.go`
- Constructor pattern: `NewX()` functions (e.g., `NewServer()`, `NewCache()`)
- Interface naming: suffix with `I` (e.g., `ArtooServiceI`, `CacheI`)
- Import ordering: (1) stdlib, (2) third-party, (3) internal `buoy/` — separated by blank lines
- Error handling: custom sentinel error structs, `connect.NewError(code, err)` for gRPC errors, check `pgx.ErrNoRows` explicitly
- Logging: `log/slog` via `logging.FromContext(ctx)` with structured key-value pairs
- Context: always first parameter; custom `FromContext()` extractors per package; context keys as unexported struct types
- Config: environment variables loaded at startup in `config.InitConfig()`; properties file for feature flags via `config.BuoyConfig`
- Database: `pgxpool` connection pool, `sqlc` for query generation, `UNNEST` pattern for bulk inserts
- DB efficiency: if a handler already queries a table, add any additional needed columns to that same query rather than making a second round trip to the same table
- Testing: table-driven tests, `gomock` for auto-generated mocks (`//go:generate mockgen`), manual mock structs for simple cases
- Enum pattern: `iota` constants with `String()` method
- Constants: `camelCase` or `PascalCase` for typed constants; descriptive `CacheId` type aliases
- gRPC: Connect RPC framework; import aliases `service` and `serviceConnect` for generated packages
- Deferred cleanup: always `defer tx.Rollback(ctx)`, `defer file.Close()`, etc.
- Before pushing: run `make check-format` from the repo root and `go test ./app/services/tracking/...` from `buoy/` — these are the same checks CI runs

# PR Review Checklist (Go / Buoy)

- [ ] No second query to a table already queried — add columns to the existing query
- [ ] New type fields use enums, not booleans
- [ ] Related fields grouped into nested proto messages, not separate primitives
- [ ] New API endpoints include server-side validations
- [ ] Shared logic extracted into a helper, not duplicated
- [ ] Helpers written against interfaces (e.g. `queries.Querier`)
- [ ] Branching duplication eliminated — build shared structure once
- [ ] Vendor/external cache TTLs are conservative (10 min)
- [ ] Integration tests verify state after mutations
- [ ] No dead code in this PR — any new RPCs, messages, helpers, or handlers you added are actually called. Don't touch pre-existing unused code (others may be scaffolding intentionally)
- [ ] Unit tests are narrowly focused — one concern per test, not bundled into broad flow tests that touch unrelated logic
- [ ] Codebase alignment: scan changed handlers/helpers against 2-3 similar existing ones — comment verbosity, variable naming, and structure should be indistinguishable from the surrounding code
- [ ] Comment style matches surrounding code — brief phrases for simple guards; avoid multi-line explanations of self-evident logic
- [ ] New read-only handlers skip transactions; new write handlers follow the `Begin → defer Rollback → WithTx` pattern
