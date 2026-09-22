# 01: Extend the shared workflow with Astra escalation

**What to build:** Sol can use the existing Sol/Luna workflow with a read-only Astra adviser for unresolved, consequential decisions. Sol remains responsible for decisions and the final response, and routine work continues without unnecessary handoffs.

**Blocked by:** None (can start immediately).

**Status:** ready-for-agent

- [ ] Sol consults Astra only for unresolved consequential design or security/data-loss decisions, repeated revisions that expose such a decision, or an explicit user request. Large diffs and failed checks alone do not trigger consultation.
- [ ] Sol gives Astra a specific question, relevant evidence, constraints, and the decision needed. Astra advises without editing or replacing Sol's decision.
- [ ] Sol states why it consulted Astra and retains responsibility for the final decision and response.
- [ ] The existing delegation packet, worker report, independent review verdicts, and limit of two unsuccessful revision rounds remain usable.
- [ ] An unavailable requested model is reported explicitly; the workflow does not silently treat a fallback as success.
