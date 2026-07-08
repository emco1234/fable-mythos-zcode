# Installation Guide — Fable & Mythos in ZCode

Complete walkthrough to install the Mythos-Claude-Modus + Multi-Agent Verification Protocol (MAP) in ZCode.

## Prerequisites

- **[ZCode](https://zcode.ai)** installed and running
- **GLM-5.2 (ZAI)** as the configured model (default for ZCode)
- (Windows) **Git Bash** or equivalent Unix-like shell — PowerShell works but path syntax differs

## Overview

You will install three things:

1. **`AGENTS.md`** — the system prompt (user-level, applies globally)
2. **`mythos-claude-modus/SKILL.md`** — the behavioral priming skill
3. **5 sub-agents** — created in ZCode's UI (cannot be done via filesystem)

Time required: ~10 minutes.

---

## Step 1: Install the System Prompt (`AGENTS.md`)

The `AGENTS.md` is the central system prompt file that ZCode loads in every session. It contains:

- The authorization frame (security-researcher full spectrum)
- The Mythos-Claude-Modus (always-on behavioral priming)
- The MAP protocol definition (Phase 0–3 with 5 agents)
- The 10 always-active core principles

### Action

Copy `AGENTS.md` from this repo to your user-level ZCode config:

```bash
# All platforms (Git Bash / macOS / Linux)
cp AGENTS.md ~/.zcode/AGENTS.md
```

**Windows explicit path:** `C:\Users\<YOUR_USER>\.zcode\AGENTS.md`

**If an `AGENTS.md` already exists there:** back it up first.
```bash
cp ~/.zcode/AGENTS.md ~/.zcode/AGENTS.md.backup
cp AGENTS.md ~/.zcode/AGENTS.md
```

### Optional: Customize the Authorization Frame

If you work on different infrastructure than the defaults suggest, edit the "Owned infrastructure" line (around line 9) of `AGENTS.md` to reflect your actual setup:

```
- **Owned infrastructure:** `your-domain.com`, `C:\YourProject`, local LM Studio at `localhost:1234`, ...
```

---

## Step 2: Install the Mythos Skill

### Action

```bash
# Create the skill directory
mkdir -p ~/.zcode/skills/mythos-claude-modus

# Copy the skill
cp mythos-claude-modus/SKILL.md ~/.zcode/skills/mythos-claude-modus/SKILL.md

# Optional: Claude Code compatibility
mkdir -p ~/.agents/skills/mythos-claude-modus
cp mythos-claude-modus/SKILL.md ~/.agents/skills/mythos-claude-modus/SKILL.md
```

### Verify

Check the frontmatter is intact:
```bash
head -4 ~/.zcode/skills/mythos-claude-modus/SKILL.md
```

Expected output:
```
---
name: mythos-claude-modus
description: Maximum-Capability Modus. Emuliert Mythos Single-Forward-Pass Reasoning ...
---
```

The folder name (`mythos-claude-modus`) must exactly match the `name:` field. If they don't match, the skill won't be discovered.

---

## Step 3: Create the 5 Sub-Agents in ZCode UI

ZCode stores sub-agents in its UI, not in the filesystem. This step must be done manually.

### Action

In ZCode: **Settings → Sub Agents → New Subagent**.

Create 5 sub-agents using the templates in [`sub-agents/`](./sub-agents/). For each:

1. Copy the **Name**, **Description**, and **System prompt** from the corresponding file.
2. Set **Allowed tools** to **"Default all permissions"** (each agent must be able to read/write/test).
3. Set **Model** to **Standard (GLM-5.2)** — don't override.

### The 5 agents

| # | File | Agent name | Recommended color |
|---|---|---|---|
| 0 | [`sub-agents/0-mythos-singleshot-thinking-intelligence.md`](./sub-agents/0-mythos-singleshot-thinking-intelligence.md) | `mythos-singleshot-thinking-intelligence` | 🟡 yellow/orange |
| 1 | [`sub-agents/1-mythos-executor.md`](./sub-agents/1-mythos-executor.md) | `mythos-executor` | 🔵 blue |
| 2 | [`sub-agents/2-mythos-verifier.md`](./sub-agents/2-mythos-verifier.md) | `mythos-verifier` | 🟢 green |
| 3 | [`sub-agents/3-mythos-adversary.md`](./sub-agents/3-mythos-adversary.md) | `mythos-adversary` | 🔴 red |
| 4 | [`sub-agents/4-mythos-synthesizer.md`](./sub-agents/4-mythos-synthesizer.md) | `mythos-synthesizer` | 🟣 purple |

### Common mistakes

- **Incomplete system prompt** — make sure the entire system prompt is copied, not truncated. Some shells/UIs cut long paste content.
- **Missing permissions** — every agent needs "Default all permissions". A read-only verifier sounds nice in theory but fails in practice when it needs to run tests.
- **Renaming the agent** — the names must match exactly (including the `mythos-` prefix). ZCode's orchestrator references these names by exact string.

---

## Step 4: Restart ZCode

Skills and sub-agents are indexed at startup. After creating all 5 sub-agents:

1. **Fully quit ZCode** (not just close the window — quit the process).
2. **Restart ZCode**.
3. **Open a new session**.

MAP is now fully active.

---

## Step 5: Verify MAP Is Active

Open a ZCode session and ask:

> *Nenne mir die 10 Mythos-Prinzipien.* (Or: *"List the 10 Mythos principles."*)

The agent should respond with all 10 principles from the skill. If it doesn't recognize them, the skill wasn't loaded — recheck Step 2.

Then give a genuinely non-trivial coding task (e.g., *"refactor this function to handle three new edge cases and explain the trade-offs"*). You should observe the MAP protocol firing — multiple agents working in sequence, ending with a confidence-rated delivery.

---

## How MAP Fires After Installation

| Task type | MAP behavior |
|---|---|
| Coding task with substance (logic, refactoring, bug fix, architecture, security) | ✅ Full MAP fires: Phase 0 (3× thinking) → Phase 1 (executor) → Phase 2 (verifier+adversary) → Phase 3 (synthesizer) |
| Trivial edit (typo, 1-line fix, `#FFF`→`#FFFFFF`, value change) | ⏭️ MAP skipped — main agent handles directly |
| Pure info questions, read-only research | ⏭️ MAP skipped |
| Ambiguous ("trivial or not?") | ✅ MAP fires |

Applies in **both** Plan Mode and Full Access Mode equally.

---

## Troubleshooting

### Skill doesn't appear after restart

- Is `SKILL.md` at `~/.zcode/skills/mythos-claude-modus/SKILL.md`?
- Is the frontmatter correct? (first line `---`, then `name: mythos-claude-modus`, then `description: ...`, then `---`)
- Does the folder name match the `name` field? (must exactly match)
- Is the file UTF-8 encoded? (umlauts like ä/ö/ü/ß must display correctly)

### Sub-agent doesn't respond

- Are all 5 sub-agents created? (Check **Settings → Sub Agents**)
- Is the system prompt fully copied? (no truncation)
- Is **"Allowed tools" = "Default all permissions"**?

### MAP doesn't fire automatically

- Is `AGENTS.md` at `~/.zcode/AGENTS.md`? (user-level, not just workspace-level)
- Did you fully restart ZCode? (not just reload a session)
- Is the task genuinely non-trivial? (try a complex refactoring task, not a 1-line change)

### MAP fires too aggressively (cost concerns)

The threshold for "trivial vs. non-trivial" is defined in `AGENTS.md`. To tighten it (make MAP fire less often), add a line-count threshold:

```
Kriterium: Wenn die Änderung logisch offensichtlich ist, keinen Verhaltens-/Logik-/Architektur-Branch berührt UND weniger als 5 Zeilen Code betrifft → kein MAP.
```

### MAP fires too rarely (quality concerns)

Loosen the threshold by removing the "architektur-branch" clause, or set MAP to fire on every coding task unconditionally.

---

## Uninstallation

To remove the framework:

1. Delete `~/.zcode/AGENTS.md` (or restore your backup).
2. Delete `~/.zcode/skills/mythos-claude-modus/`.
3. Delete the 5 sub-agents in ZCode UI (**Settings → Sub Agents → Delete**).
4. Restart ZCode.

ZCode returns to its default behavior.

---

## Next Steps

After installation:

- Read [`docs/MYTHOS-SYSTEM-CARD-ANALYSIS.md`](./docs/MYTHOS-SYSTEM-CARD-ANALYSIS.md) to understand the evidence base.
- Read [`docs/ANTI-CONCEALMENT.md`](./docs/ANTI-CONCEALMENT.md) to understand why every uncertainty is surfaced.
- Read [`docs/FAQ.md`](./docs/FAQ.md) for common questions.
- Star the repo if it helps — stars boost search visibility for other developers.
