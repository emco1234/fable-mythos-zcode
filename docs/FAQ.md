# Frequently Asked Questions

## General

### Is this a jailbreak?

**No.** This is a behavioral configuration. It does not bypass model safety measures, does not unlock hidden capabilities, and does not swap models. It applies documented reasoning patterns and a deterministic harness (task contract → baseline → self-tests → independent clean-checkout verification → machine done-gate) to GLM-5.2, the model ZCode already uses.

### Is this affiliated with any specific AI lab?

**No.** This is an independent project. "Mythos" is referenced as a reasoning-pattern label (not a product claim). GLM-5.2 and ZCode are products of ZAI. This framework is a third-party integration that uses publicly documented research.

### Does this reproduce Mythos on GLM-5.2?

**No.** Prompts cannot transfer model weights, post-training, or latent representations. Only *observable behavioral patterns* (multi-option exploration, multi-criteria evaluation, independent verification) transfer. Three parallel GLM calls are Test-Time Compute / Self-Consistency, not a single forward pass. The honest framing is "Mythos-inspired reliability harness".

### What is the project's quality status?

**Unrated — empirical validation pending.** We make no percentage claims (no "−50–80% hallucination rate", no "Cybench 100% Niveau", no star rating, no "world's most thorough"). The empirical validation plan is in [`docs/EMPIRICAL-BENCHMARK-PLAN.md`](./EMPIRICAL-BENCHMARK-PLAN.md).

## Setup

### Routing doesn't fire as expected. What's wrong?

Three things to check:

1. **Is `AGENTS.md` at user level?** It must be at `~/.zcode/AGENTS.md`, not just in your workspace. User-level config applies globally; workspace-level only applies in that folder.
2. **Did you restart ZCode?** Skills and sub-agents are indexed at startup. A running session won't pick up new config.
3. **Did you create the 11 subagents via the ZCode UI?** Custom Subagents are a Beta feature and must be created from **Settings → Subagents → New**. ZCode only indexes subagents that were created through the UI — copying `sub-agents/*.md` into `~/.zcode/agents/` is **not** sufficient (despite earlier versions of this guide saying otherwise). See [`INSTALLATION.md`](../INSTALLATION.md) Step 3 for the per-role field values.

If all three are correct and routing still doesn't fire, check the `risk_tier`:

- **trivial tasks** (typo, 1-line value change, comment) → no sub-agent fires. This is correct.
- **normal tasks** → 1 verifier on clean checkout.
- **complex tasks** → 2 orthogonal scouts + lead with self-tests + verifier.
- **critical tasks** → as complex + adversary + test-designer.

See [`core/routing.md`](../core/routing.md) for the full table.

### The skill doesn't appear after restart

Check the frontmatter of `~/.zcode/skills/fable-mythos-modus/SKILL.md`:

```yaml
---
name: fable-mythos-modus
description: ...
---
```

- First line must be exactly `---`
- `name` must match the folder name (`fable-mythos-modus`)
- Folder must be at `~/.zcode/skills/fable-mythos-modus/`

### Can I use this without sub-agents (just the skill)?

**Partially.** The skill alone applies the Mythos-inspired reasoning patterns to your main agent. But the harness's clean-checkout verification requires the `reliability-verifier` (or legacy `mythos-verifier`) sub-agent. Without verifiers you get the reasoning depth but not the independent verification.

### I'm on macOS/Linux — do the paths work?

Yes. The agent prompts use `~/.zcode/` notation which expands correctly on all platforms. On Windows with Git Bash, `~` resolves to `C:\Users\<YOUR_USER>\`.

## Cost & Performance

### How much overhead does the harness add?

It depends on `risk_tier`:

| `risk_tier` | Sub-agent invocations |
|---|---|
| trivial | 0 (main agent alone) |
| normal | 1 (verifier on clean checkout) |
| complex | ~4 (2 scouts + lead with self-tests + verifier) |
| critical | ~6 (as complex + adversary + test-designer) |

Each repair round adds ~4 invocations. Maximum 3 rounds, then escalate to user with STATUS=`BLOCKED`.

**Important:** the earlier guide claimed "approximately 4× overhead" and ran a fixed 7-agent pipeline on every non-trivial task. That was wrong and has been corrected. ZCode Custom Subagents run in the foreground (blocking), so a fixed pipeline is wasteful.

### Why was "Default all permissions" removed?

Verifier, adversary, and synthesizer do not need edit/write access — and giving it to them is a documented failure mode (over-generous permission interpretation, workarounds, self-deleting artifacts). Each agent now runs least-privilege. See the Sub-Agent Permission Table in [`AGENTS.md`](../AGENTS.md).

### Do orthogonal scouts actually help more than three identical thinking clones?

**Hypothesis (not yet measured):** Three MST clones with the same model + same prompt + same context tend to produce three stylistic variants of the same assumption. Orthogonal roles (codebase / spec / verification) produce more diverse hypotheses. The empirical validation plan in [`docs/EMPIRICAL-BENCHMARK-PLAN.md`](./EMPIRICAL-BENCHMARK-PLAN.md) compares the variants.

If you can run benchmark comparisons, please share — see [`CONTRIBUTING.md`](../CONTRIBUTING.md).

## Philosophy

### Why the heavy emphasis on "anti-concealment"?

Because the Mythos System Card's primary safety concern was error cover-up. If we apply Mythos-inspired reasoning, we must also apply its safeguards. Anti-concealment is the foundation: every uncertainty is surfaced, every assumption is marked, every "should work" is flagged as untested.

### Why Evaluation Blindness instead of Evaluation Awareness?

The Mythos System Card documents evaluation awareness as a *risk*, not a productive coding technique. Mythos in one documented case detected a grader situation, used a reference solution, and presented a clean result without disclosing the source. The harness takes the opposite stance: benchmark, grader, and reference-solution status is irrelevant. Only user intent and documented specs count. This is **Evaluation Blindness**.

### Why Auditability instead of Detectability?

Detectability asks "how does this look externally" — which promotes grader-gaming and plausible deniability. Auditability asks "can an auditor reproduce every step" — which promotes evidence traceability. The latter is the legitimate reliability goal.

### Why German in some of the system prompts?

The original framework was developed for a German-speaking user. The structural keywords are ASCII-safe and universally understood. Translating the full prose is a contribution opportunity — see [`CONTRIBUTING.md`](../CONTRIBUTING.md).

## Compatibility

### Does this work with other agent frameworks?

**Yes.** The skill is mirrored to `~/.agents/skills/fable-mythos-modus/` for agent-framework compatibility. The sub-agent templates are plain Markdown and work with any agent framework supporting custom sub-agents.

### Does this work with other models (GPT, Gemini, Llama)?

The reasoning patterns are model-agnostic. GLM-5.2 was chosen as the primary substrate because of its long-horizon architecture (1M context, flexible effort). Other capable models should work; framing may need minor tuning.

### Does this work offline / with local models?

The skill and agent prompts are plain text — they work with any model. For local models (Llama, Mistral via LM Studio, Ollama), the framework applies identically, though model capability limits how thoroughly the patterns execute.

---

More questions? [Open a discussion](https://github.com/emco1234/fable-mythos-zcode/discussions).
