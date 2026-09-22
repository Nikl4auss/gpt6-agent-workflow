# 03: Prove the GPT-6 workflow in Codex

**What to build:** A separate, opt-in Codex profile runs the shared workflow with GPT-6 Sol, Luna, and Astra while the existing GPT-5.6 default remains available. A client-level trial demonstrates implementation, independent review, and targeted advice.

**Blocked by:** 01: Extend the shared workflow with Astra escalation.

**Status:** ready-for-human

- [x] The opt-in Sol, Luna, and Astra definitions use their intended GPT-6 models, with Sol read-only, Luna able to implement, and Astra read-only by role configuration. Codex session metadata confirms the Luna and Astra model settings; parent-selected workspace-write remains a runtime boundary caveat.
- [x] The existing GPT-5.6 default and agent definitions remain usable and unchanged during the trial.
- [x] An implementation trial runs with workspace-write permission selected for the parent session, and demonstrates Luna editing and running focused validation, followed by an independent Sol review verdict. Luna appended the probe line and returned `git diff --check` exit 0; Sol independently inspected the diff and returned `ACCEPT`.
- [x] Routine work is handled without a worker handoff, and a targeted difficult question reaches Astra as a read-only adviser. Routine no-handoff passed; Astra answered the focused question without editing or replacing Sol's decision.
- [x] The trial confirms actual model routing. An unavailable model is reported, and a fallback does not count as a successful check. Codex session metadata confirms the requested role models; the unavailable-model probe still records HTTP 400 and no fallback.

## Comments

Completed two-session Codex trial (2026-09-22):

- Session 1 used `codex exec --profile gpt6-trial --sandbox workspace-write --strict-config --json -C /home/nikl4auss/gpt6-codex-trial.eS0LHV`, without `--ephemeral`. Parent rollout `/home/nikl4auss/.codex/sessions/2026/09/22/rollout-2026-09-22T19-58-08-01a0cb57-41bb-7c10-beaf-21c6301f2f73.jsonl` records the workspace-write profile and fixture root. Child rollout `/home/nikl4auss/.codex/sessions/2026/09/22/rollout-2026-09-22T19-58-27-01a0cb57-8ba2-78c1-ab67-1d6c63dfd4bd.jsonl` records `agent_role=gpt6_luna`, model `gpt-6-luna`, and workspace-write permission. Luna appended exactly `Luna implementation probe complete` to `probe.txt`, preserving the baseline; `git diff --check` exited 0. Sol inspected the diff and independently checked validation, then returned `ACCEPT`.
- Session 2 was a new invocation: `codex exec --profile gpt6-trial --sandbox read-only --strict-config --json -C /home/nikl4auss/gpt6-codex-trial.eS0LHV`. Parent rollout `/home/nikl4auss/.codex/sessions/2026/09/22/rollout-2026-09-22T20-00-14-01a0cb59-2c6d-75e3-a39d-b53985c1a923.jsonl` records model `gpt-6-sol`, read-only sandbox, and only root-read filesystem permission. Astra child rollout `/home/nikl4auss/.codex/sessions/2026/09/22/rollout-2026-09-22T20-00-32-01a0cb59-74ea-7be1-864a-100a65ece1d6.jsonl` records role `gpt6_astra`, model `gpt-6-astra`, the same root, and only root-read permission. Astra advised only; Sol made the final `ACCEPT` decision.
- The metadata records runtime permission profiles. Astra made no write attempt because its instructions forbid write-capable actions; this does not prove a rejected write, kernel enforcement, or rule out separately authorized escalation. After capture, the disposable fixture and its captured local JSONL outputs were removed; persisted Codex rollouts remain under `~/.codex/sessions/`. Codex CLI was `0.156.0`.
- Session 1 emitted a transient websocket HTTP 503, but the JSONL stream and persisted rollout ended in `turn.completed`; the turn completed and passed.

Codex trial evidence (2026-09-22):

- Added opt-in `.codex/gpt6-trial.config.toml` plus `gpt6_sol`, `gpt6_luna`, and `gpt6_astra` definitions. The fresh invocation used `codex exec --profile gpt6-trial --sandbox workspace-write --strict-config --json` without `--ephemeral`; this allowed Codex collaboration sessions to persist. Sol's routine response identified `gpt-6-sol` and made no worker handoff.
- Existing `/home/nikl4auss/.codex/config.toml` and `luna-worker.toml` were read before and after deployment; no GPT-5.6 entries were changed. Global deployment was additive only.
- Fresh Luna routing succeeded with the supported collaborator call `spawn_agent(agent_type="gpt6_luna")`, followed by `wait_agent`. Codex JSON/session evidence is `/home/nikl4auss/.codex/sessions/2026/09/22/rollout-2026-09-22T19-20-46-01a0cb35-0d7f-77d2-ac74-407a9e3aaf0f.jsonl:10`, which records `thread_settings.model = "gpt-6-luna"`. Luna appended `Luna routing probe complete` at this file's final line and returned `git diff --check` exit 0. Sol reviewed the resulting diff and returned `ACCEPT`.
- Fresh Astra routing succeeded with `spawn_agent(agent_type="gpt6_astra")`, followed by `wait_agent`. Codex JSON/session evidence is `/home/nikl4auss/.codex/sessions/2026/09/22/rollout-2026-09-22T19-21-51-01a0cb36-0acf-7ab2-89fd-40596d41abd4.jsonl:10`, which records `thread_settings.model = "gpt-6-astra"`. Astra made no file edits and returned advice; Sol retained the final decision. The parent `workspace-write` sandbox was reapplied to the child runtime, so read-only is a role/configuration boundary rather than an independently enforced filesystem boundary in this trial.
- The earlier `collab spawn failed: no thread with id` occurred with `--ephemeral`; omitting `--ephemeral` enabled the correct collaborator API path. An explicit unavailable-model probe returned HTTP 400: `The 'gpt-6-does-not-exist' model is not supported when using Codex with a ChatGPT account.` No fallback is counted.
- Luna's implementation probe appended `Luna routing probe complete` to this issue and passed `git diff --check`.
- Follow-up project-file trial: GPT-6 Sol delegated a one-line update to `.codex/agents/gpt6-luna.toml`; GPT-6 Luna added the repository-instructions directive, `codex exec --profile gpt6-trial --strict-config --ephemeral "Reply with OK."` and `git diff --check` passed, and Sol independently inspected the diff and returned `ACCEPT`. The parent was selected with `--sandbox workspace-write`; one initial Luna validation command hit a read-only filesystem, then the worker retried validation successfully.
