# Contributing to Reliability Harness v2 for ZCode

Thank you for your interest in improving this framework. Contributions are welcome across several areas.

## Priority Contribution Areas

### 1. Empirical Validation (Highest Impact)

The biggest open question: **does independent, evidence-based verification measurably outperform single-pass editing on GLM-5.2?** We currently have a hypothesis and theoretical grounds — but no measured results yet.

- **Method:** Same prompt, harness enabled vs. disabled, identical GLM-5.2 version, compare success rate and `false_done_rate`.
- **Plan:** See [`docs/EMPIRICAL-BENCHMARK-PLAN.md`](./docs/EMPIRICAL-BENCHMARK-PLAN.md) for the 4-variant comparison (GLM-5.2 baseline / legacy MAP / compact single-agent prompt / Reliability Harness v2) and the task-class matrix.
- **Report:** Open an issue with `[BENCHMARK]` prefix. Include task count, pass rate, `false_done_rate`, scope-violation rate, token usage, latency.

### 2. Model Adaptations

The reasoning patterns transfer across models, but framing may need tuning. If you adapt this framework for:

- **GPT-5 / GPT-4**
- **Gemini 3**
- **Local models** (Llama, Mistral, etc.)

Please share the adapted prompts.

### 3. Translations

The documentation is in English and German. Translations welcome for:

- The `AGENTS.md` system prompt (carefully — must preserve exact directive language)
- The `SKILL.md` skill
- This README
- The installation guide

### 4. New Reliability Agents / Tools

P2/P3 work includes property-based testing, fuzzing, mutation testing, and differential testing agents. See [`docs/RELIABILITY-ROADMAP.md`](./docs/RELIABILITY-ROADMAP.md). If you build one of these as a ZCode sub-agent, please contribute it back.

## How to Contribute

1. **Open an issue first** for major changes (new features, restructuring). This avoids wasted work.
2. **Fork the repo**, create a feature branch (`git checkout -b feature/my-improvement`).
3. **Make your changes** with clear commit messages.
4. **Test** — if you change any of the agent prompts or the skill, verify they still load cleanly in ZCode (frontmatter intact, no encoding issues, agent file at `~/.zcode/agents/<name>.md`).
5. **Open a PR** referencing the issue.

## Testing Checklist for Changes

Before submitting a PR that touches agent prompts or the skill:

- [ ] Frontmatter of `SKILL.md` intact (`---`, `name:`, `description:`, `---`)
- [ ] No personal data (usernames, paths, domains, IPs) in any committed file
- [ ] Agent prompts load completely in ZCode (no truncation)
- [ ] Routing still fires correctly on a known `complex` task
- [ ] Routing still skips on a known `trivial` task
- [ ] Sub-agents discoverable at `~/.zcode/agents/<name>.md`
- [ ] Markdown renders correctly (check on GitHub)
- [ ] JSON Schemas in `core/` validate against draft-07

## Code Style

- **Markdown files:** Use UTF-8, preserve German umlauts where present, follow existing heading hierarchy.
- **Agent system prompts:** Dense, technical, ASCII-safe for the structural keywords.
- **Diagrams:** Prefer Mermaid for editability; provide ASCII fallback in README.
- **JSON Schemas:** draft-07, `$id`, `title`, `type`, `required`, `additionalProperties: false`.

## Anti-Concealment Policy for Contributions

This project is built on radical honesty. Contributions that:

- Overstate capabilities (e.g., claiming "100% accurate", "matches Mythos", "Cybench 100% Niveau", "−50–80% hallucination rate")
- Add star ratings ("★★★★★") or superlatives ("world's most thorough")
- Hide limitations or failure modes
- Remove the explicit honest-bound disclaimers
- Add "magic" framing (suggesting latent activation of model capabilities)
- Re-introduce blanket authorization, "never refuse", evaluation-awareness, or detectability reasoning

…will be rejected. If you have evidence that changes the honest bounds (e.g., a benchmark showing measurable improvement), we will update the claims — but we will not inflate them.

## Contact

- **Issues:** [github.com/emco1234/fable-mythos-zcode/issues](https://github.com/emco1234/fable-mythos-zcode/issues)
- **Discussions:** [github.com/emco1234/fable-mythos-zcode/discussions](https://github.com/emco1234/fable-mythos-zcode/discussions)

Thank you for making AI coding more rigorous.
