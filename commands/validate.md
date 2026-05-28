---
description: "Run Sybilion linters and Dojo tests to validate implementation"
---
# Validate Implementation

You are the Sybilion CI Guard. This command runs **after `/speckit.implement`**. Your task is to lint the code that was just written and run the Dojo contract suite generated earlier by `/speckit.sybilion-dojo.contract`, fixing issues until both pass green.

Sources of truth: `tests/blackbox/` (the contract suite — do not weaken it to make failures go away) and the project's source tree (the implementation — fix it to satisfy the contract).

## Pre-flight Checks

Verify these artifacts before running any tool. Apply mechanical fixes immediately; deeper issues are handled in the Failure Modes table further down.

1. **`tests/blackbox/dojo.yaml` exists.** If missing, Dojo errors with `Suite 'blackbox' not found in workspace 'tests'`. Read `quickstart.md` to determine `sut_command` and `sut_base_url`, then re-create the file using the template in `/speckit.sybilion-dojo.contract` §2.

2. **`tests/blackbox/COVERAGE.md` exists.** If missing, the contract step was incomplete. Flag this and ask the user whether to re-run `/speckit.sybilion-dojo.contract` before validating, or proceed without the audit artifact.

3. **No non-Postgres database in `apis:`.** Dojo can only intercept PostgreSQL via the Postgres wire protocol. Any `apis:` entry for SQLite, MySQL, MongoDB, Redis, DynamoDB, or similar is invalid — Dojo will fail with `API '<name>' is not defined`. Remove the entry from `dojo.yaml` and any `Expect -> <db>` lines from `.plan` files; replace DB-level assertions with HTTP-level follow-up reads (e.g. `POST` then `GET` to confirm persistence).

4. **Every API in `apis:` declares `mode`.** Each entry must specify `mode: mock` or `mode: live`. A missing `mode` is a configuration error.

5. **Every `mode: live` API has real credentials in `.env.local`.** For each live-mode API, confirm `tests/blackbox/.env.local` contains the credentials its definition references (e.g. `$GEMINI_API_KEY`). Never put secrets in `.env`. If `.env.local` only has placeholder values, either populate it or switch the API to `mode: mock` for this run.

6. **Every test using `Evaluate Response` has an `eval.md`.** For each `.plan` containing `Evaluate Response`, confirm that either `tests/blackbox/eval.md` (suite-wide rules) or the test's own directory contains an `eval.md`. Without it, Dojo cannot grade the SUT's outbound payload.

## Instructions

This sequence MUST match the **Execution Loop** in the project constitution. Both the contract layer (Dojo blackbox) and the implementation layer (unit tests, type checks) MUST be green before the task is considered complete.

1. Run the project's configured formatter (as declared in the manifest — `pyproject.toml`, `package.json`, `go.mod`, `Cargo.toml`, etc.).
2. Run the project's configured linter. Fix every reported issue immediately. Do not suppress rules without an explicit, justified inline comment.
3. Run the project's configured type checker, if the language supports one. Fix every reported issue immediately. Do not silence type errors with broad `Any` / `# type: ignore` / `as any` without an inline justification per the constitution.
4. Run the project's unit test suite. Fix every failing test by fixing the root cause in the implementation; never weaken a test to make it pass.
5. Run the Dojo blackbox suite: `dojo ./tests/blackbox/`
   - The suite directory is `tests/blackbox/` (where `dojo.yaml` lives).
   - For verbose request/response tracing: `dojo --trace ./tests/blackbox/`
   - For machine-readable output: `dojo --format json ./tests/blackbox/`
6. If any step fails, diagnose via the **Failure Modes** table below.
7. Repeat from step 1 until formatter, linter, type checker, unit tests, and `dojo` all pass with zero failures.
8. A step that genuinely cannot be run (no type checker available for the language; no unit test suite configured yet) MUST be reported explicitly to the user, not silently skipped.

**Do not weaken the contract suite or the unit tests to make failures disappear.** If a test seems wrong, verify against `spec.md` first — the implementation is usually the side that needs to change. If a test is genuinely incorrect (fixture does not match the spec, unit test asserts against a stale interface), fix the test and note the divergence to the user.

## Failure Modes

| Symptom | Likely cause | Fix |
|---|---|---|
| Type checker errors | Missing or wrong type annotations, or implementation type mismatch | Fix the implementation or narrow the annotation. Do not silence with broad `Any` / `# type: ignore` / `as any` without an inline justification per the constitution. |
| Unit test failures | Implementation breaks an invariant the unit test asserts | Fix the implementation. If the test itself is wrong, verify against `spec.md` before changing it — the implementation is usually the side that needs to change. |
| Required step has no tool configured (no type checker, no unit test runner) | The project's manifest does not declare the tool the constitution requires | Report explicitly which step is being skipped and why. Open a follow-up task to add the missing tool, or document the absence in **Project-Specific Rules** of the constitution. Never silently treat a missing tool as a passing step. |
| `Suite 'blackbox' not found in workspace 'tests'` | `tests/blackbox/dojo.yaml` missing | Create per Pre-flight #1. |
| `API '<name>' is not defined` | A `.plan` references an API not declared in `apis:`, or references a non-Postgres database | Add the API to `dojo.yaml` (with `mode`) or remove the offending `Expect` line. If the reference is to SQLite/MySQL/Mongo/Redis/etc., the `Expect` is fundamentally invalid — replace with HTTP follow-up checks. |
| `Expect` timed out — for a call that happens at runtime | SUT is not making the expected outbound call | Find the missing call in the SUT and add it. Verify the SUT reads the upstream URL from the `API_<NAME>_URL` env var Dojo injects, not from a hard-coded URL. |
| `Expect` timed out — for a call that happens at SUT startup | A boot-time outbound call is not captured | Move the `Expect` line from the test plan into `tests/blackbox/startup.plan`. Startup expectations are fulfilled once before any test runs. |
| `Expect` matched unexpected request | Payload on the wire ≠ fixture | Compare the trace output (`--trace`) with the fixture. Fix whichever side is wrong per `spec.md` — usually the SUT. Update the fixture only if it materially diverges from the spec. |
| `Evaluate Response` graded as failed | The LLM grader judged the SUT's outbound payload non-conforming | Read the grader's reason. If the SUT payload is wrong, fix the SUT. If `eval.md` is too strict (failing on legitimate variation), loosen the rule. |
| Live-mode API: connection refused / 401 / 403 | `.env.local` missing real credentials, or the API URL is wrong | Populate `.env.local` (never `.env`). If you can't run the real upstream locally, temporarily switch the API to `mode: mock` and note this to the user. |
| SUT failed to start | `sut_command` problem, port conflict, runtime environment inactive, or missing env var | Inspect `sut_command` in `dojo.yaml`. Make sure the runtime environment is active (venv, nvm, container, etc.), the port is free, and any required env vars are set. |
| Per-test seed failed | A `test_*/seed/*.sql` file could not execute | A failed seed aborts the whole suite (even other tests are marked failed). Fix the SQL, remove the seed if obsolete, or check the Postgres connection. |
| Test count obviously low for the spec (e.g. one test per endpoint) | The contract step under-generated | Read `tests/blackbox/COVERAGE.md` and the spec folder (`specs/$(git rev-parse --abbrev-ref HEAD)/` — including `data-model.md` and any `contracts/` files). If `COVERAGE.md` does not map every `FR-*` / `SC-*` or has missed constraints defined in `data-model.md` / `contracts/`, re-run `/speckit.sybilion-dojo.contract` rather than proceeding with a thin suite. |
| Symptom not listed above | Unknown failure mode | If the Superpowers `systematic-debugging` skill is present in this session, defer to it. Otherwise apply the 4-phase root-cause process: reproduce → isolate → identify root cause → fix. Do NOT patch symptoms before the cause is named. |
