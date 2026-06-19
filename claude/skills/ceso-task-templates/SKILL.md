---
name: ceso-task-templates
description: Task intake skill for the CESO Labs monorepo. User-invoked when starting a new Asana task — handles branch setup and structured template intake (Buoy endpoint, Boat screen, Artoo integration, tests, bug fix). Do NOT invoke proactively during normal conversation; only run when the user explicitly calls it.
---

## Purpose

This skill is for starting new work, not for every interaction. The user invokes it manually when picking up an Asana task. Normal conversation and ongoing work don't need it.

---

## How to use this skill

When invoked, follow this flow exactly:

### Step 1: Branch setup

Ask: "Are you starting on a new branch, or already on your working branch?"

**Already on a branch:** Skip — move straight to Step 2.

**New branch:** Walk through the following in order:

1. **Base branch** — Ask: "Are you branching from `develop`, or a different branch?" (Default is `develop`; occasionally work branches off another feature branch.)

2. **Pull latest** — Ask: "Do you want to pull the latest on `[base branch]` before branching?" If yes, run:
   ```
   git checkout [base branch]
   git pull
   ```

3. **Branch name** — Based on the task description, suggest a name following the convention `Component-Short-Description` in PascalCase with dashes (e.g., `Boat-Vehicle-Conflict-Exit-Screen`, `Buoy-Add-Driver-Status-Endpoint`). Confirm with the user.

4. **Create the branch** — run:
   ```
   git checkout -b [branch-name] [base branch]
   ```
   Confirm the branch was created successfully before moving on.

### Step 2: Template matching

Ask the user to describe the task (or paste the Asana task description).

Identify which template(s) apply using the matching rules below.

**If a match is found:**
- Tell the user which template(s) matched and why (one sentence each)
- Ask the clarifying questions for each matched template — consolidate into a single message, not one question at a time
- Fill in the template placeholders with the answers
- Ask one broad downstream impact question: "Does this change affect other users, other parts of the app, or external systems (Artoo, Compass, route state)?" — use the answer to surface any cross-cutting concerns before work begins, not after
- Present the completed brief and ask: "Anything else to add before I start?"
- Begin work once confirmed

**If no template matches:**
- Say: "I don't have a template that fits this task yet. Want to work through it now and add one afterward?"
- If yes: proceed with the work, then follow the "Building a new template" flow below once complete
- If no: proceed with the work without a template

If a task spans multiple templates (e.g., a new feature needs both a Buoy endpoint AND a Boat screen), run the intake for both and present them together.

---

## Building a new template (when no match is found)

After completing work on a task that didn't have a template:

1. **Review what was just built** — look at the files changed and the patterns used
2. **Draft a new template entry** modeled on the existing ones:
   - A short heading: `### Template [N]: [Name]`
   - A `Use when:` line describing the task type
   - A `Reference:` line pointing to the real code example just written
   - The clarifying questions a user would need to answer
   - The filled template structure with placeholders
3. **Ask the user:** "Here's a draft template based on what we just built — want me to add it to the skill file?"
4. **If yes:** append the new template to `/Users/mike.olson/.claude/skills/ceso-task-templates.md` and update the Template Matching Rules table

---

## Call Chain Reference

```
Boat (Flutter gRPC client)
  → Buoy (Go handler, Connect RPC)
    → Artoo (external SIS, HTTP REST)
```

Templates are organized along this chain. A single task often spans 2–3 templates.
Compass (Next.js) is a separate app — Compass templates will be added to this skill when needed.

---

## Template Matching Rules

| If the task involves... | Use template(s) |
|---|---|
| new endpoint, new API, Buoy handler, backend logic | 1a or 1b (+ 1c if Artoo is involved) |
| reads data, no DB writes, lookup, check, validate | 1a |
| saves data, creates, updates, deletes, completes, starts | 1b |
| Artoo, SIS, external data, sync, student info | 1c (+ 1a or 1b) |
| new screen, display, show, Flutter UI, Boat screen | 2a |
| button, action, tap, trigger, submit, confirm | 2b |
| test, unit test, write tests | 3 |
| bug, broken, not working, fix, error, crash | 4 |

When unsure between 1a and 1b, ask: "Does this write to the database?"

---

## Clarifying Questions Per Template

### Template 1a/1b (Buoy endpoint)
Ask all of these at once:
- What service does this go in? (tracking, students, auth, artoo — or a new one?)
- What should the method be named? (e.g., `CheckDevice`, `GetRoutes`)
- What fields does the **request** need? (name and type for each)
- What fields does the **response** need? (name and type, or empty `{}` if success is implicit)
- Does this **write** to the database? (→ 1a if no, 1b if yes)
- If 1b: how many separate DB tables are written to? (determines if transaction is needed)
- Does this need to **call Artoo**? (→ also use Template 1c)

### Template 1c (Artoo integration)
Ask all of these at once:
- Which Artoo method is needed? (see list below)
- Is this a **read** (map Artoo response to gRPC), an **async write** (fire-and-forget goroutine after commit), or a **sync write** (block and map Artoo errors)?
- What Artoo fields need to map to gRPC response fields?

Available Artoo methods:
  GetRoutes, GetRoute, GetRouteWithStops, GetDriver,
  GetParentStudents, GetRoutesForParent, GetStopDataForRoute,
  GetRoutesForStudent, GetAllRouteStops, GetVehiclesForVendor,
  GetVehicleModelList, PostOneTimeNoRide, StartRoute,
  CompleteRoute, UpdateRouteStatuses

### Template 2a (Flutter display screen)
Ask all of these at once:
- What is the screen called, and where will it live? (`lib/screens/[feature]/screen.dart`)
- What gRPC method does it call to load data? (which `Services.tracking!.X()` call?)
- What are the different **states** the screen needs to show? (loading, data, error, edge cases?)
- Does it navigate anywhere on user action? (what route, what triggers it?)

### Template 2b (Flutter mutation)
Ask all of these at once:
- What does the button/action do in plain English?
- Which **Buoy endpoint** does it call? (method name)
- Does it need a **confirmation dialog** before making the call?
- Does it need to pass **GPS coordinates** (`Point`) to the endpoint?
- What happens on **success**? (navigate, refresh state, show message?)
- What happens on **error**? (retry, show dialog, navigate to exit?)

### Template 3 (Unit tests)
Ask all of these at once:
- Which handler are we testing?
- What are the **test cases**? (success path + each failure/edge case — list them)
- Does the handler call any Artoo methods? (so mock setup is correct)

### Template 4 (Bug fix)
Ask all of these at once:
- Which file and function is broken? (exact path)
- What is the **symptom**? (what the user sees, what error/log appears)
- What have you **already ruled out**?

---

## Template Content

### Template 1a: Buoy Read Endpoint

```
You are working in the buoy monorepo Go backend (buoy/app/services/[SERVICE]/main.go).
Framework: Connect RPC. Auth is handled by interceptors — do NOT add auth logic here.
DB access: sqlc-generated queries in buoy/gen/queries/.

## Task
Add a new gRPC read endpoint: [METHOD_NAME]
It should: [ONE_SENTENCE_DESCRIPTION]

## Proto changes (proto/[SERVICE]/[SERVICE].proto)
Add to service block:
  rpc [METHOD_NAME]([METHOD_NAME]Request) returns ([METHOD_NAME]Response);

Messages:
  message [METHOD_NAME]Request {
    [REQUEST_FIELDS]
  }
  message [METHOD_NAME]Response {
    [RESPONSE_FIELDS]
  }

## Handler pattern (follow CheckDevice in tracking/main.go:281)
  func (s *server) [METHOD_NAME](
      ctx context.Context,
      req *connect.Request[service.[METHOD_NAME]Request],
  ) (*connect.Response[service.[METHOD_NAME]Response], error) {
      logging.FromContext(ctx).Debug("service.[METHOD_NAME]")

      row, err := s.queries.[QUERY_NAME](ctx, queries.[QUERY_NAME]Params{
          [PARAM_MAPPING]
      })
      if err == pgx.ErrNoRows {
          return nil, connect.NewError(connect.CodeNotFound, err)
      }
      if err != nil {
          return nil, err
      }

      res := connect.NewResponse(&service.[METHOD_NAME]Response{
          [RESPONSE_MAPPING]
      })
      return res, nil
  }

## Rules
- No transaction — read-only handlers skip Begin/Rollback/WithTx
- Check pgx.ErrNoRows before generic err != nil
- If a table is already queried here, add needed columns to that query — no second roundtrip
- No TODO comments, swagger comments, or placeholder logic
- Import order: (1) stdlib (2) third-party (3) internal buoy/ — blank line between groups

## After writing
  make gen && go test ./app/services/[SERVICE]/... -v
```

---

### Template 1b: Buoy Write Endpoint (transaction)

```
You are working in the buoy monorepo Go backend (buoy/app/services/[SERVICE]/main.go).
Framework: Connect RPC. Auth is handled by interceptors — do NOT add auth logic here.
DB access: sqlc-generated queries in buoy/gen/queries/.

## Task
Add a new gRPC write endpoint: [METHOD_NAME]
It should: [ONE_SENTENCE_DESCRIPTION]

## Proto changes (proto/[SERVICE]/[SERVICE].proto)
Add to service block:
  rpc [METHOD_NAME]([METHOD_NAME]Request) returns ([METHOD_NAME]Response);

Messages:
  message [METHOD_NAME]Request {
    [REQUEST_FIELDS]
  }
  message [METHOD_NAME]Response {}

## Handler pattern (follow CompleteRoute in tracking/main.go:1917)
  func (s *server) [METHOD_NAME](
      ctx context.Context,
      req *connect.Request[service.[METHOD_NAME]Request],
  ) (*connect.Response[service.[METHOD_NAME]Response], error) {
      logging.FromContext(ctx).Debug("service.[METHOD_NAME]")

      tx, err := config.Clients.Db.Begin(ctx)
      if err != nil {
          return nil, err
      }
      defer tx.Rollback(ctx)
      qtx := config.Clients.Queries.WithTx(tx)

      if err := qtx.[WRITE_QUERY](ctx, queries.[WRITE_QUERY]Params{
          [PARAM_MAPPING]
      }); err != nil {
          return nil, err
      }

      if err := tx.Commit(ctx); err != nil {
          return nil, err
      }

      // Side effects (Artoo calls, notifications) go AFTER commit

      res := connect.NewResponse(&service.[METHOD_NAME]Response{})
      return res, nil
  }

## Transaction rules
- Begin → defer Rollback → WithTx → all DB ops use qtx → explicit Commit
- defer Rollback is safe after Commit (pgx ignores it)
- Side effects after Commit only — they don't roll back with the DB

## After writing
  make gen && go test ./app/services/[SERVICE]/... -v
```

---

### Template 1c: Artoo Integration

```
You are working in buoy/app/services/[SERVICE]/main.go.
The Artoo service is injected as s.artoo (type ArtooServiceI, defined in artoo/main.go:167).

## Task
[DESCRIPTION_OF_WHAT_ARTOO_IS_NEEDED_FOR]

## Pattern A — Read from Artoo, map to gRPC (use for: GetRoutes, GetRoute, GetDriver, etc.)
  result, err := s.artoo.[ARTOO_METHOD](ctx, [ARGS])
  if err != nil {
      return nil, err
  }
  // Map Artoo fields to gRPC fields explicitly:
  [FIELD_MAPPING]

## Pattern B — Async write to Artoo after DB commit (use for: StartRoute, CompleteRoute)
  if err := tx.Commit(ctx); err != nil {
      return nil, err
  }
  go s.artoo.[ARTOO_METHOD](context.WithoutCancel(ctx), [ARGS])

## Pattern C — Sync write with error mapping (use for: PostOneTimeNoRide, etc.)
  err = s.artoo.[ARTOO_METHOD](ctx, artoo.[RequestType]{[FIELDS]})
  if err != nil {
      if _, ok := err.(*artoo.[DomainErrorType]); ok {
          return nil, connect.NewError(connect.[CONNECT_CODE], err)
      }
      return nil, err
  }

## Artoo domain error → Connect code mapping
  DriverMisconfiguredError  → CodeInvalidArgument or CodeFailedPrecondition
  DriverDoesNotExistError   → CodeNotFound
  DuplicateSubmissionError  → CodeAlreadyExists
  InvalidPermissionError    → CodePermissionDenied

## Pattern to use: [A / B / C]
## Artoo method: [METHOD_NAME]
## Field mapping: [ARTOO_FIELD → GRPC_FIELD]
```

---

### Template 2a: Flutter Display Screen

```
You are working in the Boat Flutter app (boat/lib/).
State: States class (BLoC). Services: Services class (gRPC clients).
Routing: go_router via lib/main.dart — always use GoRouter, never Navigator.
Localization: AppLocalizations.of(context)! — no hardcoded strings.

## Task
Create lib/screens/[FEATURE]/screen.dart that [DESCRIPTION].

## Pattern (follow ExitScreen in screens/exit/screen.dart)

class [FEATURE]ScreenData {
  final [TYPE]? [FIELD];
  const [FEATURE]ScreenData({this.[FIELD]});
}

class [FEATURE]Screen extends StatefulWidget {
  const [FEATURE]Screen({super.key, required this.data});
  final [FEATURE]ScreenData data;
  @override
  State<[FEATURE]Screen> createState() => _[FEATURE]ScreenState();
}

class _[FEATURE]ScreenState extends State<[FEATURE]Screen> {
  late ResponseFuture<[GRPC_RESPONSE_TYPE]> _[DATA_FIELD];

  @override
  void initState() {
    super.initState();
    _[DATA_FIELD] = Services.tracking!.[GRPC_METHOD]([GRPC_REQUEST_TYPE]());
  }

  @override
  Widget build(BuildContext context) {
    var l10n = AppLocalizations.of(context)!;
    return Scaffold(
      appBar: AppBar(centerTitle: false, title: Text(l10n.[TITLE_KEY])),
      body: FutureBuilder<[GRPC_RESPONSE_TYPE]>(
        future: _[DATA_FIELD],
        builder: (BuildContext context, AsyncSnapshot<[GRPC_RESPONSE_TYPE]> snapshot) {
          [CONDITION_CHECKS]  // special states first
          if (snapshot.hasData) { [DATA_UI] }
          return Center(child: CircularProgressIndicator());
        },
      ),
    );
  }
}

## Navigation: GoRouter.of(navigatorKey.currentContext!).go('/[ROUTE]')
## New strings: add to boat/lib/l10n/app_en.arb, then flutter gen-l10n

## After writing: dart format . && flutter analyze
```

---

### Template 2b: Flutter Mutation

```
You are working in the Boat Flutter app (boat/lib/).

## Task
Add a button/action that calls [BUOY_METHOD] on Buoy.
[DESCRIPTION_OF_WHAT_IT_DOES]

## State guard (add to State class)
bool _is[ACTION]ing = false;

## Pattern (follow _completeRoute in screens/boat/screen.dart:422)

Future<void> _[ACTION]() async {
  if (_is[ACTION]ing) return;
  final l10n = AppLocalizations.of(context)!;

  [CONFIRMATION_DIALOG_IF_NEEDED]

  if (_is[ACTION]ing) return;  // re-check after async gap
  setState(() => _is[ACTION]ing = true);
  LoadingOverlay.show(context, l10n.loading);

  try {
    await Services.tracking!.[BUOY_METHOD](
      [METHOD_NAME]Request(
        [REQUEST_FIELDS]
        [POINT_IF_GPS_NEEDED]
      ),
    );
    NewrelicMobile.instance.recordCustomEvent("Custom", eventName: "[ACTION_NAME]");
    [SUCCESS_ACTION]  // navigate or States.boat.check()
  } catch (e, st) {
    if (mounted) {
      setState(() => _is[ACTION]ing = false);
      if (LoadingOverlay.isShowing) LoadingOverlay.hide();
      await handleError(context, error: e, stack: st, close: true, endpoint: "Tracking/[METHOD_NAME]");
    }
  } finally {
    if (mounted) {
      setState(() => _is[ACTION]ing = false);
      if (LoadingOverlay.isShowing) LoadingOverlay.hide();
    }
  }
}

OutlinedButton.icon(
  onPressed: _is[ACTION]ing ? null : _[ACTION],
  icon: const Icon(Icons.[ICON]),
  label: Text(l10n.[BUTTON_LABEL]),
)

## Rules
- dialogContext (not context) in dialog builders
- GPS timestamps: use _positionTimestamp, not DateTime.now()
- Handle status enum cases before the success path if response has multiple outcomes

## After writing: dart format . && flutter analyze
```

---

### Template 3: Unit Tests (Go)

```
You are writing tests in buoy/app/services/[SERVICE]/main_test.go (package [SERVICE]).
Table-driven, gomock, MockQuerier from buoy/gen/queries/.

## Handler to test: [HANDLER_NAME]

## Test cases:
[TEST_CASES]

## Pattern (follow Test_server_CheckDevice in tracking/main_test.go:30)

func Test_server_[HANDLER_NAME](t *testing.T) {
    type args struct {
        ctx context.Context
        req *connect.Request[service.[HANDLER_NAME]Request]
    }
    tests := []struct {
        name      string
        args      args
        mockSetup func(t *testing.T, m *queries.MockQuerier)
        wantErr   bool
        want      *connect.Response[service.[HANDLER_NAME]Response]
        wantCode  connect.Code
    }{
        // [TEST_CASES — one struct per scenario]
        // Include DoAndReturn with param validation, .Times(1) on all expectations
        // wantErr + wantCode for error cases
    }
    for _, tt := range tests {
        tt := tt
        t.Run(tt.name, func(t *testing.T) {
            ctrl := gomock.NewController(t)
            defer ctrl.Finish()
            mockArtoo := artoo.NewMockArtooServiceI(ctrl)
            mockQueries := queries.NewMockQuerier(ctrl)
            if tt.mockSetup != nil {
                tt.mockSetup(t, mockQueries)
            }
            s := &server{artoo: mockArtoo, queries: mockQueries, cache: nil}
            got, err := s.[HANDLER_NAME](tt.args.ctx, tt.args.req)
            if tt.wantErr {
                require.Error(t, err)
                require.Nil(t, got)
                require.Equal(t, tt.wantCode, connect.CodeOf(err))
            } else {
                require.NoError(t, err)
                require.NotNil(t, got)
            }
        })
    }
}

## Run: go test ./app/services/[SERVICE]/... -run Test_server_[HANDLER_NAME] -v
```

---

### Template 4: Bug Fix

```
You are fixing a bug in the buoy monorepo.
Component: [COMPONENT — Buoy (Go) / Boat (Flutter/Dart)]
File: [FILE_PATH]

## Symptom
[SYMPTOM — what the user sees, what error appears, what log shows]

## Relevant code
[PASTE_RELEVANT_FUNCTION]

## Already ruled out
[RULED_OUT]

## Rules
- Only modify code inside the described function
- No whitespace, formatting, or line ending changes
- No TODO comments, no added blank lines

## After fixing
[Buoy] go test ./app/services/[PACKAGE]/... -v
[Boat] dart format . && flutter analyze
```
