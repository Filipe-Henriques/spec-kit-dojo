# Constitution

## Core Principles
- **Contract-Driven Development:** All features MUST have a Dojo contract defined before implementation begins.
- **Test-Driven:** Code is only considered complete when it passes the Dojo blackbox tests.

## Sybilion Engineering Standards
- **Linting:** All code MUST pass the standard Sybilion linter before being committed.
- **Types:** Strict typing is mandatory. No `any` in TypeScript.
- **Logging:** Use the internal Sybilion logger, never `console.log`.

## Tooling Versions
- **Spec-Kit:** Must be >= 0.8.0
- **Dojo:** Must be installed via `go install github.com/elmacnifico/dojo@latest`. If the `dojo` command is not found, the agent MUST install it before running validations.

## Project Specific Rules
[PROJECT_SPECIFIC_RULES]
