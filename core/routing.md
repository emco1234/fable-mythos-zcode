# Dynamic Routing — Reliability Harness v2

Routing is **dynamic by `risk_tier`**, not a fixed 7-invocation pipeline on every non-trivial task. The earlier "approximately 4× overhead" claim was wrong; ZCode Custom Subagents run in the foreground (blocking), so a fixed pipeline costs at least 7 invocations per task plus 4 per repair round. Running that on every normal change is wasteful and produces correlated pseudo-explanations.

## Classification (main agent assigns `risk_tier` at task start)

The main agent reads the task, the task contract, and the repository state, then assigns one of:

| `risk_tier` | Trigger signals |
|---|---|
| **trivial** | Typo, 1-line value change, comment, import add, CSS color literal swap. No logic / behavior / architecture branch touched. |
| **normal** | Clear-scope bugfix, single-file refactor, well-specified small feature. No concurrency, no security sensitivity, no schema/API change. |
| **complex** | Multi-file change, API/schema change, unclear or ambiguous spec, refactoring across module boundaries, long-horizon task. |
| **critical** | Security-sensitive (auth, crypto, deserialization, input parsing, privilege boundary), concurrency/race-prone, data-loss risk, infra/CI mutation, or anything where a regression is high-blast-radius. |

**In doubt → upgrade.** If "normal or complex?" → complex. If "complex or critical?" → critical.

## Routing Table

| `risk_tier` | Phase 0 Investigation | Phase 3 Implementation | Phase 4 Verification | Adversary | Sub-agent count |
|---|---|---|---|---|---|
| **trivial** | none (main agent reads file) | main agent | none | none | 0 |
| **normal** | built-in `explore` subagent if needed | main agent with self-tests | 1 `reliability-verifier` on clean checkout | none | 1 |
| **complex** | `reliability-scout` + `reliability-spec-critic` parallel (read-only); optionally built-in `explore` | `reliability-lead` with mandatory self-tests | `reliability-verifier` on clean checkout + `reliability-test-designer` for regression cases | none | ~4 |
| **critical** | As complex + `reliability-test-designer` early | `reliability-lead` with self-tests | `reliability-verifier` on clean checkout | `reliability-adversary` in isolated worktree | ~6 |

## Rules

1. **No three identical thinking agents.** Three MST clones with same model + same prompt + same context produce stylistic variants of the same assumption. Use orthogonal roles instead: codebase (`reliability-scout`), spec (`reliability-spec-critic`), verification (`reliability-test-designer`).
2. **Prefer the built-in `explore` subagent** for architecture/call-chain/file-search investigation. It is purpose-built and cheaper than a freely-formulating thinking agent.
3. **Self-tests are mandatory at `normal` and above.** The implementer never delegates all verification.
4. **Clean-checkout verification is mandatory at `normal` and above.** The verifier creates a fresh worktree from `base_commit`, applies the frozen patch, and re-runs the 9-point check. Verification on a dirty/shared worktree is invalid.
5. **Adversary only at `critical`.** At `normal`/`complex` the verifier is sufficient. Spamming the adversary on every task wastes tokens and produces correlated findings.
6. **Repair loops:** Each repair round re-runs Phase 3 self-tests + Phase 4 clean-checkout verification. Maximum 3 rounds, then escalate to user with STATUS=`BLOCKED`.
7. **Trivial override:** trivial tasks never dispatch sub-agents. If a task looked trivial but turns out non-trivial mid-flight, the main agent re-classifies and re-routes.

## Cost expectations (foreground / blocking model)

- trivial: ~1 model call total (main agent).
- normal: main agent + 1 verifier = ~2-3 calls + 1 repair round if needed.
- complex: ~4-6 calls + 4 per repair round.
- critical: ~6-8 calls + 4 per repair round.

These are honest estimates for ZCode's current foreground-subagent model. When ZCode adds async subagents (see `docs/RELIABILITY-ROADMAP.md`), parallel investigation costs will drop further.

## Override (user-specified)

The user may force a `risk_tier` in the task contract (`risk_tier: critical`). The router honors the user's choice even if classification would have picked a lower tier.
