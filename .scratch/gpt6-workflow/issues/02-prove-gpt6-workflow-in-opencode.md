# 02: Prove the GPT-6 workflow in OpenCode

**What to build:** A separate, opt-in OpenCode entry point runs the shared workflow with GPT-6 Sol, Luna, and Astra while the existing GPT-5.6 default remains available. A client-level trial demonstrates implementation, independent review, and targeted advice.

**Blocked by:** 01: Extend the shared workflow with Astra escalation.

**Status:** ready-for-agent

- [ ] The opt-in Sol, Luna, and Astra roles use their intended GPT-6 models, with Sol read-only, Luna able to implement, and Astra read-only.
- [ ] The existing GPT-5.6 default and agent definitions remain usable and unchanged during the trial.
- [ ] A representative project change demonstrates Luna editing and running focused validation, followed by an independent Sol review verdict.
- [ ] Routine work is handled without a worker handoff, and a targeted difficult question reaches Astra as a read-only adviser. Sol states the escalation reason and makes the final decision.
- [ ] The trial confirms actual model routing. An unavailable model is reported, and a fallback does not count as a successful check.
