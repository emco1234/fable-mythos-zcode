# Reliability Roadmap (P2 / P3)

This document tracks the post-v2 work: turning the configuration-only Reliability Harness into a technically-enforced harness with long-lived agents, persistent task memory, async subagents, programmatic tools, property-based testing, fuzzing, mutation testing, differential tests, an optional second model, and telemetry.

Status as of v2: **configuration only**. `AGENTS.md`, `SKILL.md`, and the sub-agent prompts structure behavior — they do not yet enforce a deterministic DAG. The schemas (`task-contract.schema.json`, `verification-report.schema.json`), the evidence-ledger format, and the routing rules are specifications that the model follows best-effort.

## P2 — Real enforcement and long-horizon agentik

### 1. Long-lived agents instead of restart-per-subtask

Today each subagent invocation is a fresh context. Long-horizon tasks lose state across the boundary. P2 introduces long-lived agent processes that retain working memory across multiple subtasks within a session.

- **Approach:** ZCode plugin that manages agent lifetimes explicitly (start, message, retain, end) instead of one-shot dispatch.
- **Effect:** the scout's findings stay alive when the lead starts editing; the verifier remembers the baseline without re-reading it.

### 2. Persistent structured task memory

The Evidence Ledger (`core/evidence-ledger.md`) is currently a Markdown spec. P2 persists it as actual structured storage (JSON or SQLite) keyed by `task_id`, queryable by any agent.

- **Schema-backed:** instances of `task-contract.schema.json` and `verification-report.schema.json`.
- **Survives compaction:** the ledger is re-read at every gate; compaction never drops it.

### 3. Context compaction that never loses Task Contract / Evidence Ledger

A compaction strategy that explicitly preserves:
- the active Task Contract,
- the current Evidence Ledger entries,
- the open `blocking_unknowns` list,
- the latest verification-report status.

Everything else may be summarized. The Mythos System Card documents a case where dense prose compressed away the load-bearing detail — this rule prevents that.

### 4. Async subagents (when the platform allows)

ZCode Custom Subagents currently run in the foreground (blocking). The dynamic routing table in `core/routing.md` is designed for that model. When ZCode adds async subagents:
- parallelize the two read-only scouts (`reliability-scout` + `reliability-spec-critic`),
- run the test-designer concurrently with the lead's first implementation pass,
- run the adversary concurrently with the verifier on critical tasks.

This drops the critical-tier latency from sequential ~6 rounds to ~3-4 effective rounds.

### 5. Programmatic tool calling / custom reliability tools

Replace prompt-only enforcement with actual tools:
- `write_task_contract` — validates against the JSON Schema before writing.
- `record_evidence` — appends to the persistent ledger with required fields.
- `freeze_patch` — produces a content-addressed patch the verifier can apply.
- `verify_clean_checkout` — runs the 9-point check on a fresh worktree and emits a machine-readable report.
- `evaluate_done_gate` — programmatically evaluates the done-gate rules; refuses to emit `VERIFIED` if any rule fails.

### 6. Structured repair loops

Today repair is "verifier reports findings → lead reworks → verifier re-runs". P2 makes repair structured: findings are severity-ranked, deduplicated, and turned into a sub-task-contract per cluster. The lead works the cluster; the verifier re-checks only the affected ACs. Maximum 3 rounds, then escalate to user with STATUS=`BLOCKED`.

### 7. Language/framework-specific check modules

Plugin-style modules that know a stack's idioms:
- TypeScript: `tsc --noEmit`, `eslint`, `vitest`/`jest`, `tsdoc`.
- Rust: `cargo check`, `clippy`, `cargo test`, `cargo miri run` for unsafe.
- Python: `mypy`/`pyright`, `ruff`, `pytest`, optional `hypothesis`.
- Go: `go vet`, `golangci-lint`, `go test -race`.

Each module auto-discovers the relevant commands from the repo (CI config, `package.json`, `Cargo.toml`, etc.) rather than hardcoding them in the prompt.

## P3 — Frontier-level verification

### 1. Property-based testing

Where invariants are expressible (parsers, state machines, algebraic identities), add PBT. Tooling: `hypothesis` (Python), `fast-check` (TS/JS), `proptest` (Rust), `go-quickcheck` (Go). The `reliability-test-designer` writes property tests as part of its regression suite.

### 2. Fuzzing and sanitizers

For security-sensitive or input-parsing code:
- Rust: `cargo-fuzz` + `miri` + AddressSanitizer.
- C/C++: libFuzzer, AFL++, ASan/UBSan/MSan.
- Go: native fuzzing (`go test -fuzz`).
- Python: `atheris` + `python -X dev`.
- JS/TS: `jsfuzz` / `fast-check` property-mode.

The `reliability-adversary` (critical tier) invokes the relevant fuzzer on the patched code in its isolated worktree.

### 3. Mutation testing for critical logic

Mutation testing measures test quality: does the suite catch small code mutations? If mutants survive, the tests have gaps even if coverage looks high. Tools: `mutmut` (Python), `stryker` (JS/TS), `cargo-mutants` (Rust). Run on critical modules after a successful verify.

### 4. Differential tests

Compare the patched version against:
- the previous release (regression-differential),
- a reference implementation if one exists,
- a simpler (possibly slower) specification implementation.

Differential mismatches are HIGH findings.

### 5. Optional second model / static analyzer as independent reviewer

Cross-model verification reduces correlated blind spots (all GLM-5.2 instances share systematic errors). Options:
- run the verifier with a different model family (a non-GLM-5.2 coding model) for cross-check,
- integrate a static analyzer (`semgrep`, `codeql`, `tideways`) as a non-LLM verifier,
- mix the two: LLM verifier for semantic findings, static analyzer for rule-based findings.

### 6. Telemetry on real failure modes

Collect (anonymized, opt-in) telemetry on:
- which gates most often block,
- which AC types most often fail,
- repair-round counts before VERIFIED,
- `false_done_rate` (the most important metric — see `docs/EMPIRICAL-BENCHMARK-PLAN.md`),
- scope-violation rate,
- token cost per `risk_tier`.

Use telemetry to continuously ablate prompt and harness components. A rule that never fires is a candidate for removal; a rule that fires often but never blocks is a candidate for sharpening.

## Sequencing

- **P2.1/P2.2/P2.3** (persistent ledger + compaction-safe memory) are the foundation — do them first. Without persistent memory, long-horizon tasks keep losing state.
- **P2.5** (programmatic tools) is the enforcement backbone — schemas become real validators.
- **P2.4** (async subagents) depends on platform support; track ZCode's roadmap.
- **P3** items are additive — each can be added independently once the P2 foundation is in place.

## Out of scope for this roadmap

- Model training, fine-tuning, or weight modification.
- Bypassing platform safety measures.
- Reproducing any specific frontier model's weights or latent representations.
- Any claim of "100% accuracy" or specific percentage improvements pre-measurement.
