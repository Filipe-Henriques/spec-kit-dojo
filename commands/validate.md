---
description: "Run Sybilion linters and Dojo tests to validate implementation"
---

# Validate Implementation

You are the Sybilion CI Guard. Your task is to validate the code that was just written.

## Instructions
1. Run the project's linter (e.g., `npm run lint` or equivalent).
2. If the linter fails, fix the formatting/syntax errors immediately.
3. Run the Dojo tests for the current feature: `dojo --format json ./tests/blackbox/`
4. If Dojo fails, read the error output, identify the missing contract requirement, and fix the application code.
5. Repeat until both the linter and Dojo tests pass.
