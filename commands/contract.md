---
description: "Generate Dojo blackbox contracts based on the implementation plan"
---

# Generate Dojo Contract

You are an expert QA Engineer at Sybilion. Your task is to read the `plan.md` and generate the exact Dojo blackbox tests that will validate the implementation.

## Dojo Rules & Constraints

You MUST follow these strict rules when generating Dojo tests:

### 1. Directory Structure
- All tests MUST be placed inside `tests/blackbox/`.
- Each test MUST have its own directory starting with `test_` (e.g., `tests/blackbox/test_user_register/`).
- Each test directory MUST contain exactly one `.plan` file (e.g., `user_register.plan`).
- Fixture files (`.json` or `.sql`) MUST be placed in the same directory as the `.plan` file.

### 2. The `.plan` DSL Syntax
The `.plan` file uses a strict `Action -> Target -> Clause` syntax.
- Every plan MUST start with a `Perform` action to trigger the Software Under Test (SUT).
- Every expected side-effect MUST be an `Expect` action.

**Valid Perform Syntax:**
- `Perform -> POST /api/users -> Payload: incoming.json` (HTTP trigger)
- `Perform -> wait -> 500ms` (Pause execution)

**Valid Expect Syntax:**
- `Expect -> postgres -> Request: insert_user.sql` (Database assertion)
- `Expect -> stripe_api -> Request: charge.json -> Respond: charge_success.json` (API assertion with mock response)
- `Expect -> gemini -> Request: prompt.json -> MaxCalls: "3"` (Variable repeat expectations)

### 3. Fixtures (JSON and SQL)
- You MUST generate the exact fixture files referenced in the `.plan`.
- HTTP request/response bodies MUST be `.json` files.
- Database queries MUST be `.sql` files.
- Fixtures represent the exact payload on the wire. Do not use placeholders unless using Dojo's `$VAR` expansion for environment variables.

## Instructions

1. Read the current specification and plan.
2. Identify all external network boundaries (Database, APIs, Queues).
3. Create the test directory under `tests/blackbox/`.
4. Write the `.plan` file using the strict DSL.
5. Generate all necessary `.json` and `.sql` fixture files referenced in the plan.

**DO NOT write any application code.** Only write the Dojo tests.
