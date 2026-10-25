# AGENTS.md — Agent Contract for `forge`

This repository maintains `forge`, an open-source CLI skill installer and agent context scaffolder.

## Universal Agent Rules

1. **Single Source of Truth**:
   - `AGENTS.md` is canonical law for all agents working in this repository.
   - Symlinks `agent.md`, `CLAUDE.md`, `GEMINI.md`, and `.cursorrules` point directly to `AGENTS.md`. Never replace these symlinks with independent files.
2. **Safety & Privacy Boundaries**:
   - Never commit API keys, personal credentials, private server hostnames, or client names.
   - Run `scan.sh` before staging commits to ensure 0 secrets and 0 personal paths.
3. **Attribution & Intellectual Property**:
   - Every skill ported into `skills/` must clearly document its origin in its YAML frontmatter.
   - Maintain community author credits in `README.md` and `CHANGELOG.md`.
4. **Tool Testing**:
   - Any modifications to `bin/forge` must pass `shellcheck` with zero warnings.
   - Test `bin/forge list` and `bin/forge doctor` locally before committing.
