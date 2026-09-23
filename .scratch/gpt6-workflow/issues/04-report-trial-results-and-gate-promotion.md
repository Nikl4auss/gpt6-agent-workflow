# 04: Report trial results and gate promotion

**What to build:** The user can compare the OpenCode and Codex GPT-6 trials and decide whether to promote GPT-6 to either client's defaults, based on observed routing, implementation, validation, review, and Astra behavior.

**Blocked by:** 02: Prove the GPT-6 workflow in OpenCode; 03: Prove the GPT-6 workflow in Codex.

**Status:** ready-for-human

- [x] The results record actual model routing, the representative change, validation run, Sol's independent review verdict, Astra consultation behavior, failures, and residual risks for each client.
- [x] Unavailable models or fallback routing are called out and do not count as successful trial results.
- [x] Both client checks passed, and the user explicitly approved promotion (“Promote both”; “Do it”).
- [x] The applied OpenCode defaults are `model = openai/gpt-6-sol` and `default_agent = gpt6-sol-trial`.
- [x] The applied Codex defaults are `model = gpt-6-sol` and `default_subagent_model = gpt-6-luna`; developer instructions name `gpt6_luna` for implementation and `gpt6_astra` only for focused advice or explicit requests, with Sol retaining review and final decisions.

## Comparison

- **OpenCode:** GPT-6 Sol routed to GPT-6 Luna and GPT-6 Astra through the corrected Sol-primary flow. Luna made the representative one-line change and passed `git diff --check`; Sol independently returned `ACCEPT`. Astra advised read-only on the consequential default-agent question; Sol retained the decision. The earlier top-level Luna invocation fell back and is not counted; the unavailable-model probe returned HTTP 400.
- **Codex:** A workspace-write Luna session and a separate read-only Sol/Astra session used the requested models and recorded rollout metadata. Luna appended `Luna implementation probe complete` to the disposable fixture's `probe.txt`; `git diff --check` exited 0; Sol independently returned `ACCEPT`. Astra advised only and Sol made the final decision. A transient websocket HTTP 503 occurred, but the session ended `turn.completed`; the unavailable-model probe returned HTTP 400 with no fallback and was not counted as a successful routing check. Metadata is permission-profile evidence, not proof of rejected-write enforcement or impossible separately authorized escalation. The fixture and its captured local JSONL outputs were removed after capture; persisted Codex rollouts remain under `~/.codex/sessions/`.
- The user approved promotion after considering both client results, and this ticket applied both default changes. GPT-5.6 rollback files and the GPT-6 trial profile remain intact. The Codex Astra metadata caveat above remains: it is permission-profile evidence, not proof of rejected-write enforcement or impossible separately authorized escalation.
- Rollback restores the prior OpenCode and Codex selectors and `luna_worker` delegation documented in the workflow spec; keep Codex `sandbox_mode = read-only`.

## Post-promotion default routing verification

The post-promotion checks used no-edit intent and completed without fallback. OpenCode's defaults were `model = openai/gpt-6-sol` and `default_agent = gpt6-sol-trial`. Session `ses_f3463ec14ffecuE6ZhwJkMoBur` completed the no-edit invocation, and `opencode agent list` showed `gpt6-sol-trial` as the primary agent alongside the GPT-6 Luna and Astra agents. The displayed model and agent IDs corroborate the configured selectors; they are not treated as routing evidence by themselves.

Codex's defaults were `model = gpt-6-sol` and `default_subagent_model = gpt-6-luna`. The default no-edit invocation completed with `turn.completed` in read-only sandbox mode. Its rollout records show `gpt-6-sol` in `/home/nikl4auss/.codex/sessions/2026/09/22/rollout-2026-09-22T21-13-35-01a0cb9c-54b7-79b3-823b-7ad330ee593d.jsonl`, Luna (`gpt6_luna`, `gpt-6-luna`) in `/home/nikl4auss/.codex/sessions/2026/09/22/rollout-2026-09-22T21-13-54-01a0cb9c-9eac-7ab0-9465-5ab40a985bad.jsonl`, and Astra (`gpt6_astra`, `gpt-6-astra`) in `/home/nikl4auss/.codex/sessions/2026/09/22/rollout-2026-09-22T21-13-57-01a0cb9c-acd3-7191-8fe6-d7d7f63dd095.jsonl`. All three used read-only sandbox mode and returned successfully. The earlier usage-limit attempt was superseded by this successful retry.
