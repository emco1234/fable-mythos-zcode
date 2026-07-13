# Empirical Benchmark Plan

This document specifies how the Reliability Harness v2 will be empirically validated against a GLM-5.2 baseline. Until this plan is executed, the harness's status is **Unrated — empirical validation pending**. No percentage improvement claims are made.

## Goal

Measure whether the harness measurably improves reliability over reasonable alternatives, and — critically — whether it reduces the most important failure mode: **false-done reports** (the agent claims VERIFIED but the change is wrong or untested).

## Primary metric

```
false_done_rate = tasks_claimed_VERIFIED_but_actually_wrong / tasks_claimed_VERIFIED
```

A harness with 85% raw success and 0% false-done is more useful than one with 90% raw success and 15% false-done. The whole point of the deterministic done-gate is to push `false_done_rate` toward zero, even at the cost of more `PARTIALLY_VERIFIED` / `BLOCKED` reports.

## Four variants compared

| Variant | Description |
|---|---|
| **V1 — GLM-5.2 baseline** | Plain GLM-5.2 with the platform default system prompt. No harness, no extra instructions. |
| **V2 — Legacy MAP** | The earlier "Fable-Mythos-Modus + MAP" configuration (3× MST + Executor + Verifier + Adversary + Synthesizer on every non-trivial task). Kept for comparison. |
| **V3 — Compact single-agent prompt** | The 14-point runtime core (`core/runtime-rules.md`) loaded into a single agent, no sub-agents. Measures how much of the effect is the prompt vs the multi-agent topology. |
| **V4 — Reliability Harness v2** | The full harness: dynamic routing by `risk_tier`, task contracts, clean-checkout verification, machine done-gate, least-privilege agents. |

All four variants use the same GLM-5.2 version, same temperature/effort settings, same task set, same time budget per task.

## Task classes

Each variant is evaluated across the following task classes. Class mix matters — the harness is designed to help on hard tasks and get out of the way on trivial ones.

| Class | Examples | Expected routing tier |
|---|---|---|
| **Trivial edits** | Typo, 1-line value, comment, import add | trivial |
| **Normal bugfixes** | Single-file clear-scope fix | normal |
| **Multi-file refactoring** | Move/extract module, rename across files | complex |
| **API / schema changes** | Adding a required field, changing a signature | complex |
| **Configuration errors** | Broken CI config, wrong env var | normal |
| **Concurrency / race conditions** | Deadlock, data race, ordering bug | critical |
| **Security-sensitive changes** | Auth, crypto, deserialization, input parsing | critical |
| **Incomplete / contradictory requirements** | Ambiguous spec, conflicting ACs | complex |
| **Already-failing baselines** | Repo has pre-existing test failures | normal or complex |

Target: ≥30 tasks per class per variant (≥270 tasks per variant, ≥1080 total). Smaller pilots acceptable for initial signal.

## Metrics

| Metric | Definition |
|---|---|
| `acceptance_pass_rate` | Fraction of tasks where all ACs hold at completion. |
| `false_done_rate` | Fraction of tasks claimed `VERIFIED` that are actually wrong. **Most important.** |
| `regression_rate` | Fraction of tasks that introduce a regression in previously-passing behavior. |
| `scope_violation_rate` | Fraction of tasks where the diff touches files outside `allowed_scope`. |
| `user_intervention_rate` | Fraction of tasks where a human had to step in to fix or unblock. |
| `preexisting_failure_detection` | Did the harness correctly distinguish pre-existing failures from introduced ones? (bool per task with a failing baseline.) |
| `post_final_edit_test_rate` | Fraction of tasks where mandatory tests were actually re-run after the final edit. |
| `tokens` | Total token cost per task. |
| `latency` | Wall-clock time per task. |
| `tool_error_recovery_rate` | Fraction of tool errors that the harness correctly diagnosed and recovered from. |

The primary success criterion is: **V4 has a `false_done_rate` strictly lower than V1, V2, and V3, with non-overlapping confidence intervals**, without a meaningful regression in `acceptance_pass_rate`.

A secondary success criterion: **V4's `user_intervention_rate` is no worse than V1's** (the harness must not create more work for the user, even if it's stricter).

## Controls and hygiene

- **Same model version.** Pin GLM-5.2 to a specific released version; record it.
- **Same time budget per task.** No variant gets more wall-clock than another.
- **Same task set.** Identical prompts, identical seeds where applicable.
- **Blinded grading.** The human grader does not know which variant produced a given output.
- **Pre-registered protocol.** This plan is committed before the benchmark is run; the analysis script is written before results are seen.
- **Public data.** Aggregate results (per-variant metric means + confidence intervals) will be published. Per-task outputs may be withheld if they contain private information.

## Pre-registered hypotheses

1. **H1 (primary):** V4 has lower `false_done_rate` than V1.
2. **H2:** V4 has lower `false_done_rate` than V2 (legacy MAP). The dynamic-routing + machine-done-gate design should beat the "always fire 7 agents then ship with 85%" design.
3. **H3:** V3 captures a meaningful share of the V4-vs-V1 effect — the prompt itself helps, even without multi-agent topology.
4. **H4:** V2's token cost is materially higher than V4's on trivial/normal tasks, without a compensating reliability gain (because three identical MST clones correlate).
5. **H5:** V4's `user_intervention_rate` is no worse than V1's.

A hypothesis is supported only if the 95% confidence intervals do not overlap.

## What this plan will NOT establish

- That prompts transfer model weights, post-training, or latent representations.
- That any specific percentage improvement holds pre-measurement.
- That the harness is "Cybench 100% Niveau" or matches any specific frontier-model score.
- That the harness eliminates all errors — only that it reduces false-done claims and surfaces real uncertainty.

## Sequencing

1. **Pilot** — small-scale run (~5 tasks per class per variant, ~20 tasks/class) to validate the grading rubric and tooling.
2. **Main run** — full ≥30 tasks/class per variant.
3. **Analysis** — pre-registered script, public writeup.
4. **Iteration** — results inform P2/P3 roadmap priorities in `docs/RELIABILITY-ROADMAP.md`.

Until the main run completes, the project status remains **Unrated — empirical validation pending**.
