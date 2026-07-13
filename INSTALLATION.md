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
3. **11 sub-agents** (5 legacy + 6 new orthogonal reliability agents) — created via the ZCode UI (**Settings → Subagents → New**). Custom Subagents are Beta; you must create each one in the UI, you cannot just copy `.md` files into `~/.zcode/agents/`.

Time required: ~10-15 minutes (UI-based creation of 11 subagents; the harness block in `AGENTS.md` itself installs idempotently in seconds).

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

## Step 3: Create Sub-Agents via ZCode UI

> **Custom Subagents are Beta and require UI-based creation.** ZCode Custom Subagents are a **Beta** feature. Per the official ZCode docs (https://zcode.z.ai/en/docs/subagents), you create them from **Settings → Subagents → New**. ZCode then writes the subagent as a Markdown file under `~/.zcode/agents/<name>.md` and loads it on the next run.
>
> **Manually copying `.md` files into `~/.zcode/agents/` is NOT sufficient.** ZCode only indexes a subagent after it has been created through the Settings UI. The `sub-agents/*.md` files in this repo are the **system-prompt source** (the prompt body you paste into the UI), not a drop-in install.

### What you actually do (once per subagent, 11 in total)

For **each** of the 11 subagents listed in the table below:

1. Open **Settings → Subagents → New** in the ZCode TUI.
2. Fill the fields:
   - **Name**: the `Agent name` value from the table (e.g. `reliability-verifier`).
   - **Description**: paste the text from the `## Feld: Description` block of the matching `sub-agents/*.md` file.
   - **Available tools**: set according to the **Permission Table in `AGENTS.md`** (see the per-role mapping in the table below; this is the authoritative source — ZCode writes its own frontmatter when you save, so do not rely on any inline `Allowed tools` hint alone).
   - **System prompt**: open the matching `sub-agents/*.md` file in this repo, copy the body **after** the `## Feld: System prompt` heading (the text inside the fenced block), and paste it here. Do not include the `## Feld: Name` / `## Feld: Description` blocks.
3. Click **Save**. ZCode writes `~/.zcode/agents/<name>.md` and indexes the subagent on the next run.
4. Repeat for all 11 subagents.

> **Beta limitation note.** This per-subagent UI step exists because ZCode's Custom Subagents are still Beta and have no filesystem auto-discovery yet. If a later ZCode release adds true auto-discovery of `~/.zcode/agents/*.md`, this section will shrink to a single `cp` command. Until then, UI creation is required for the agents to be loaded.

### Where to find each subagent's values

The `sub-agents/*.md` files in this repo are the **prompt and configuration source** for the UI fields above — they are NOT a drop-in install. Each file contains a `## Feld: Name`, `## Feld: Description`, and `## Feld: System prompt` block whose contents you paste into the corresponding UI fields. The **Available tools** per role are listed in the table below and in the Sub-Agent Permission Table in `AGENTS.md`.

### The 11 sub-agents (UI field values + available-tools mapping)

> The **Available tools** column below is exactly what you set in the ZCode UI field `Available tools` for each role. The same mapping appears in the **Sub-Agent Permission Table** in `AGENTS.md`, which is the authoritative reference.

Legacy (kept for backward compatibility):

| # | File | Name (UI) | Description (UI) source | Available tools (UI) | Color (UI) |
|---|---|---|---|---|---|
| 0 | `0-mythos-singleshot-thinking-intelligence.md` | `mythos-singleshot-thinking-intelligence` | `## Feld: Description` in file | Read, Grep, Glob (read-only) | yellow/orange |
| 1 | `1-mythos-executor.md` | `mythos-executor` | `## Feld: Description` in file | Read, Edit, Write, Bash | blue |
| 2 | `2-mythos-verifier.md` | `mythos-verifier` | `## Feld: Description` in file | Read + Bash (tests/build/lint only) | green |
| 3 | `3-mythos-adversary.md` | `mythos-adversary` | `## Feld: Description` in file | Read + Bash (tests/fuzzing, isolated worktree) | red |
| 4 | `4-mythos-synthesizer.md` | `mythos-synthesizer` | `## Feld: Description` in file | Read, Grep, Glob (no Edit/Write/Bash) | purple |

New orthogonal reliability agents (recommended for `risk_tier ≥ complex`):

| # | File | Name (UI) | Description (UI) source | Available tools (UI) |
|---|---|---|---|---|
| 5 | `reliability-scout.md` | `reliability-scout` | `## Feld: Description` in file | Read, Grep, Glob (read-only) |
| 6 | `reliability-spec-critic.md` | `reliability-spec-critic` | `## Feld: Description` in file | Read, Grep, Glob (read-only) |
| 7 | `reliability-test-designer.md` | `reliability-test-designer` | `## Feld: Description` in file | Read + Edit (own worktree only) + Bash (tests) |
| 8 | `reliability-lead.md` | `reliability-lead` | `## Feld: Description` in file | Read, Edit, Write, Bash (own worktree) |
| 9 | `reliability-verifier.md` | `reliability-verifier` | `## Feld: Description` in file | Read + Bash (tests/build/lint, no Edit/Write) |
| 10 | `reliability-adversary.md` (only `risk_tier=critical`) | `reliability-adversary` | `## Feld: Description` in file | Read + Bash (isolated worktree, tests/fuzzing) |

### Least-privilege notes

- Each agent's system-prompt body declares the allowed tools as a **descriptive restriction**, and you must additionally enforce them in the ZCode UI field **Available tools** (per the table above / the Sub-Agent Permission Table in `AGENTS.md`).
- **Verifier, Adversary, Synthesizer never get "Default all permissions".** The earlier instruction to give every agent full permissions has been removed.
- The `reliability-synthesizer` (legacy `mythos-synthesizer`) never gets `edit`/`write`/`bash` — it only aggregates.

---

## Step 4: Restart ZCode

Skills are indexed at startup. After creating all 11 subagents via the Settings UI and saving them:

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

- Did you create the subagent via **Settings → Subagents → New** and click **Save**? (ZCode only indexes subagents that were created through the UI — see Step 3. Dropping an `.md` file into `~/.zcode/agents/` is not sufficient in the current Beta.)
- Did you fully restart ZCode after saving all 11 subagents? (subagents created in the UI are indexed on the next startup)
- Is the agent name in the UI field `Name` matching what the orchestrator references (e.g. `reliability-verifier`, `mythos-executor`)?

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
3. Remove the 11 subagents via **Settings → Subagents** (delete each one), or delete the `~/.zcode/agents/<name>.md` files that ZCode wrote when you created them.
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
