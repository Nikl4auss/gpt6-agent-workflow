# 04: Report trial results and gate promotion

**What to build:** The user can compare the OpenCode and Codex GPT-6 trials and decide whether to promote GPT-6 to either client's defaults, based on observed routing, implementation, validation, review, and Astra behavior.

**Blocked by:** 02: Prove the GPT-6 workflow in OpenCode; 03: Prove the GPT-6 workflow in Codex.

**Status:** ready-for-human

- [x] The results record actual model routing, the representative change, validation run, Sol's independent review verdict, Astra consultation behavior, failures, and residual risks for each client.
- [x] Unavailable models or fallback routing are called out and do not count as successful trial results.
- [x] GPT-5.6 remains the default unless both client checks pass and the user explicitly approves promotion.
- [x] The results present promotion as a decision for the user; defaults are not changed without that approval.

## Comparison

- **OpenCode:** GPT-6 Sol routed to GPT-6 Luna and GPT-6 Astra through the corrected Sol-primary flow. Luna made the representative one-line change and passed `git diff --check`; Sol independently returned `ACCEPT`. Astra advised read-only on the consequential default-agent question; Sol retained the decision. The earlier top-level Luna invocation fell back and is not counted; the unavailable-model probe returned HTTP 400.
- **Codex:** A workspace-write Luna session and a separate read-only Sol/Astra session used the requested models and recorded rollout metadata. Luna appended `Luna implementation probe complete` to the disposable fixture's `probe.txt`; `git diff --check` exited 0; Sol independently returned `ACCEPT`. Astra advised only and Sol made the final decision. A transient websocket HTTP 503 occurred, but the session ended `turn.completed`; the unavailable-model probe returned HTTP 400 with no fallback and was not counted as a successful routing check. Metadata is permission-profile evidence, not proof of rejected-write enforcement or impossible separately authorized escalation. The fixture and its captured local JSONL outputs were removed after capture; persisted Codex rollouts remain under `~/.codex/sessions/`.
- GPT-5.6 remains the default. Promotion is not performed by this ticket and requires explicit user approval after considering both client results.
