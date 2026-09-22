# Global GPT-6 project workflow

## Problem Statement

I use a global Sol-orchestrator workflow in OpenCode and Codex across different projects. It still pins GPT-5.6 Sol and Luna, and it has no Astra role. I want to use the GPT-6 models according to the work they do best without paying for Astra on routine work, losing an independent review step, or overriding each project's rules. I also want to try the new workflow before replacing my existing defaults.

## Solution

Provide an opt-in, global GPT-6 workflow in both OpenCode and Codex. Sol is the main orchestrator and reviewer. Luna handles bounded exploration, implementation, writing, and validation. Sol consults a read-only Astra adviser for difficult, consequential decisions that it cannot resolve confidently from the available evidence, or when I request Astra explicitly. Sol remains responsible for decisions and the final response. Keep the GPT-5.6 workflow available during the trial. Promote GPT-6 to the defaults only after successful client-level checks and my approval.

## User Stories

1. As a developer, I want one workflow available in different repositories, so that I do not copy agent configuration into each project.
2. As a developer, I want the workflow available in OpenCode, so that I can use my existing OpenCode setup with GPT-6 models.
3. As a developer, I want the same role boundaries in Codex, so that changing clients does not change who implements and who reviews.
4. As a developer, I want each repository's instructions to apply, so that the global workflow follows project-specific conventions and constraints.
5. As a developer, I want Sol to handle requirements and planning, so that workers receive bounded tasks with clear acceptance criteria.
6. As a developer, I want Sol to split broad work into bounded assignments, so that Luna is not asked to resolve an underspecified project-wide change alone.
7. As a developer, I want Luna to investigate relevant code and report evidence, so that Sol can make decisions based on the actual project.
8. As a developer, I want Luna to implement well-defined changes, so that routine work uses the efficient model.
9. As a developer, I want Luna to run focused validation and report failures plainly, so that Sol can review both the change and the evidence.
10. As a developer, I want Sol to inspect the affected behavior and diff, so that Luna does not approve its own work.
11. As a developer, I want Sol to request concrete revisions when it finds defects, so that the worker can correct the same task without losing context.
12. As a developer, I want Sol to retain decision-making and final-response responsibility, so that agent handoffs do not obscure accountability.
13. As a developer, I want Astra consulted for unresolved, high-impact design tradeoffs, so that the hardest decisions receive deeper scrutiny.
14. As a developer, I want Astra consulted for unresolved security or data-loss risks, so that consequential uncertainty is examined before implementation proceeds.
15. As a developer, I want Astra consulted when repeated revisions reveal a design problem, so that the workflow addresses the cause rather than looping on patches.
16. As a developer, I want to request Astra at the start of a task, so that I can override the automatic routing decision.
17. As a developer, I want Sol to state why it called Astra, so that I can tell when the extra model was warranted.
18. As a developer, I want Astra to advise without editing or replacing Sol's decision, so that escalation does not break the review boundary.
19. As a developer, I want routine failures handled by Sol and Luna without an automatic Astra call, so that a large diff or failed test alone does not raise cost.
20. As a developer, I want simple conversation answered without worker handoffs, so that the workflow adds no needless delay to non-project questions.
21. As a developer, I want a separate GPT-6 trial entry point in each client, so that my existing GPT-5.6 defaults remain usable while I test it.
22. As a developer, I want the trial agents to pin their intended GPT-6 models, so that switching only the primary does not silently leave Luna on GPT-5.6.
23. As a developer, I want an unavailable model reported explicitly, so that I do not mistake a fallback for a successful GPT-6 trial.
24. As a developer, I want Codex implementation sessions to let Luna edit, so that its parent sandbox setting does not make the worker read-only.
25. As a developer, I want to see a successful implementation, validation, and Sol review in each client, so that I know the handoff works end to end.
26. As a developer, I want to see a targeted Astra consultation in each client, so that I know escalation routes to the intended model.
27. As a developer, I want the trial results before changing defaults, so that I can decide whether to promote the workflow.

## Implementation Decisions

- Use a globally discoverable shared workflow skill for the client-neutral delegation protocol. OpenCode agent definitions and a Codex profile and subagent definitions provide client-specific entry points and permissions.
- Add a separate GPT-6 Sol primary in OpenCode and a separate GPT-6 Sol profile in Codex. Keep existing GPT-5.6 defaults and agent definitions intact during the trial.
- Pin GPT-6 Sol, Luna, and Astra in their respective trial definitions. Do not depend on changing a global primary model to change a worker's explicitly configured model.
- Sol owns clarification, scope, decomposition, architecture decisions, acceptance criteria, review, and the final response. It may inspect targeted files and diffs, but it does not edit or repair the worker's changes.
- Luna receives bounded discovery or implementation requests, follows repository instructions, preserves unrelated work, runs relevant checks, and returns cited evidence. Discovery-only requests do not change files. Luna does not approve its own work, delegate further, commit, push, or perform destructive actions without an explicit user request.
- Sol reviews Luna's changes and validation evidence and returns ACCEPT, REVISE, or ESCALATE. It sends concrete findings to the same worker thread when possible. After at most two unsuccessful revision rounds, it reports the blocker rather than silently continuing.
- Astra is a read-only adviser. Sol gives it a specific question, the available evidence, constraints, and the decision needed. Astra does not replace Sol as orchestrator or final reviewer.
- Automatically consult Astra only when a consequential design or security/data-loss decision remains unresolved after Sol examines the evidence, or when repeated revisions expose such a design problem. An explicit user request also triggers consultation. Size or a failed check alone is insufficient.
- Apply the agent workflow to project work such as implementation, debugging, project research, documentation, and review. Answer simple conversations directly.
- In OpenCode, keep the primary agent's edit permission denied while allowing the Luna worker to write and restricting the Astra adviser to reading.
- In Codex implementation trials, select workspace-write permission for the parent turn before delegation. Codex may reapply the parent's live sandbox settings to subagents. Sol's no-edit rule in that session is instructional rather than a separate enforceable sandbox boundary. Read-only sessions remain available for analysis.
- Keep the trial opt-in. Do not promote it to the default automatically or silently substitute an unavailable model. Report model-routing failures to the user.
- No project schema, application API, or repository-wide configuration migration is part of this workflow.

## Testing Decisions

- Test external behavior at the client entry points rather than asserting prompt text or internal handoff mechanics. The trial should show which model handled each role, what work it returned, what validation ran, and what Sol decided.
- Use one representative project change in OpenCode and one in Codex. In each, verify a Luna edit, focused validation, and an independent Sol review verdict. Preserve the project's own instructions and existing changes during the check.
- Use one targeted, difficult advisory question in each client to verify that Astra is reached as a read-only adviser, Sol states the reason for escalation, and Sol makes the final decision.
- Check model unavailability explicitly if encountered: the workflow must report the failure and must not count a fallback as a successful trial.
- No new application-level test seam is needed. The existing client invocation and visible agent results are the highest useful seam for this configuration change.
- Prior art is the current Sol/Luna workflow's delegation packet, worker report, and ACCEPT/REVISE/ESCALATE review gate. The pilot checks exercise those existing contracts through the clients.

## Out of Scope

- Replacing the GPT-5.6 defaults before the trial results receive user approval.
- Project-specific agent rules, application code changes, or copying global definitions into each repository.
- A custom router, benchmark framework, telemetry pipeline, or automatic model fallback.
- Routine Astra participation in every task or review.
- Automatic commits, pushes, or destructive actions.

## Further Notes

- The current workflow has a GPT-5.6 Sol primary and a GPT-5.6 Luna worker in both clients. Its shared skill already defines delegation packets and review verdicts, so the new protocol should reuse that behavior where appropriate.
- The current Codex parent configuration is read-only, while its Luna worker requests workspace-write. Parent-turn runtime sandbox overrides can constrain the child, which is why implementation trials must select workspace-write before spawning Luna.
- Local model catalogs list the GPT-6 model IDs, but that does not prove the configured provider route or account can serve them. The trial must verify actual calls.
- No project issue tracker, triage labels, domain glossary, or relevant ADRs were identified from the current home-directory context. Publication and the ready-for-agent label require the project tracker to be configured.
