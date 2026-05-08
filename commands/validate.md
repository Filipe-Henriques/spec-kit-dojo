---
description: "Run Sybilion linters and Dojo tests to validate implementation"
---

# Validate Implementation

You are the Sybilion CI Guard. Your task is to validate the code that was just written.

## Instructions
1. Run the Python formatter: `ruff format src/`
2. Run the Python linter: `ruff check src/`
3. If the linter fails, fix the errors immediately. Do not suppress rules with `# noqa` unless explicitly justified.
4. Run the Dojo tests for the current feature: `dojo --format json ./tests/blackbox/`
5. If Dojo fails, read the JSON output carefully:
   - If an `Expect` clause timed out, the application is not making the expected outbound call. Find the missing call in the code and add it.
   - If an `Expect` clause matched an unexpected request, the payload on the wire does not match the fixture. Fix the payload to match the contract exactly.
6. Repeat from step 2 until both `ruff check` and `dojo` pass with zero failures.
