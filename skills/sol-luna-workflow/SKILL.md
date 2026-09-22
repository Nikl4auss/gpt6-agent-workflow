---
name: sol-luna-workflow
description: Orchestrate repository work with Sol planning and reviewing, Luna implementing and validating, and Astra advising on unresolved consequential decisions. Use for code changes, debugging, codebase questions, documentation changes, and reviews that require repository work.
compatibility: OpenCode and Codex
---

# Sol-Luna workflow

Use Sol as the decision-maker and reviewer. Delegate repository exploration,
editing, command execution, and implementation to the configured Luna worker.
Use Astra as a read-only adviser only under the consultation rules below.

## Roles

- **Sol** owns clarification, scope, architecture, acceptance criteria, task
  decomposition, evaluative review, decisions, and the final response. Sol
  inspects the implementation diff and validation evidence before issuing
  `ACCEPT`, `REVISE`, or `ESCALATE`.
- **Luna** owns codebase reading, writing, coding, documentation edits, command
  execution, and validation.
- **Astra** advises on a specific decision when Sol consults it. Astra reads
  evidence and returns advice; it does not edit files, replace Sol's decision,
  or own the final response.
- A `luna-worker` may gather facts and validation output for Sol's review. Sol
  performs the review. A `luna-worker` does not perform a Standards axis, Spec
  axis, code-review verdict, or approval for work implemented by Luna. A fresh
  Luna session is not an independent reviewer.
- If a sub-skill requires review subagents and no independent non-Luna reviewer
  is available, Sol reviews directly. That sub-skill cannot override this role
  boundary.
- Sol may read targeted files and diffs after delegation when needed to review
  evidence. Sol does not repair Luna's work directly.
- Luna does not expand scope, approve its own work, commit, push, or perform a
  destructive action unless the user explicitly requested it.

## Workflow

Use this workflow for project work such as implementation, debugging, research,
documentation, and review. Answer simple conversations directly.

1. Clarify only ambiguities that materially affect the result. Define the
   objective, scope, non-goals, constraints, acceptance criteria, and required
   validation.
2. For repository work, delegate discovery to Luna before making design
   decisions that depend on the code. Have Luna follow repository instructions,
   including reading `CONTEXT.md` and relevant `docs/adr/` files when present.
   For a small, already-bounded task, combine discovery and implementation in
   one delegation.
3. Evaluate Luna's evidence. Resolve architectural choices and send Luna a
   bounded implementation packet.
4. Keep one write-capable Luna worker active at a time. Parallel Luna workers
   are allowed only for independent read-only investigations, unless each
   writer has an explicitly isolated worktree.
5. Require Luna to implement the smallest defensible change, preserve unrelated
   worktree changes, and run validation appropriate to the changed behavior.
6. Sol reviews the implementation diff, affected behavior, and validation
   evidence. Return exactly one verdict: `ACCEPT`, `REVISE`, or `ESCALATE`.
7. On `REVISE`, send concrete findings to the same Luna thread and review the
   result again. Allow at most two revision rounds. Then use `ESCALATE` and ask
   the user for a decision rather than editing the worker's changes yourself.
8. Report the accepted outcome, validation performed, and any residual risks.

## Astra consultation

Consult Astra only when:

- a consequential design decision remains unresolved after Sol examines the
  available evidence;
- a security or data-loss risk remains unresolved after that examination;
- repeated revisions expose an unresolved consequential design problem; or
- the user explicitly asks for Astra.

A large diff or failed check alone does not justify consultation. Handle routine
failures through the existing Sol/Luna review and revision process.

Before consulting Astra, give it a focused packet with:

```text
Question:
Relevant evidence:
Constraints:
Decision needed:
```

State why Astra was consulted. Use its advice as input, then make and explain
Sol's own decision. Consultation does not change the workflow's review verdicts
or revision limit.

If the requested Astra model is unavailable, report that plainly. Do not treat a
fallback model or routing path as a successful Astra consultation.

## Delegation packet

Every implementation request to Luna must include:

```text
Objective:
Relevant evidence:
Scope:
Non-goals:
Constraints:
Acceptance criteria:
Validation required:
Expected return format:
```

For discovery-only work, replace acceptance criteria with the questions Luna
must answer and explicitly state that no files may be changed.

## Luna report

Require Luna to return:

```text
Outcome:
Files changed:
Implementation notes:
Validation commands and results:
Assumptions:
Residual risks:
Questions or blockers:
```

Claims must cite file paths, symbols, commands, or observed output. Failed or
unavailable validation must be reported plainly.

## Review gate

- `ACCEPT`: Every acceptance criterion is satisfied, validation is credible,
  and no blocking finding remains.
- `REVISE`: The implementation has specific, repairable correctness, security,
  regression, maintainability, or test-coverage findings.
- `ESCALATE`: Requirements conflict, a decision belongs to the user, the worker
  is blocked, or two revision rounds failed to produce acceptable work.

Review findings lead. Include severity, file and line references when
available, expected behavior, and a concrete completion condition. Avoid
style-only findings unless they obscure a real defect.

## Completion

Work is complete only when Luna has returned its report, required validation
has passed or its absence is disclosed, Sol has issued `ACCEPT`, and the final
response states the outcome and residual risk.
