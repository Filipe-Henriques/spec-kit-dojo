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

7. **Every new unit test on this branch has captured red-evidence.** This pre-flight enforces the constitution's Testing §TDD discipline ("the agent MUST be able to cite the failing-run output that preceded the green run") as a checkable artifact.

   *Defer-to-Superpowers.* If the Superpowers `test-driven-development` skill is present in this session, that skill enforces red-then-green at write time and produces its own evidence. Skip this pre-flight and note the deferral.

   *Inline check (standalone).* Identify unit test files added on this branch:

   ```bash
   BASE=$(git merge-base HEAD origin/main 2>/dev/null \
        || git merge-base HEAD main 2>/dev/null \
        || git merge-base HEAD master)
   git diff --diff-filter=A --name-only "$BASE"...HEAD
   ```

   From the listed paths, keep only **unit test files** under the project's test runner convention (consult the project manifest):

   | Language / runner | Test file pattern |
   |---|---|
   | Python (pytest, unittest) | `**/test_*.py`, `**/*_test.py` |
   | JS / TS (jest, vitest, mocha) | `**/*.{test,spec}.{js,jsx,ts,tsx}`, `**/__tests__/**/*.{js,jsx,ts,tsx}` |
   | Go | `**/*_test.go` |
   | Rust | `**/tests/**/*.rs` (inline `#[cfg(test)]` blocks are scoped to their host source file) |
   | Java / Kotlin (JUnit, Kotest) | `**/src/test/**/*.{java,kt}`, `**/*{Test,Tests,Spec}.{java,kt}` |
   | Ruby (RSpec, Minitest) | `**/spec/**/*.rb`, `**/test/**/*_test.rb` |
   | Other | Fall back to the runner's CLI discovery (`pytest --collect-only -q`, `jest --listTests`, `go test -list '.*' ./...`, etc.) |

   *Blackbox tests under `tests/blackbox/` are NOT unit tests* and are excluded — they are owned by the Dojo contract layer (pre-flights #1–#6).

   For every kept test file, a matching evidence file MUST exist at:

   ```
   tests/unit/.red-evidence/<encoded-path>.md
   ```

   `<encoded-path>` is the test file's path relative to the repo root, with `/` replaced by `__` and the extension stripped. Examples:

   - `tests/unit/test_validators.py` → `tests/unit/.red-evidence/tests__unit__test_validators.md`
   - `pkg/foo/foo_test.go` → `tests/unit/.red-evidence/pkg__foo__foo_test.md`
   - `src/utils/strings.test.ts` → `tests/unit/.red-evidence/src__utils__strings.test.md`

   The evidence file MUST contain at least one block per test function defined in the file, with this shape:

   ```markdown
   ## <test_function_name>

   - Command: `<exact shell command that was run>`
   - Captured: <ISO 8601 timestamp>
   - Exit code: <non-zero integer>

   ```
   <captured stderr/stdout including the assertion error and traceback>
   ```
   ```

   The `tests/unit/.red-evidence/` directory MUST be committed to the repository (it is testing evidence, not build output). If the project's `.gitignore` excludes it, remove the exclusion before validation can pass.

   If any discovered unit test file is missing its evidence file, or any test function in the file lacks a block, validation STOPS. The agent MUST recover the evidence before resuming:

   1. Identify the unit (function / method / struct method) under test in the matching implementation file.
   2. Temporarily revert the body of that unit to its pre-implementation state — typically a language-appropriate not-implemented placeholder (`raise NotImplementedError(...)` in Python, `throw new Error(...)` in JS/TS, `panic("not implemented")` in Go, `unimplemented!()` in Rust, `throw UnsupportedOperationException(...)` in Kotlin, etc.).
   3. Run the offending test in isolation with the project's test runner; capture stdout + stderr + exit code.
   4. Write the captured output to `tests/unit/.red-evidence/<encoded-path>.md` in the format above.
   5. Restore the implementation. Re-run the test to confirm green.
   6. Re-run validation from pre-flight #1.

   Forging the evidence file from memory is a constitutional violation and is detectable: the captured traceback contains real line numbers, file paths, runner-specific framing, and a real exit code that must match a real failed run. Do not fabricate.

   *Skip-with-justification.* If a unit-test file genuinely cannot be red-then-greened in isolation (e.g. a test that exercises a pure import-time constant), record the file path and reason in `tests/unit/.red-evidence/SKIPS.md`, one line per skip. The agent MUST cite each skip when reporting validation status.

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
9. Once steps 1–5 are all green, run the **Completion Verification** gate (see section below). The task is COMPLETE only when both the Execution Loop is green **and** the verification gate passes.

**Do not weaken the contract suite or the unit tests to make failures disappear.** If a test seems wrong, verify against `spec.md` first — the implementation is usually the side that needs to change. If a test is genuinely incorrect (fixture does not match the spec, unit test asserts against a stale interface), fix the test and note the divergence to the user.

## Completion Verification

After the Execution Loop reaches green (steps 1–5 passing), this gate runs **once** to verify FR/SC traceability before the task is declared COMPLETE. Dojo green proves the contract suite passes; this gate proves the contract suite actually covers every requirement in the spec.

### Detect Superpowers — defer if present

Check the available skills in your session (system reminder / skill index) for the Superpowers `verification-before-completion` skill. If present:

1. Announce: "Superpowers `verification-before-completion` will handle the gate — standing down."
2. Invoke it through its normal trigger context (the agent is about to claim completion). Superpowers will demand fresh verification evidence — the COVERAGE→dojo cross-check is the command that proves the claim.

If absent, run the inline procedure below.

### Inline procedure (standalone)

Two checks. Both MUST pass before the task is COMPLETE.

**1. FR/SC → green test mapping.** Parse `tests/blackbox/COVERAGE.md`. From the coverage matrix (§7.C) and the pre-flight worksheets (§7.A), extract every non-`(skip)` row mapping `FR-*` / `SC-*` to one or more test names. For each row, run `dojo --format json ./tests/blackbox/` and confirm the mapped test exists in the result, is not skipped, and is green. Emit a structured failure list naming each FR/SC whose mapped test is missing, renamed, skipped, or red.

**2. Bidirectional drift check.** Scan `spec.md` for `FR-*` / `SC-*` identifiers and diff against the Source column of COVERAGE.md. Flag identifiers present in `spec.md` but missing from COVERAGE.md (accidental deletion during refactor), and identifiers present in COVERAGE.md but absent from `spec.md` (stale rows pointing at deleted requirements). On drift, re-run `/speckit.sybilion-dojo.contract` to regenerate COVERAGE.md, then re-run the Execution Loop, then re-run this verification.

### Anti-patterns specific to agents

- **Treating "all dojo tests green" as sufficient.** Dojo green is necessary, not sufficient. A test can be silently disabled, accidentally satisfied by unrelated code, or renamed out of the matrix.
- **Declaring COMPLETE before running the gate fresh.** "I already verified this earlier" is not fresh evidence. The gate runs once per completion claim, not once per session.
- **Patching COVERAGE.md to make the drift check pass.** If `spec.md` mandates an FR the matrix does not have, add the test — do not delete the FR from the spec. Verify against the spec before changing it.

## Failure Modes

| Symptom | Likely cause | Fix |
|---|---|---|
| Type checker errors | Missing or wrong type annotations, or implementation type mismatch | Fix the implementation or narrow the annotation. Do not silence with broad `Any` / `# type: ignore` / `as any` without an inline justification per the constitution. |
| Unit test failures | Implementation breaks an invariant the unit test asserts | Fix the implementation. If the test itself is wrong, verify against `spec.md` before changing it — the implementation is usually the side that needs to change. |
| Required step has no tool configured (no type checker, no unit test runner) | The project's manifest does not declare the tool the constitution requires | Report explicitly which step is being skipped and why. Open a follow-up task to add the missing tool, or document the absence in **Project-Specific Rules** of the constitution. Never silently treat a missing tool as a passing step. |
| New unit test file has no entry under `tests/unit/.red-evidence/` | The agent skipped the RED phase (constitutional violation per Testing §TDD) or wrote test + implementation in the same turn | Apply the inline recovery procedure in Pre-flight #7 (revert the implementation, run the test, capture the failure to evidence file, restore implementation). Report the breach to the user. Do NOT proceed to green without captured evidence. |
| Red-evidence file exists but a test function block is missing | A new test function was added without the RED phase being observed | Same recovery as above, scoped to the missing test function. Append the new block to the existing evidence file. |
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
