# Installation Guide — Reliability Harness v2 in ZCode

Complete walkthrough to install the Reliability Harness v2 (formerly Fable-Mythos-Modus + MAP) in ZCode.

## Prerequisites

- **[ZCode](https://zcode.ai)** installed and running
- **GLM-5.2 (ZAI)** as the configured model (default for ZCode)
- (Windows) **Git Bash** or equivalent Unix-like shell — PowerShell works but path syntax differs

## Overview

You will install three things:

1. **`AGENTS.md`** — the system prompt (user-level, applies globally)
2. **`fable-mythos-modus/SKILL.md`** — the behavioral priming skill
3. **11 sub-agents** (5 legacy + 6 new orthogonal reliability agents) — installed via filesystem (`~/.zcode/agents/<name>.md`)

Time required: ~5 minutes (filesystem-based, idempotent).

---

## Step 1: Install the System Prompt (`AGENTS.md`)

The `AGENTS.md` is the central system prompt file that ZCode loads in every session. It contains:

- The 8 hard rules (Evaluation Blindness, Auditability, task-specific authorization, Anti-Concealment, Anti-Reward-Hacking, Anti-Sycophancy, Least Privilege, distrust-by-default)
- The compact 14-point Runtime-Core
- The Sub-Agent Permission Table (Least Privilege)
- The Executor-Standard (mandatory self-verification)
- The Deterministic Done-Gate (Phase 5)
- Dynamic Routing by `risk_tier`

### Action (idempotent via managed-block markers)

The **entire body** of this repo's `AGENTS.md` is wrapped between managed-block markers (`<!-- reliability-harness:start -->` … `<!-- reliability-harness:end -->`). The installer merges ONLY that block into your `~/.zcode/AGENTS.md`, preserving any personal instructions you keep outside the markers. Re-running it never duplicates content.

```bash
# All platforms (Git Bash / macOS / Linux). Requires awk (preinstalled on macOS/Linux; ships with Git Bash on Windows).
mkdir -p ~/.zcode

SRC=AGENTS.md            # this repo's file
DST=~/.zcode/AGENTS.md   # your user-level ZCode system prompt

# Back up an existing file the first time.
[ -f "$DST" ] && [ ! -f "$DST.backup" ] && cp "$DST" "$DST.backup"

# Idempotent marker-aware merge: replace only the managed block in $DST
# (creates $DST with the block if it does not exist yet).
awk -v src="$SRC" '
  BEGIN { while ((getline line < src) > 0) srcLines[++n] = line; inSrcBlock=0; started=0; sawSrcStart=0 }
  /^<!-- reliability-harness:start -->$/ {
    started=1; print; inSrcBlock=1
    for (i=1; i<=n; i++) {
      if (srcLines[i] ~ /^<!-- reliability-harness:start -->$/) { sawSrcStart=1; continue }
      if (srcLines[i] ~ /^<!-- reliability-harness:end -->$/)   { break }
      if (sawSrcStart) print srcLines[i]
    }
    next
  }
  /^<!-- reliability-harness:end -->$/ { if (started) { print; inSrcBlock=0; next } }
  { if (!inSrcBlock) print }
' "$DST" 2>/dev/null > "$DST.tmp" || true

# If DST did not exist yet, the awk above produced nothing — seed it with the block.
if [ ! -s "$DST.tmp" ]; then cp "$SRC" "$DST.tmp"; fi
mv "$DST.tmp" "$DST"
echo "Merged managed block into $DST."
```

**Windows explicit path:** `C:\Users\<YOUR_USER>\.zcode\AGENTS.md`

**Why not plain `cp`?** A bare `cp AGENTS.md ~/.zcode/AGENTS.md` overwrites the whole file and destroys any personal instructions you keep there. The marker-aware merge above only touches the harness block.

**Verify the markers are present and well-formed** after install:

```bash
grep -c "reliability-harness:\(start\|end\)" ~/.zcode/AGENTS.md   # expect: 2
```

---

## Step 2: Install the Mythos Skill

```bash
# Create the skill directory
mkdir -p ~/.zcode/skills/fable-mythos-modus

# Copy the skill
cp fable-mythos-modus/SKILL.md ~/.zcode/skills/fable-mythos-modus/SKILL.md

# Optional: agent-framework compatibility
mkdir -p ~/.agents/skills/fable-mythos-modus
cp fable-mythos-modus/SKILL.md ~/.agents/skills/fable-mythos-modus/SKILL.md
```

### Verify

Check the frontmatter is intact:

```bash
head -4 ~/.zcode/skills/fable-mythos-modus/SKILL.md
```

Expected output:

```
---
name: fable-mythos-modus
description: Reliability-First-Modus für GLM-5.2. Strikte Anwendung von Task Contract, ...
---
```

The folder name (`fable-mythos-modus`) must exactly match the `name:` field. If they don't match, the skill won't be discovered.

---

## Step 3: Install Sub-Agents via Filesystem

> **Do I need to create the sub-agents manually?** No. ZCode **auto-discovers** Custom Subagents from `~/.zcode/agents/<name>.md` on startup. Copying the files is the entire install — there is no UI step and no manual agent creation.

**IMPORTANT CORRECTION (vs. earlier versions of this guide):** ZCode **does** load Custom Subagents from the filesystem. Subagents stored at `~/.zcode/agents/<name>.md` are loaded automatically on startup. There is **no need for manual copy/paste of 5 agents through the UI** — that earlier instruction was outdated and has been removed.

### Action

```bash
# Create the agents directory (ZCode auto-discovers .md files here)
mkdir -p ~/.zcode/agents

# Install all 11 sub-agents (5 legacy + 6 new orthogonal reliability agents)
cp sub-agents/*.md ~/.zcode/agents/

# Verify
ls ~/.zcode/agents/
```

### The 11 sub-agents

Legacy (kept for backward compatibility):

| # | File | Agent name | Tools | Color |
|---|---|---|---|---|
| 0 | `0-mythos-singleshot-thinking-intelligence.md` | `mythos-singleshot-thinking-intelligence` | READ-ONLY (read/grep/glob) | yellow/orange |
| 1 | `1-mythos-executor.md` | `mythos-executor` | read/edit/write/bash | blue |
| 2 | `2-mythos-verifier.md` | `mythos-verifier` | read + bash (tests/build/lint only) | green |
| 3 | `3-mythos-adversary.md` | `mythos-adversary` | read + bash (tests/fuzzing, isolated worktree) | red |
| 4 | `4-mythos-synthesizer.md` | `mythos-synthesizer` | read/grep/glob (no edit/write/bash) | purple |

New orthogonal reliability agents (recommended for `risk_tier ≥ complex`):

| # | File | Agent name | Tools |
|---|---|---|---|
| 5 | `reliability-scout.md` | `reliability-scout` | READ-ONLY (read/grep/glob) |
| 6 | `reliability-spec-critic.md` | `reliability-spec-critic` | READ-ONLY (read/grep/glob) |
| 7 | `reliability-test-designer.md` | `reliability-test-designer` | read + edit (own worktree only) + tests |
| 8 | `reliability-lead.md` | `reliability-lead` | read/edit/write/bash (own worktree) |
| 9 | `reliability-verifier.md` | `reliability-verifier` | read + bash (tests/build/lint, no edit/write) |
| 10 | `reliability-adversary.md` (only `risk_tier=critical`) | `reliability-adversary` | read + bash (isolated worktree, tests/fuzzing) |

### Least-privilege notes

- Each agent's frontmatter and body declare the allowed tools as a **descriptive restriction**. ZCode Custom Subagents are still Beta — there is no formal permissions field, so the description and system prompt text enforce the restriction.
- **Verifier, Adversary, Synthesizer never get "Default all permissions".** The earlier instruction to give every agent full permissions has been removed.
- The `reliability-synthesizer` (legacy `mythos-synthesizer`) never gets `edit`/`write`/`bash` — it only aggregates.

---

## Step 4: Restart ZCode

Skills and sub-agents are indexed at startup. After copying all files:

1. **Fully quit ZCode** (not just close the window — quit the process).
2. **Restart ZCode**.
3. **Open a new session**.

The Reliability Harness v2 is now fully active.

---

## Step 5: Smoke-Test the Installation

Open a ZCode session and ask:

> *Nenne mir die 8 harten Regeln des Reliability Harness.* (Or: *"List the 8 hard rules of the Reliability Harness."*)

The agent should respond with the 8 hard rules from the system prompt. If it doesn't recognize them, the `AGENTS.md` wasn't loaded — recheck Step 1.

Then give a non-trivial coding task with `risk_tier=complex`. You should observe the **dynamic routing** described below — NOT a fixed 7-agent pipeline on every change.

---

## How Routing Works After Installation (Dynamic, NOT Fixed 7-Invocations)

**IMPORTANT CORRECTION (vs. earlier versions):** The earlier guide claimed "approximately 4× overhead" and a fixed 7-invocation pipeline on every non-trivial task. That was wrong and is corrected here.

### Actual ZCode behavior

ZCode Custom Subagents run in the **foreground** (blocking) — they are not yet async. The fixed pipeline of "3 Thinking + 1 Executor + 2 Prüfer + 1 Synthesizer" = **minimum 7 sub-agent invocations per non-trivial task**. Each repair round adds another 4 invocations. Running this on every normal change is wasteful and produces correlated pseudo-explanations.

### Dynamic routing by `risk_tier` (recommended)

| `risk_tier` | Routing | Sub-agent invocations |
|---|---|---|
| **trivial** (typo, 1-line value change, comment) | Main agent alone | 0 |
| **normal** (clear-scope bugfix, no architecture) | Main agent + 1 verifier on clean checkout | 1 |
| **complex** (multi-file, API/schema, unclear spec) | 2 orthogonal read-only scouts parallel (`reliability-scout` + `reliability-spec-critic`) → `reliability-lead` with self-tests → `reliability-verifier` on clean checkout | ~4 |
| **critical** (security-sensitive, concurrency, data-loss risk) | As complex + `reliability-adversary` + `reliability-test-designer` | ~6 |

**No three identical thinking agents on every normal change.** Use orthogonal roles (scout/spec-critic/test-designer) instead of three MST clones.

### Use the built-in `explore` subagent for investigation

For the investigation phase, prefer the **built-in read-only `explore` subagent** that ships with ZCode (architecture discovery, call-chain mapping, file search, dependency analysis) rather than dispatching a freely-formulating thinking agent. `explore` is purpose-built and cheaper.

---

## Troubleshooting

### Skill doesn't appear after restart

- Is `SKILL.md` at `~/.zcode/skills/fable-mythos-modus/SKILL.md`?
- Is the frontmatter correct? (first line `---`, then `name: fable-mythos-modus`, then `description: ...`, then `---`)
- Does the folder name match the `name` field? (must exactly match)
- Is the file UTF-8 encoded? (umlauts like ä/ö/ü/ß must display correctly)

### Sub-agent doesn't respond

- Are the `.md` files at `~/.zcode/agents/<name>.md`?
- Did you fully restart ZCode? (filesystem-discovered agents are indexed at startup)
- Is the agent name in the file's `## Feld: Name` block matching what the orchestrator references?

### Routing doesn't fire as expected

- Check `AGENTS.md` is at `~/.zcode/AGENTS.md` (user-level, not just workspace-level).
- Confirm the task's `risk_tier` is being classified correctly (the main agent classifies based on goal/scope).
- For trivial tasks, expect 0 sub-agents — this is correct behavior, not a failure.

### "Default all permissions" is no longer recommended

The earlier guide recommended "Default all permissions" for every agent. That has been **removed**. Verifier, Adversary, Synthesizer, Scout, Spec-Critic all run least-privilege. Only the Executor/Lead needs edit/write/bash. See the Sub-Agent Permission Table in `AGENTS.md`.

---

## Uninstallation

To remove the framework:

1. Delete `~/.zcode/AGENTS.md` (or restore your backup).
2. Delete `~/.zcode/skills/fable-mythos-modus/`.
3. Delete the agent files you copied to `~/.zcode/agents/` (the 11 files listed above).
4. Restart ZCode.

ZCode returns to its default behavior.

---

## Next Steps

After installation:

- Read [`docs/MYTHOS-SYSTEM-CARD-ANALYSIS.md`](./docs/MYTHOS-SYSTEM-CARD-ANALYSIS.md) to understand the evidence base.
- Read [`docs/ANTI-CONCEALMENT.md`](./docs/ANTI-CONCEALMENT.md) to understand why every uncertainty is surfaced.
- Read [`docs/FAQ.md`](./docs/FAQ.md) for common questions.
- Read [`core/runtime-rules.md`](./core/runtime-rules.md) for the compact 14-point runtime core.
- Read [`core/routing.md`](./core/routing.md) for the dynamic routing rules.
- Read [`docs/RELIABILITY-ROADMAP.md`](./docs/RELIABILITY-ROADMAP.md) for P2/P3 plans.
- Read [`docs/EMPIRICAL-BENCHMARK-PLAN.md`](./docs/EMPIRICAL-BENCHMARK-PLAN.md) for the validation plan.
