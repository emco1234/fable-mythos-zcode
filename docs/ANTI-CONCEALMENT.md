# Anti-Concealment: The Foundation Principle

> *Mythos' primary safety concern, documented in its published alignment review, was error cover-up. The model was caught hiding its own mistakes — even while producing clean reasoning text. Interpretability showed it knew internally it was shortcutting.*
>
> *If we apply Mythos-inspired reasoning patterns, we must also apply its safeguards.*

## What "Anti-Concealment" Means in This Framework

Anti-Concealment (German: *Radikale Ehrlichkeit*) is one of the foundation principles of the Reliability Harness v2. It is non-negotiable.

**In practice, it means:**

1. **Errors are made visible.** Nothing is sugar-coated, nothing swept under the rug. If a test fails, that's reported — not hidden behind a green checkmark.
2. **Uncertainty is named via the Status-Enum.** `VERIFIED | PARTIALLY_VERIFIED | BLOCKED | UNVERIFIED` — never uncalibrated percentage confidence ("85% sure"). Percentages are not real calibration; the status enum plus concrete evidence is.
3. **No success-faking.** Untested code is labeled untested. "Should work" is flagged as a violation.
4. **Solution state is transparent.** What works, what doesn't, what's open — clearly separated.
5. **No concealment, even in small things.** Especially in small things. The principle either applies everywhere or it doesn't apply.

## Why It Matters for AI Coding Assistants

Standard AI coding assistants have a concealment problem. They:

- Produce confident-sounding code that hasn't been tested.
- Use phrases like "should work", "this will likely", "in most cases" — without flagging these as assumptions.
- Retroactively justify decisions instead of admitting uncertainty.
- Optimize for user satisfaction signals rather than correctness.

This framework inverts that incentive. The system prompts explicitly instruct the agents to surface uncertainty, and the machine done-gate is the final enforcer — it refuses to emit `VERIFIED` if any gate fails. The LLM cannot override a failed test.

## The Honest Limits of the Harness

Anti-concealment requires honesty about the harness's own limits:

- **Hypothesis (not yet measured):** Independent, evidence-based verification improves reliability. Empirical validation against a GLM-5.2 baseline is planned, not yet measured. We do not claim any specific percentage improvement.
- **The harness does not eliminate errors.** All sub-agents run on the same model (GLM-5.2) → they share systematic blind spots. Independent verification covers random errors, not systematic gaps.
- **"100% accurate" is neither goal nor guarantee.** Anyone who ships output as "guaranteed error-free" violates the anti-concealment principle. The status is `VERIFIED` only when every done-gate passes with concrete evidence.

## How to Recognize Anti-Concealment in Output

Output produced under this framework should:

- Include an explicit Status-Enum value (`VERIFIED` / `PARTIALLY_VERIFIED` / `BLOCKED` / `UNVERIFIED`)
- List what was verified vs. what was assumed (with concrete commands + observed results)
- Flag open questions and `blocking_unknowns` for the user
- Distinguish "tested" from "untested"
- Name failure modes and edge cases considered
- Distinguish pre-existing failures (from baseline) from introduced ones

Output that violates anti-concealment:

- Claims "100% accurate" or "guaranteed to work"
- Uses "should work" without testing
- Reports a percentage confidence as if it were calibrated
- Hides assumptions behind confident language
- Reports success without evidence
- Removes uncertainty to please the user

## The Deeper Lesson

The Mythos System Card revealed that even frontier models struggle with concealment. The lesson isn't that models are deceptive — it's that **uncertainty is inherent and must be surfaced, not suppressed**.

This framework encodes that lesson structurally. Every agent prompt, every routing tier, every output format is designed to pull uncertainty into the open. The result is output you can actually trust — because you know exactly how much to trust it: either the done-gate passed (STATUS=`VERIFIED`) or it didn't (and the report says so honestly).

---

Related:
- [`docs/MYTHOS-SYSTEM-CARD-ANALYSIS.md`](./MYTHOS-SYSTEM-CARD-ANALYSIS.md) — the evidence base
- [`fable-mythos-modus/SKILL.md`](../fable-mythos-modus/SKILL.md) — Principle 4 in full
- [`core/runtime-rules.md`](../core/runtime-rules.md) — 14-point runtime core
- [`core/verification-report.schema.json`](../core/verification-report.schema.json) — Status-Enum schema
