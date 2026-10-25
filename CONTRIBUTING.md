# Contributing to Forge

Thank you for contributing to `forge`! Forge helps developers scaffold tailored agent rules (`AGENTS.md`), MCP configurations, and reusable skills into any software project.

---

## 1. Ground Rules & Code of Conduct
- Be respectful, constructive, and collaborative. All interactions follow our [Code of Conduct](CODE_OF_CONDUCT.md).
- **Verified receipts only**: New skills and profiles should be based on real development patterns, tested locally with `forge doctor` and `shellcheck`.

---

## 2. Ways to Contribute
1. **New Stacks / Profiles**: Add stack profiles under `profiles/<name>.profile` (e.g. Next.js, Django, Go microservices, Rust web).
2. **New Agent Skills**: Add modular skills under `skills/<skill-name>/` with a clear `SKILL.md` frontmatter and workflow guides.
3. **Doctor Rules**: Enhance `bin/forge doctor` to catch more configuration drift, orphaned transcripts, and session persistence issues.
4. **Agent Templates**: Refine `templates/AGENTS.md.<profile>` to improve multi-agent alignment across Claude Code, Antigravity, and Gemini.

---

## 3. Submitting a Pull Request
1. Fork the repo and create your branch from `main`:
   ```bash
   git checkout -b feat/add-fastapi-profile
   ```
2. Verify syntax and doctor checks:
   ```bash
   shellcheck bin/forge
   ./bin/forge doctor
   ```
3. Open a Pull Request with a clear description of the new stack or skill and test receipts.
