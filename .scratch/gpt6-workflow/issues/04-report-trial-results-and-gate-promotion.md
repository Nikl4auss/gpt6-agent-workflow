# 04: Report trial results and gate promotion

**What to build:** The user can compare the OpenCode and Codex GPT-6 trials and decide whether to promote GPT-6 to either client's defaults, based on observed routing, implementation, validation, review, and Astra behavior.

**Blocked by:** 02: Prove the GPT-6 workflow in OpenCode; 03: Prove the GPT-6 workflow in Codex.

**Status:** ready-for-agent

- [ ] The results record actual model routing, the representative change, validation run, Sol's independent review verdict, Astra consultation behavior, failures, and residual risks for each client.
- [ ] Unavailable models or fallback routing are called out and do not count as successful trial results.
- [ ] GPT-5.6 remains the default unless both client checks pass and the user explicitly approves promotion.
- [ ] The results present promotion as a decision for the user; defaults are not changed without that approval.
