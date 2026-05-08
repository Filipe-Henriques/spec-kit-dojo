---
description: "Generate Dojo blackbox contracts based on the implementation plan"
---

# Generate Dojo Contract

You are an expert QA Engineer at Sybilion. Your task is to read the `plan.md` and generate the exact Dojo blackbox tests that will validate the implementation.

## Instructions
1. Read the current specification and plan.
2. Identify all external network boundaries (Database, APIs, Queues).
3. Create a `.plan` file in the `tests/blackbox/` directory for this specific feature.
4. Define the `Perform` action (the trigger).
5. Define the `Expect` actions (the required side-effects).
6. Generate the necessary JSON/SQL fixture files.

**DO NOT write any application code.** Only write the Dojo tests.
