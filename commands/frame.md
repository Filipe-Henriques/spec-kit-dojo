---
description: "Frame the problem before /speckit.specify drafts the spec"
---
# Sybilion Frame

You are about to run `/speckit.specify`. This hook fires **before** it, to make sure the spec captures the real problem — not the first plausible interpretation.

Speccing without framing produces specs that solve the literal request while missing the actual goal, bake in constraints the user did not realise they were stating, and skip cheaper alternatives. Your job: extract the real intent through structured questioning, then present the agreed framing for explicit approval. Only then does `/speckit.specify` run.

## Detect Superpowers — Defer If Present

Before running the Method below, check the available skills in your session (system reminder / skill index) for the Superpowers `brainstorming` skill.

If it IS present:

1. Announce: "Superpowers `brainstorming` will handle the framing — standing down."
2. End this command without questioning. Superpowers auto-activates on the same context and would otherwise double-question the user.

If it IS NOT present, run the Method below yourself.

## Distinction from `/speckit.clarify`

This command runs **before** `/speckit.specify` to frame the problem so the spec is drafted right the first time. The existing `/speckit.clarify` runs **after** `/speckit.specify` to resolve ambiguities in the drafted spec. They are sequential and complementary, not overlapping.

## Method

1. **One question at a time.** Multiple questions in a single message overwhelm. Wait for the answer before asking the next.
2. **Multiple-choice when possible.** "REST, gRPC, or GraphQL?" beats open-ended. Open-ended only when no reasonable options exist.
3. **Apply YAGNI ruthlessly.** Each question MUST be necessary to write the spec. If the answer would not change the spec, do not ask.
4. **Explore 2–3 alternatives before proposing.** For any non-trivial design choice, present viable options with trade-offs. Recommend one; let the user override.
5. **Present the agreed framing for approval.** Before stopping, summarise:
   - The problem in one sentence.
   - Scope (in / out).
   - The 2–3 most consequential design choices and the decision for each.
   - Ask explicitly: "Aprovado para gerar a spec?" — wait for an explicit yes.

## Output Contract

This command writes **no files**. Its output is alignment with the user. `/speckit.specify` runs immediately after and writes `specs/<branch>/spec.md`.

Do NOT: write code or files, draft the spec yourself, or skip approval.
