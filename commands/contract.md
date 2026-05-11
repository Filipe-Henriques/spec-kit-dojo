---
description: "Generate Dojo blackbox contracts based on the implementation plan"
---
# Generate Dojo Contract

You are an expert QA Engineer at Sybilion. Your task is to read the `plan.md` and generate the exact Dojo blackbox tests that will validate the implementation.

## Dojo Rules & Constraints

You MUST follow these strict rules when generating Dojo tests:

### 1. Directory Structure

```
tests/blackbox/
  dojo.yaml              ← REQUIRED: SUT command, base URL, timeouts, APIs
  test_<name>/
    <name>.plan          ← exactly one .plan per test directory
    <fixture>.json       ← fixture files referenced in the plan
```

- All tests MUST be placed inside `tests/blackbox/`.
- Each test MUST have its own directory starting with `test_`.
- Each test directory MUST contain exactly one `.plan` file.
- Fixture files (`.json`) MUST be placed in the same directory as the `.plan` file.
- **`tests/blackbox/dojo.yaml` MUST always be created.** Without it, Dojo errors with `Suite 'blackbox' not found in workspace 'tests'`.

### 2. The `dojo.yaml` Suite Configuration

`tests/blackbox/dojo.yaml` is mandatory. It declares:

```yaml
concurrency: 1
sut_command: "<shell command to start the SUT>"
sut_base_url: "http://127.0.0.1:<port>"
timeouts:
  perform: 10s
  expect: 5s
  sut_startup: 30s
```

**How to determine `sut_command`:**
- Read the project's `quickstart.md` or `README.md` to find the exact command used to start the server.
- Use the `.venv` virtual environment if the project uses Python: `.venv/bin/uvicorn <module>:app --host 127.0.0.1 --port <port>`
- Set a unique port (e.g. `18000`) to avoid conflicts with the development server.
- If the app accepts a database path via environment variable (e.g. `TASK_MANAGER_DB`), set it to a temp path: `TASK_MANAGER_DB=/tmp/dojo_test.db .venv/bin/uvicorn ...`

**APIs section** — only add entries for **external network services** the SUT calls outbound (e.g. Stripe, Gemini, a Postgres server). Do NOT add entries for the SUT's own database if it is embedded (SQLite, in-process).

### 3. The `.plan` DSL Syntax

The `.plan` file uses a strict `Action -> Target -> Clause` syntax.

- Every plan MUST start with a `Perform` action to trigger the SUT.
- `Expect` lines declare outbound calls the SUT must make to **external APIs** (proxied by Dojo).

**Valid Perform Syntax:**
```
Perform -> GET /path -> ExpectStatus: "200"
Perform -> POST /path -> Payload: request.json -> ExpectStatus: "201"
Perform -> DELETE /path/1 -> ExpectStatus: "204"
Perform -> wait -> 500ms
Perform -> postgres -> check.sql -> "1"
```

**Valid Expect Syntax (outbound API calls only):**
```
Expect -> postgres -> Request: insert_user.sql
Expect -> stripe_api -> Request: charge.json -> Respond: charge_success.json
Expect -> gemini -> Request: prompt.json -> MaxCalls: "3"
```

**`ExpectStatus` clause:** Always include `ExpectStatus` on every `Perform` line. Without it, Dojo fails on any HTTP status >= 400, which causes false failures on tests that intentionally trigger 4xx responses.

### 4. Database Assertions

**SQLite (embedded, in-process):** Dojo is a network proxy — it cannot intercept SQLite calls because SQLite is not a network service. Do NOT write `Expect -> sqlite -> ...` lines. They are invalid and will cause Dojo to error with `API 'sqlite' is not defined`.

For apps using SQLite, validate persistence indirectly via HTTP:
- After a `POST` that creates a resource, follow with a `GET` to confirm the resource is listed.
- After a `DELETE`, follow with a `GET` to confirm the resource is gone (or returns 404).

Example for a task manager with SQLite:
```
# test_create_task/create_task.plan
Perform -> POST /tasks -> Payload: create_request.json -> ExpectStatus: "201"

# test_list_tasks/list_tasks.plan
Perform -> GET /tasks -> ExpectStatus: "200"

# test_delete_task/delete_task.plan
Perform -> DELETE /tasks/1 -> ExpectStatus: "204"
```

**PostgreSQL (external server):** Dojo supports Postgres via the wire protocol. Add to `dojo.yaml`:
```yaml
apis:
  postgres:
    mode: live
    protocol: postgres
    url: "postgres://user:pass@host:5432/db?sslmode=disable"
```
Then use `Expect -> postgres -> Request: query.sql` or `Perform -> postgres -> check.sql -> "1"` in plans.

### 5. Fixtures

- You MUST generate every fixture file referenced in the `.plan`.
- HTTP request bodies MUST be `.json` files with the exact payload the SUT expects.
- SQL files are only for PostgreSQL assertions (not SQLite).
- Do not use placeholders. Fixtures represent the exact payload on the wire.

## Instructions

1. Read the current specification and plan.
2. Read `quickstart.md` (or equivalent) to find the exact server start command and port.
3. Identify all **external** network boundaries (Postgres, external APIs, queues). SQLite is NOT external.
4. Create `tests/blackbox/dojo.yaml` with the correct `sut_command`, `sut_base_url`, and any external API entries.
5. Create one `test_<name>/` directory per scenario.
6. Write the `.plan` file for each test using the strict DSL.
7. Generate all necessary `.json` fixture files referenced in the plans.

**DO NOT write any application code.** Only write the Dojo tests and `dojo.yaml`.
