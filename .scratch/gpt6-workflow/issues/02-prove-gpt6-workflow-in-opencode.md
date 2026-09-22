# 02: Prove the GPT-6 workflow in OpenCode

**What to build:** A separate, opt-in OpenCode entry point runs the shared workflow with GPT-6 Sol, Luna, and Astra while the existing GPT-5.6 default remains available. A client-level trial demonstrates implementation, independent review, and targeted advice.

**Blocked by:** 01: Extend the shared workflow with Astra escalation.

**Status:** ready-for-agent

- [x] The opt-in Sol, Luna, and Astra roles use their intended GPT-6 models, with Sol read-only, Luna able to implement, and Astra read-only.
- [x] The existing GPT-5.6 default and agent definitions remain usable and unchanged during the trial.
- [x] A representative project change demonstrates Luna editing and running focused validation, followed by an independent Sol review verdict.
- [x] Routine work is handled without a worker handoff, and a targeted difficult question reaches Astra as a read-only adviser. Sol states the escalation reason and makes the final decision.
- [x] The trial confirms actual model routing. An unavailable model is reported, and a fallback does not count as a successful check.

## Comments

- Definitions: `.opencode/agents/gpt6-{sol,luna,astra}-trial.md` pin `openai/gpt-6-{sol,luna,astra}`; Sol/Astra deny edits, Luna allows edits and is instructed to read repository `AGENTS.md`. `opencode agent list` showed all three after additive global installation. Existing `opencode.jsonc` still has `openai/gpt-5.6-sol` and `sol-orchestrator`; existing GPT5 agent files were not changed.
- Corrected implementation trial: `opencode run --dir /home/nikl4auss/orca/workspaces/gpt6-agent-workflow/ticket-02-opencode --agent gpt6-sol-trial --model openai/gpt-6-sol --format json` ran the Sol primary. `opencode export ses_f34cb21c9ffeDbVP5OVi7mPahB` exposed parent `agent: gpt6-sol-trial`, `model.id: gpt-6-sol`, and the task response `model.modelID: gpt-6-luna`. Sol called the allowed `gpt6-luna-trial` task, awaited it, and independently reviewed its change. Luna changed only the third checkbox, ran `git diff --check` successfully, and Sol returned `ACCEPT`. No fallback was counted.
- Corrected Astra trial used the same Sol-primary syntax and required the allowed `gpt6-astra-trial` task, not a top-level Astra invocation. `opencode export ses_f34ca3ed4ffex412yGMft9k9sC` exposed parent `model.id: gpt-6-sol` and the task response `model.modelID: gpt-6-astra`. Sol stated escalation was warranted because changing `default_agent` would affect all sessions. Astra made no edits and advised not changing `default_agent`; Sol made the same final decision. Existing GPT-5.6 defaults remain unchanged.
- The earlier incorrect top-level `--agent gpt6-luna-trial` attempt is retained as a non-successful probe: OpenCode warned it was a subagent and fell back to the default primary. The corrected Sol-primary trial above supersedes it and is the evidence for criterion 3.
- Follow-up project-file trial: GPT-6 Sol delegated a one-line update to `.opencode/agents/gpt6-luna-trial.md`; GPT-6 Luna added the repository-instructions directive, `opencode agent list` and `git diff --check` passed, and Sol independently inspected the diff and returned `ACCEPT`.
