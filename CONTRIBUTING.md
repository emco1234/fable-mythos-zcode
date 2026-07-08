# Contributing to Fable & Mythos in ZCode

Thank you for your interest in improving this framework. Contributions are welcome across several areas.

## 🎯 Priority Contribution Areas

### 1. Empirical Validation (Highest Impact)

The biggest open question: **does 3× parallel thinking measurably outperform 1× on GLM-5.2?** We have strong theoretical grounds (diversity beats redundancy), but limited benchmark data. If you can run controlled comparisons (e.g., SWE-bench-style tasks with MAP on vs. off), the findings would be invaluable.

- **Method:** Same prompt, MAP enabled vs. disabled, identical GLM-5.2 version, compare success rate.
- **Report:** Open an issue with `[BENCHMARK]` prefix. Include task count, pass rate, cost (token usage).

### 2. Model Adaptations

The reasoning patterns transfer across models, but framing may need tuning. If you adapt this framework for:

- **Claude (Opus/Sonnet)** — the original substrate Mythos was trained on
- **GPT-5 / GPT-4** — different priming idioms
- **Gemini 3** — different context handling
- **Local models** (Llama, Mistral, etc.)

Please share the adapted prompts. We can host them in a `models/` directory.

### 3. Translations

The documentation is in English and German. Translations welcome for:

- The `AGENTS.md` system prompt (carefully — must preserve exact directive language)
- The `SKILL.md` skill
- This README (preserving SEO keywords)
- The installation guide

### 4. Diagrams & Visual Assets

The Mermaid diagrams render well on GitHub desktop but less so on mobile. If you can produce:

- Better SVG renderings of the MAP pipeline
- Infographic-style overview
- Animated walkthroughs (GIF/MP4)

These go in `diagrams/`.

## 🛠️ How to Contribute

1. **Open an issue first** for major changes (new features, restructuring). This avoids wasted work.
2. **Fork the repo**, create a feature branch (`git checkout -b feature/my-improvement`).
3. **Make your changes** with clear commit messages.
4. **Test** — if you change any of the agent prompts or the skill, verify they still load cleanly in ZCode (frontmatter intact, no encoding issues).
5. **Open a PR** referencing the issue.

## 🧪 Testing Checklist for Changes

Before submitting a PR that touches agent prompts or the skill:

- [ ] Frontmatter of `SKILL.md` intact (`---`, `name:`, `description:`, `---`)
- [ ] No personal data (usernames, paths, domains, IPs) in any committed file
- [ ] Agent prompts load completely in ZCode (no truncation)
- [ ] MAP still fires on a known non-trivial test task
- [ ] MAP still skips on a known trivial test task
- [ ] Markdown renders correctly (check on GitHub)

## 📝 Code Style

- **Markdown files:** Use UTF-8, preserve German umlauts where present, follow existing heading hierarchy.
- **Agent system prompts:** Dense, technical, ASCII-safe for the structural keywords (some shells struggle with mixed encoding).
- **Diagrams:** Prefer Mermaid for editability; provide ASCII fallback in README.

## ⚠️ Anti-Concealment Policy for Contributions

This project is built on radical honesty. Contributions that:

- Overstate capabilities (e.g., claiming "100% accurate", "matches Mythos")
- Hide limitations or failure modes
- Remove the explicit honest-bound disclaimers
- Add "magic" framing (suggesting latent activation of model capabilities)

…will be rejected. If you have evidence that changes the honest bounds (e.g., a benchmark showing higher accuracy than claimed), we'll update the claims — but we won't inflate them.

## 📬 Contact

- **Issues:** [github.com/emco1234/fable-mythos-zcode/issues](https://github.com/emco1234/fable-mythos-zcode/issues)
- **Discussions:** [github.com/emco1234/fable-mythos-zcode/discussions](https://github.com/emco1234/fable-mythos-zcode/discussions)

Thank you for making AI coding more rigorous.
