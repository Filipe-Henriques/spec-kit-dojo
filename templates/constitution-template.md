# Constitution

## Core Principles
- **Contract-Driven Development:** All features MUST have a Dojo contract defined before implementation begins.
- **Test-Driven:** Code is only considered complete when it passes the Dojo blackbox tests.
- **Single Responsibility:** Every module, class, and function must have one clear responsibility.
- **Explicit over Implicit:** Prefer clarity over cleverness. Code should be readable without comments.

## Sybilion Python Engineering Standards

### Code Quality
- **Formatter:** All code MUST be formatted with `ruff format` before committing.
- **Linter:** All code MUST pass `ruff check` with zero errors before committing.
- **Type Hints:** All function signatures MUST have type hints. No untyped `def` is allowed.
- **No bare exceptions:** Never use `except Exception` or `except:` without logging the error.

### Project Structure
- **Package layout:** Use `src/` layout. Application code lives in `src/<package_name>/`.
- **Dependencies:** Use `pyproject.toml` for dependency management. No `requirements.txt`.
- **Environment variables:** All config MUST come from environment variables. Never hardcode secrets.
- **Logging:** Use Python's standard `logging` module. Never use `print()` for application output.

### API & Data
- **Validation:** All external input (API payloads, env vars) MUST be validated with `pydantic`.
- **HTTP clients:** Use `httpx` for all HTTP calls. Never use `requests`.
- **Database:** Use raw SQL with a connection pool. No ORMs unless explicitly approved.

## Tooling Versions
- **Spec-Kit:** Must be >= 0.8.0
- **Dojo:** Must be installed via `go install github.com/elmacnifico/dojo@latest`. If the `dojo` command is not found, the agent MUST install it before running validations.
- **Python:** Must be >= 3.11
- **Ruff:** Must be installed via `uv tool install ruff`

## Project Specific Rules
[PROJECT_SPECIFIC_RULES]
