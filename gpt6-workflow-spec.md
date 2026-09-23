# Global GPT-6 project workflow

## Problem Statement

I use a global Sol-orchestrator workflow in OpenCode and Codex across different projects. I want GPT-6 Sol and Luna according to the work they do best without paying for Astra on routine work, losing an independent review step, or overriding each project's rules. The GPT-6 workflow passed both client trials and is now the approved default, while the GPT-5.6 setup remains available for rollback.

## Solution

Provide a global GPT-6 workflow in both OpenCode and Codex. GPT-6 Sol is the main orchestrator and reviewer; GPT-6 Luna handles bounded exploration, implementation, writing, and validation. Sol consults a read-only GPT-6 Astra adviser for difficult, consequential decisions that it cannot resolve confidently from the available evidence, or when I request Astra explicitly. Sol remains responsible for decisions and the final response. Keep the GPT-5.6 workflow available as rollback material.

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
21. As a developer, I want the tested GPT-6 entry points retained in each client, so that the promoted defaults and GPT-5.6 rollback setup remain usable.
22. As a developer, I want the trial agents to pin their intended GPT-6 models, so that switching only the primary does not silently leave Luna on GPT-5.6.
23. As a developer, I want an unavailable model reported explicitly, so that I do not mistake a fallback for a successful GPT-6 trial.
24. As a developer, I want Codex implementation sessions to let Luna edit, so that its parent sandbox setting does not make the worker read-only.
25. As a developer, I want to see a successful implementation, validation, and Sol review in each client, so that I know the handoff works end to end.
26. As a developer, I want to see a targeted Astra consultation in each client, so that I know escalation routes to the intended model.
27. As a developer, I want the trial results before changing defaults, so that I can decide whether to promote the workflow.

## Implementation Decisions

- Use a globally discoverable shared workflow skill for the client-neutral delegation protocol. OpenCode agent definitions and a Codex profile and subagent definitions provide client-specific entry points and permissions.
- Use the promoted GPT-6 Sol primary in OpenCode and Codex. Rollback restores OpenCode `model = openai/gpt-5.6-sol` and `default_agent = sol-orchestrator`; Codex `model = gpt-5.6-sol`, `agents.default_subagent_model = gpt-5.6-luna`, and developer instructions delegated to `luna_worker`. Keep Codex `sandbox_mode = read-only` during rollback.
- Preserve the existing GPT-5.6 agents at `~/.config/opencode/agents/sol-orchestrator.md`, `~/.config/opencode/agents/luna-worker.md`, and `~/.codex/agents/luna-worker.toml`. The GPT-6 trial profile and agent files are known-good GPT-6 configuration, not GPT-5.6 rollback material.
- The tested global GPT-6 agent files are OpenCode `~/.config/opencode/agents/gpt6-{sol,luna,astra}-trial.md` and Codex `~/.codex/agents/gpt6-{sol,luna,astra}.toml`.
- Pin GPT-6 Sol, Luna, and Astra in their respective trial definitions. Do not depend on changing a global primary model to change a worker's explicitly configured model.
- Sol owns clarification, scope, decomposition, architecture decisions, acceptance criteria, review, and the final response. It may inspect targeted files and diffs, but it does not edit or repair the worker's changes.
- Luna receives bounded discovery or implementation requests, follows repository instructions, preserves unrelated work, runs relevant checks, and returns cited evidence. Discovery-only requests do not change files. Luna does not approve its own work, delegate further, commit, push, or perform destructive actions without an explicit user request.
- Sol reviews Luna's changes and validation evidence and returns ACCEPT, REVISE, or ESCALATE. It sends concrete findings to the same worker thread when possible. After at most two unsuccessful revision rounds, it reports the blocker rather than silently continuing.
- Astra is a read-only adviser. Sol gives it a specific question, the available evidence, constraints, and the decision needed. Astra does not replace Sol as orchestrator or final reviewer.
- Automatically consult Astra only when a consequential design or security/data-loss decision remains unresolved after Sol examines the evidence, or when repeated revisions expose such a design problem. An explicit user request also triggers consultation. Size or a failed check alone is insufficient.
- Apply the agent workflow to project work such as implementation, debugging, project research, documentation, and review. Answer simple conversations directly.
- In OpenCode, keep the primary agent's edit permission denied while allowing the Luna worker to write and restricting the Astra adviser to reading.
- In the normal Codex workflow, select workspace-write for the parent session when Luna must edit. Use a separate read-only session for Sol's review and any Astra advice. Codex may reapply the parent's live sandbox settings to subagents; Sol's no-edit rule during workspace-write turns is instructional rather than a separate enforceable sandbox boundary. Astra's metadata test is permission-profile evidence, not proof of a rejected write or evidence that separately approved escalation is impossible.
- Keep the approved GPT-6 selectors as defaults. Do not silently substitute an unavailable model; report model-routing failures to the user. Rollback remains a deliberate selector change using the preserved GPT-5.6 files.
- No project schema, application API, or repository-wide configuration migration is part of this workflow.

## Testing Decisions

- Test external behavior at the client entry points rather than asserting prompt text or internal handoff mechanics. The trial should show which model handled each role, what work it returned, what validation ran, and what Sol decided.
- Use one representative project change in OpenCode and one in Codex. In each, verify a Luna edit, focused validation, and an independent Sol review verdict. Preserve the project's own instructions and existing changes during the check.
- Use one targeted, difficult advisory question in each client to verify that Astra is reached as a read-only adviser, Sol states the reason for escalation, and Sol makes the final decision.
- Check model unavailability explicitly if encountered: the workflow must report the failure and must not count a fallback as a successful trial.
- No new application-level test seam is needed. The existing client invocation and visible agent results are the highest useful seam for this configuration change.
- Prior art is the current Sol/Luna workflow's delegation packet, worker report, and ACCEPT/REVISE/ESCALATE review gate. The pilot checks exercise those existing contracts through the clients.

## Out of Scope

- Changing either default without explicit user approval.
- Project-specific agent rules, application code changes, or copying global definitions into each repository.
- A custom router, benchmark framework, telemetry pipeline, or automatic model fallback.
- Routine Astra participation in every task or review.
- Automatic commits, pushes, or destructive actions.

## Further Notes

- The promoted workflow has GPT-6 Sol as the primary and GPT-6 Luna as the default worker in both clients. Its shared skill already defines delegation packets and review verdicts, so the protocol reuses that behavior where appropriate.
- The current Codex parent configuration is read-only for ordinary analysis. When Luna must edit, the normal workflow uses a workspace-write parent session, followed by a separate read-only session for Sol review and Astra advice. Parent-turn runtime sandbox overrides can constrain the child, and Sol's no-edit boundary remains instructional during workspace-write turns. Astra's metadata test does not prove a rejected write or rule out separately approved escalation.
- Local model catalogs list the GPT-6 model IDs, but that does not prove the configured provider route or account can serve them. The trial must verify actual calls.
- The GPT-6 promotion was explicitly approved after both client trials. The GPT-5.6 selector values and preserved agents document rollback; the GPT-6 trial profile and agent files document known-good GPT-6 definitions.
