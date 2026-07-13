# Evidence Ledger — Reliability Harness v2

The Evidence Ledger is a **persistent, structured** record of every observed fact, executed command, and decision made during a task. It is the single source of truth that survives context compaction.

## Why a ledger?

Context compaction during long tasks silently drops load-bearing details. The Mythos System Card itself documents a case where dense prose compressed away the very detail that mattered. Agent-to-agent handoffs must therefore be **lossless and structured** — compression applies only to the final user report, never to internal handoffs.

The Evidence Ledger is never compressed away. It is re-read at every gate.

## Format (Markdown with embedded YAML blocks)

```markdown
# Evidence Ledger — <task_id>

## Task Contract Ref
- task_id: <id>
- base_commit: <sha>
- risk_tier: <trivial|normal|complex|critical>
- goal: <one-line>

## Baseline (Phase 2 — captured before any edit)
- command: `git status --porcelain`
  result: <verbatim or trimmed output>
- command: `git rev-parse HEAD`
  result: <base_commit>
- command: <baseline test command, e.g. `npm test`>
  exit_code: <int>
  preexisting_failures:
    - <failure 1>
    - <failure 2>
- reproduction:
  command: <repro command>
  result: <observed bug>

## Investigation (Phase 0/1 — scout/spec-critic/test-designer findings)
- scout_findings:
    affected_files: [...]
    call_graph: [...]
    conventions: [...]
    existing_tests: [...]
- spec_critic_findings:
    ambiguities: [...]
    scope_risks: [...]
- test_designer_findings:
    repro_test: <path or inline>
    regression_tests: [...]
    edge_cases: [...]

## Implementation (Phase 3 — executor/lead)
- patch:
    commit: <sha>
    files_changed: [...]
    lines_added: <int>
    lines_removed: <int>
- self_verification:
  - command: <test command 1>
    exit_code: <int>
    passed: <bool>
    summary: <observed result>
  - command: <test command 2>
    ...

## Independent Verification (Phase 4 — clean checkout)
- verifier: <agent name>
- worktree:
    fresh_checkout: true
    base_commit: <sha>
    patch_applied: <sha or hash>
- nine_point_check:
  - id: repro_test
    command: ...
    passed: <bool>
  - id: new_tests
    ...
  - id: affected_existing_tests
    ...
  - id: typecheck
    ...
  - id: lint
    ...
  - id: build
    ...
  - id: full_suite
    ...
  - id: diff_scope_audit
    ...
  - id: acceptance_criteria_audit
    ...
- scope_violations: []
- severity_findings:
  - severity: <CRITICAL|HIGH|MEDIUM|LOW>
    description: ...
    open: <bool>

## Adversary (Phase 4b — only risk_tier=critical)
- adversary: <agent name>
- findings: [...]

## Done-Gate (Phase 5 — machine-enforced)
- must_mapped_to_evidence: <bool>
- mandatory_tests_passed_after_final_edit: <bool>
- build_typecheck_lint_ok: <bool>
- new_or_changed_logic_tested: <bool>
- no_open_critical_high: <bool>
- no_scope_violations: <bool>
- no_unexecuted_tool_claims: <bool>
- preexisting_vs_introduced_separated: <bool>
- final_check_on_clean_checkout: <bool>
- STATUS: <VERIFIED|PARTIALLY_VERIFIED|BLOCKED|UNVERIFIED>
- residual_unknowns: [...]
```

## Rules

1. **Every claim cites a ledger entry.** No orphans. If it is not in the ledger, it did not happen.
2. **Commands appear verbatim** with their observed exit code and trimmed output. No paraphrases.
3. **Pre-existing failures are recorded at baseline**, never conflated with introduced failures.
4. **The ledger is re-read at every gate** — Phase 4 verification, Phase 5 done-gate, and any repair loop.
5. **Compaction never drops the ledger.** If the harness must compact context, the ledger survives. Everything else may be summarized.
6. **STATUS is the final field**, set by the machine done-gate, not by an LLM verdict.

## Relationship to JSON Schemas

The ledger is human/agent-readable Markdown. The structured fields mirror:

- `task-contract.schema.json` (Phase 1)
- `verification-report.schema.json` (Phase 4/5)

A ledger can be compiled into a verification-report JSON for machine consumption.
