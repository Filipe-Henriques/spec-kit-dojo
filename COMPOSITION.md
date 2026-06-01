# Companion Tooling Composition

How `sybilion-dojo` composes with third-party Claude Code plug-ins. Rules here apply only when the named plug-in is present; the extension's hooks and commands work standalone otherwise.

> This file lives outside the constitution by design. The constitution
> ([`templates/constitution-template.md`](templates/constitution-template.md))
> encodes engineering invariants that apply to the project regardless of
> tooling. Tool-coupling rules belong here — they decay independently of the
> code.

## Superpowers ([obra/superpowers](https://github.com/obra/superpowers))

Install: `/plugin install superpowers@claude-plugins-official`

### Rules

1. **Pre-spec framing — defer.** When the `brainstorming` skill is present in
   the session, `/speckit.sybilion-dojo.frame` stands down on detection. Both
   gates running in parallel would double-question the user. The deference is
   one-way: Superpowers does not (and need not) know about `frame`.

2. **TDD demarcation.** The Dojo blackbox contract suite
   (`/speckit.sybilion-dojo.contract`) owns the *contract* layer. The
   Superpowers `test-driven-development` skill applies to the *unit-test* layer
   only. Both red/green loops MUST be green before a task can be declared
   COMPLETE. The Dojo contract is the semantic gate (per constitution
   Principle 1); the unit-test layer is the implementation discipline that
   produces the green contract. They are parallel disciplines, not duplicates
   — the contract layer asserts observable behaviour from outside; the unit
   layer asserts implementation internals from inside.

   *Red-evidence capture — defer.* When the `test-driven-development` skill is
   present, `validate` pre-flight #7 stands down: the skill enforces red-then-
   green at write time and produces its own evidence. When the skill is
   absent, the inline standalone enforcement applies — for every unit test
   file added on the branch (discovered via the project's test runner
   convention, not a single hard-coded path), an evidence file MUST exist at
   `tests/unit/.red-evidence/<encoded-path>.md` (the test file's path with
   `/` → `__` and extension stripped) with one captured-failure block per
   test function (exact command, ISO 8601 timestamp, non-zero exit code,
   captured stderr/stdout). The `.red-evidence/` directory is testing
   evidence and MUST be committed. Without it, validation fails closed.

3. **Unknown validate failures — defer.** When
   `/speckit.sybilion-dojo.validate` encounters a symptom outside its Failure
   Modes table, the tail row defers to the Superpowers `systematic-debugging`
   skill if present, otherwise runs the 4-phase root-cause process inline
   (reproduce → isolate → identify root cause → fix).

4. **Completion verification — defer.** When the
   `verification-before-completion` skill is present in the session, the
   `validate` command's Completion Verification gate defers to it. The skill
   enforces the Iron Law "no completion claims without fresh verification
   evidence" — for us, the COVERAGE→dojo cross-check is the command that
   proves the claim. The standalone inline procedure (FR/SC → green test
   mapping + bidirectional drift between `spec.md` and `COVERAGE.md`) is the
   equivalent enforcement when the skill is absent.

### Known Limitations

- **`before_specify` is `optional: true`.** Spec-Kit
  [issue #2149](https://github.com/github/spec-kit/issues/2149) documents that
  some agents (notably Cursor) do not reliably honour `optional: false` on
  hooks. The hook is therefore advisory; the discipline rests on the agent
  reading the constitution and following its "MUST" rules. For Claude Code,
  this works as expected.

- **`frame.md` uses text-based numbered options for questions.** Spec-Kit
  [issue #2181](https://github.com/github/spec-kit/issues/2181) is migrating
  user-clarification UI to the native `AskUserQuestion` tool. `frame.md` will
  be refactored to use it once the upstream API stabilises.

## Adding a new companion plug-in

Future companions follow the same contract:

1. Add a section above with install command, rules, and limitations.
2. If the plug-in provides a skill that overlaps a `sybilion-dojo` command, the
   command MUST detect-and-defer (one-way). Never assume the other side knows
   about you.
3. Do NOT add tool-coupling rules to the constitution template — they belong
   here.
