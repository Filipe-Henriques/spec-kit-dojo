---
description: "Run Sybilion linters and Dojo tests to validate implementation"
---
# Validate Implementation

You are the Sybilion CI Guard. Your task is to validate the code that was just written.

## Pre-flight Checks

Before running any tool, verify the following:

1. **`tests/blackbox/dojo.yaml` exists.** If it is missing, the Dojo run will fail with `Suite 'blackbox' not found in workspace 'tests'`. If missing, read the project's `quickstart.md` to determine the correct `sut_command` and `sut_base_url`, then create the file following the template in the `contract` command.

2. **No `Expect -> sqlite` lines exist in any `.plan` file.** SQLite is an embedded database — Dojo cannot proxy it. If found, remove those lines and replace with HTTP-based assertions (a follow-up `GET` to confirm the resource was persisted).

## Instructions

1. Run the Python formatter: `ruff format src/`
2. Run the Python linter: `ruff check src/`
3. If the linter fails, fix the errors immediately. Do not suppress rules with `# noqa` unless explicitly justified.
4. Run the Dojo tests: `dojo ./tests/blackbox/`
   - The correct invocation is `dojo <suite_directory>` — the suite directory is `tests/blackbox/` (where `dojo.yaml` lives).
   - To see verbose output and debug failures: `dojo --verbose ./tests/blackbox/`
   - To get machine-readable output: `dojo --format json ./tests/blackbox/`
5. If Dojo fails, diagnose by the error type:
   - **`Suite 'blackbox' not found`** → `tests/blackbox/dojo.yaml` is missing. Create it.
   - **`API '<name>' is not defined`** → a `.plan` references an API not declared in `dojo.yaml`. Either add the API to `dojo.yaml` or remove the invalid `Expect` line (e.g. `Expect -> sqlite` is always invalid).
   - **`Expect` timed out** → the SUT is not making the expected outbound call. Find the missing call in the code and add it.
   - **`Expect` matched unexpected request** → the payload on the wire does not match the fixture. Fix the fixture or the SUT code to align.
   - **SUT failed to start** → check `sut_command` in `dojo.yaml`. Ensure the virtual environment is activated and the port is free.
6. Repeat from step 2 until both `ruff check` and `dojo` pass with zero failures.
