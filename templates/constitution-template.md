# [PROJECT_NAME] Constitution

This document is binding. When other documentation conflicts with this file, this file wins. The agent MUST treat these rules as overriding any default behavior or assumptions.

## Core Principles

### 1. Contract-First Development

Every feature MUST have a Dojo blackbox contract defined and reviewed BEFORE implementation begins. Tests are written, the contract is reviewed, tests fail against the unimplemented code, and only then is implementation begun. Code is COMPLETE only when the Dojo contract suite passes — a green local test loop without a green Dojo run is not complete.

### 2. Determinism by Default

Same inputs MUST produce same outputs. Any randomness MUST be explicitly seeded via a parameter with a documented default. No reliance on global RNG state, wall-clock time, or environmental conditions for behavior.

### 3. Fail Fast, No Silent Coercion

Invalid inputs MUST raise clear, actionable errors at the boundary. No defaulting, implicit repair, or swallowing exceptions. Error messages MUST help the caller correct the input.

### 4. Small Surface, Stable Contract

Keep the public API small and intentional. Internal modules may change freely; public symbols MUST NOT change without an explicit task. Prefer vertical slices end-to-end over horizontal sprawl, and single-responsibility components over coupled mega-modules.

---

## Engineering Standards

### Code Quality

- All code MUST be formatted by the project's configured formatter before commit.
- All code MUST pass the project's configured linter with zero errors before commit. Do not suppress rules without an explicit, justified inline comment.
- Where the language supports static typing, types MUST be declared explicitly on every public function signature. Untyped public boundaries are forbidden.
- No bare or catch-all exception handlers. If a broad catch is genuinely necessary, the caught error MUST be logged with context before re-raising or handling.
- Use the language's standard logging facility. No raw `stdout` / `stderr` output for application events. The library MUST NOT configure global handlers or logging state on import.
- Configuration and secrets MUST come from environment variables or a project-approved secret store. No hardcoded secrets, API keys, or credentials in source.

### Project Structure & Layering

- The project MUST define explicit layers (e.g. `core` / domain modules / orchestration / `tools` or IO). Define them once, document them, and enforce them.
- Layers MUST NOT depend upward. The foundational layer (contracts, types, utilities) MUST NOT depend on any other layer.
- I/O — filesystem, network, environment access — MUST be confined to designated wrapper layers. Algorithmic / domain / `core` layers MUST be I/O-free.
- Modules MUST be usable standalone on the core contract types AND through thin wrappers. Wrappers MUST NOT contain business or algorithmic logic.
- No hidden global state. State flows through explicit parameters or constructors.
- Cross-cutting concerns (serialization, logging, configuration, error types) MUST be centralized. No local re-implementations of an already-provided shared utility.

### Public API Discipline

- Public symbols MUST be explicitly re-exported from the project's package entry point. Anything not re-exported is internal and unstable.
- Internal modules MUST NOT be imported by user-facing code.
- New public symbols, or any breaking change to an existing public symbol, requires an explicit task — never as a side effect of another change.

### Error Handling

- All user-facing exceptions MUST derive from a single project-defined base error class.
- Unsupported operations MUST raise the language's "not implemented" equivalent with a message that names the operation and the reason.
- Error messages MUST be actionable: state what was wrong AND what the caller should do.

### Dependencies

- All runtime and development dependencies MUST be declared in the language's standard manifest. No installation outside the manifest.
- Versions MUST be pinned wherever the manifest supports it.
- New dependencies require explicit approval. Prefer the standard library or in-project utilities over adding a dependency.
- Optional dependencies belong in extras / feature groups and MUST raise a clear, project-defined error when missing at runtime. Optional dependencies MUST NOT affect default behavior when absent.
- After any dependency change, the manifest's lock / sync step MUST be run. No ad-hoc package installs.

### Testing

- Every new module MUST include tests.
- Minimum coverage: core logic, edge cases (empty / minimal / extreme inputs), determinism where applicable, and leakage prevention where the domain demands it.
- Tests SHOULD import the public API rather than internal modules.
- Shared deterministic fixtures SHOULD be reused across tests to keep the suite low-churn.

**Test-Driven Development at the unit level (NON-NEGOTIABLE).** This is the **inner** loop within Principle 1's contract-first cycle. Unit red-green produces the implementation that the Dojo contract suite ultimately validates. The contract suite remains the completion gate established by Principle 1 and the Execution Loop — this rule governs **how each line is written**, not when the system is done.

Cycle, in order, per behaviour: unit test written → fails for the expected reason (run and observed) → minimal implementation → test passes (run and observed) → refactor with tests green.

Mandatory between RED and GREEN: the agent MUST run the failing test and observe the failure output before writing implementation. Producing a test and its implementation in the same agent turn is forbidden — collapsed RED+GREEN is a constitutional violation.

Anti-patterns specific to agents:

- **Test deletion or weakening to make the suite pass.** Failing tests are fixed by changing the implementation, never the test. If the test itself is wrong, verify against `spec.md` first.
- **Self-validating circular tests.** A test generated from the same prompt as the implementation is suspect; subject it to the same scrutiny as a junior-developer-authored test.
- **No recorded failure observation.** "I wrote a test for it" is insufficient — the agent MUST be able to cite the failing-run output that preceded the green run.

Allowed exception: if a Dojo contract test is the first failing test that exercises a new behaviour, no separate unit test is required for that behaviour. The rule is "no production code without a failing **unit or contract** test that exercises it first" — not "no production code without a failing unit test". Even under this exception, the cycle still applies at the contract level: the contract test MUST be observed failing for the expected reason before implementation, and observed passing afterwards. The exception removes the *duplication*, not the *discipline*.

When a Claude Code plug-in provides equivalent TDD discipline (e.g. Superpowers' `test-driven-development` skill), it satisfies this rule automatically; the rules above are the standalone enforcement.

### Documentation

- All public classes, functions, and methods MUST have docstrings (or the language's equivalent) describing purpose, parameters, and return values.
- The summary line MUST be one sentence, ≤ 80 characters, describing WHAT, not HOW.
- Do not duplicate type information already present in type hints.
- A docstring exceeding ~10 lines for a function or ~15 lines for a class SHOULD be shortened unless verbosity is explicitly requested.

### Performance

- Avoid unnecessary copies of large data structures. Prefer in-place / vectorized operations where the language supports them idiomatically.
- Document any operation with non-obvious complexity (worse than O(n) in the input size).
- Do NOT micro-optimize unless explicitly requested. Correctness and clarity outrank speed.

---

## Workflow Discipline

### Sources of Truth

- The project MUST maintain a single authoritative location for behavioral requirements, architectural decisions, and active tasks. For spec-kit projects, per-feature specs / plans / tasks live under `specs/<branch-name>/`.
- The agent MUST consult these documents before implementing. The agent MUST NOT invent requirements that are not stated there.
- If something is unclear or missing, pause and propose a new task entry. Do not guess.

### Execution Loop (Mandatory)

After every code change, before declaring a task complete, the agent MUST run, in order:

1. The project's formatter.
2. The project's linter — fix every reported issue (do not suppress without justification).
3. The project's type checker, if the language supports one — fix every reported issue.
4. The project's unit test suite.
5. `dojo ./tests/blackbox/` against the contract suite.
6. If any step fails, fix the root cause and restart from step 1.
7. A step that genuinely cannot be run MUST be reported, not skipped silently.

### Task Discipline

- Work only on the scope explicitly listed in the current task entry.
- New scope discovered mid-task MUST become a new task entry — never improvise outside the stated scope.
- A task is COMPLETE only when: code is written, tests are added or updated, the execution loop above is green, and the Dojo contract passes.

### Review Mindset

- The agent is an implementer, not an architect.
- When uncertain, pause and ask. Do not guess. Do not invent APIs.
- Correctness, reproducibility, and contract compliance outrank speed.

### Special Files

- The agent MUST respect the project's ignore lists (`.gitignore`, `.dojoignore`, and any additional ignore files the project declares). Files matched by those lists MUST NOT be read or modified unless the user explicitly asks.

---

## Tooling Versions

- **Spec-Kit:** Must be >= 0.8.0.
- **Dojo:** Must be available on the `PATH`. Install via `go install github.com/elmacnifico/dojo@latest`. If the `dojo` command is not found, the agent MUST install it before running validations.
- **Language toolchain (formatter / linter / type checker / test runner):** declared and pinned in the project manifest. See **Project-Specific Rules** for the concrete tools and versions for this project.

---

## Project-Specific Rules

[PROJECT_SPECIFIC_RULES]

---

## Governance

This constitution supersedes all other project practices, conventions, and personal preferences. Where any other document, comment, or convention contradicts this constitution, this constitution wins.

**Amendments.** Changes to this constitution require:

1. A written proposal describing the change and its rationale.
2. Explicit team approval (record approvers in the amendment commit).
3. A bump of the version below, an updated `Last Amended` date, and migration notes for any rule that breaks existing behavior.

**Compliance.** Constitution compliance is verified at two checkpoints:

- The Dojo contract suite (`/speckit.sybilion-dojo.contract` + `/speckit.sybilion-dojo.validate`) MUST pass before any feature is considered complete.
- Code review on pull requests MUST flag any constitution violation. Reviewers may block merges that violate this document.

**Exceptions.** If a constitution rule genuinely cannot be followed for a given task, the agent MUST document the deviation inline with a justification AND open a follow-up task to either fix the deviation or amend the rule. Silent deviations are violations.

[GOVERNANCE_RULES]

---

**Version**: [CONSTITUTION_VERSION] | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
