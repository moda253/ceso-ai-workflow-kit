# Dart/Flutter Conventions (boat/)

- File naming: `snake_case.dart`; screens at `screens/{feature}/screen.dart`
- State management: Flutter BLoC pattern — `Cubit` for simple state, `Bloc` for event-driven state machines
- Global state: `States` class with static bloc/cubit instances (e.g., `States.auth`, `States.boat`)
- Global services: `Services` class as service locator with static client fields
- Sealed classes for type-safe event/state hierarchies (e.g., `sealed class AuthEvent`)
- Service pattern: singleton via factory constructor + `_internal()` named constructor
- Import ordering: (1) `dart:` stdlib, (2) `package:flutter/` and third-party packages, (3) internal `package:boat/` imports
- Naming: `PascalCase` classes, `camelCase` methods/variables, underscore prefix for private members
- Prefer `final` for immutable references, `const` for compile-time constants and widget constructors
- Explicit types preferred over `var`; use `late` for deferred initialization
- Null safety: `!` assertion after checked conditions, `??` for defaults, `?.` for safe access
- Error handling: catch specific types first (e.g., `GrpcError`), then generic `catch (e, st)`; always capture stack trace
- Error reporting: NewRelic `recordError()` and `recordCustomEvent()`
- Routing: `go_router` package with centralized route definitions in `main.dart`
- Localization: `AppLocalizations.of(context)!` via `intl` package
- Debug logging: `debugPrint('[ClassName] message')` with class prefix
- gRPC interceptors: `AuthInterceptor` for token refresh, `NewRelicInterceptor` for error tracking
- Lint config: `flutter_lints` package with default Flutter recommendations
- Before pushing: run `~/fvm/default/bin/dart format .` from `boat/` — same check CI runs

# PR Review Checklist (Dart / Boat)

- [ ] `dart format .` run from `boat/` — no formatting changes outstanding
- [ ] Flutter composition tools used (SliverMainAxisGroup, SliverToBoxAdapter) — not hacking list indices
- [ ] Dialog builder context renamed to `dialogContext` to avoid stale context after pop
- [ ] All needed data built in a single pass — not filtering or looping twice
- [ ] Shared logic extracted into a helper, not duplicated
- [ ] Related fields grouped into nested proto messages, not separate primitives
- [ ] New type fields use enums, not booleans
- [ ] Codebase alignment: scan changed widgets/helpers against 2-3 similar existing screens — naming, error display style, and widget structure should feel native to the codebase
- [ ] Inline validation errors use `Padding` + `Text` with `color: Colors.red, fontSize: 12` — matches existing pattern
- [ ] FutureBuilder structure follows waiting → error → data order, consistent with existing screens
- [ ] Dialog-local UI state uses `setState` directly; only introduce a new cubit if state is shared across widgets
