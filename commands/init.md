---
description: "Apply Sybilion constitution template before constitution generation"
---
# Sybilion Init

You are applying the Sybilion corporate constitution template. This command runs **before `/speckit.constitution`**, so when spec-kit's constitution generator runs next, it sees the Sybilion baseline and only needs to fill in the placeholders (`[PROJECT_NAME]`, version, dates, project-specific rules, governance additions).

- **Source:** `.specify/extensions/sybilion-dojo/templates/constitution-template.md` — the Sybilion template, installed by spec-kit when the extension is added to the consuming repo.
- **Target:** `.specify/memory/constitution.md` — the consuming project's constitution file, owned by spec-kit.

## Instructions

1. Read the source file. If it does not exist, **stop and report** — the extension is not installed correctly. Do not proceed.
2. Write its content **verbatim** to the target file, overwriting any existing content.
3. Verify the copy by reading the first three lines of the target:
   - Line 1 starts with `# ` and ends with `Constitution`.
   - Line 3 contains `This document is binding`.
   If either check fails, stop and report — the copy or the template is malformed.
4. Output: `✓ Sybilion constitution template applied.`

**Do not fill in any placeholders.** `/speckit.constitution` runs immediately after this hook and is responsible for filling `[PROJECT_NAME]`, `[CONSTITUTION_VERSION]`, `[RATIFICATION_DATE]`, `[LAST_AMENDED_DATE]`, `[PROJECT_SPECIFIC_RULES]`, and `[GOVERNANCE_RULES]`.

**Do not generate or modify any content.** This is a pure copy operation. Edits to the Sybilion baseline belong in the extension repo (`templates/constitution-template.md`), not in this hook.
