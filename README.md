# Forge 🔨

> Stack-based skill library and multi-agent context scaffolder for Claude Code, Antigravity, and AI-assisted engineering teams.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![CI](https://github.com/umairmsm/forge/actions/workflows/ci.yml/badge.svg)](https://github.com/umairmsm/forge/actions/workflows/ci.yml)
[![Good First Issues](https://img.shields.io/github/issues/umairmsm/forge/good%20first%20issue?color=brightgreen&label=Good%20First%20Issues)](https://github.com/umairmsm/forge/issues)

Every AI coding assistant performs better when given explicit stack constraints, proven architectural patterns, and reusable verification skills. `forge` automates scaffolding these capabilities into your projects in seconds.

---

## What Forge Does

1. **Scaffolds Universal Agent Contracts (`AGENTS.md`)**: Configures single-source-of-truth rules with symlinks (`CLAUDE.md`, `GEMINI.md`, `.cursorrules`) to eliminate drift across AI tools.
2. **Installs Curated Skills**: Bundles 16 production-tested skills covering backend APIs, PostgreSQL/pgvector, accessibility (WCAG 2.2), motion design, and performance profiling.
3. **Maintains System Hygiene (`forge doctor`)**: Scans for headless CLI session persistence leaks, cleans zero-message transcript bloat, and verifies local Claude Code configurations.

---

## Quick Start

### Installation

Clone the repository and add `bin/` to your PATH (or run directly):

```bash
git clone https://github.com/umairmsm/forge.git ~/.forge
export PATH="$HOME/.forge/bin:$PATH"
```

### Inspect Available Profiles

```bash
forge list
```

Output:
```text
Forge profiles

  nest-pg-static — NestJS + Postgres/pgvector + static export
      Database-backed NestJS backend that renders a static HTML site. Docker Compose infra...
      skills: nest-api-patterns pg-pgvector-ops docker-stack-dev static-export-build playwright-verify...
      tools:  node>=20 docker docker-compose rsync openssh-client

  static-site — Static site (no backend)
      Hand-built or generator-produced static HTML/CSS/JS with no runtime...
      skills: static-export-build playwright-verify deploy-static-cpanel web-accessibility-wizard...
      tools:  node>=20 rsync openssh-client

  wordpress-bedrock — WordPress (Bedrock + DDEV)
      Composer-managed WordPress with Bedrock layout, DDEV local dev, WP-CLI ops, Pest tests...
      skills: wordpress-modern wp-performance wp-performance-review wp-plugin-development...
      tools:  php>=8.2 composer ddev wp-cli rsync
```

### Scaffold a New or Existing Project

Navigate to your project root and run:

```bash
forge init .
```

The interactive wizard asks 3 quick questions:
1. Stack profile (e.g. `nest-pg-static`, `static-site`, `wordpress-bedrock`)
2. Project name
3. Production URL and deploy targets

Forge immediately installs:
- `.claude/skills/` containing the exact skills your stack requires
- `AGENTS.md` tailored to your stack with build/test/deploy conventions
- `GEMINI.md -> AGENTS.md` symlink
- `.mcp.json` template

### Add Individual Skills

Already have a project and just need one specific capability?

```bash
forge add web-accessibility-wizard .
forge add pg-pgvector-ops .
```

### Sync Updates

Keep installed skills in sync with the central library without overwriting custom project edits:

```bash
forge sync .
```

### Run System Doctor

Check your development environment and automated scripts for Claude Code session hygiene:

```bash
forge doctor
```

Catches headless invocations missing `--no-session-persistence`, purges empty transcripts in `~/.claude/projects/`, and guarantees project memories are preserved.

---

## Skill Library Inventory

| Skill | Category | Description | Origin / Attribution |
|---|---|---|---|
| `agent-interop` | Architecture | Multi-agent alignment and pointer patterns | Native |
| `context-budget` | Token Optimization | Claude Code session token auditor | [ECC](https://github.com/affaan-m/ECC) |
| `deploy-static-cpanel` | DevOps | Pre-flight validation and safe rsync deployment | Native |
| `design-motion-principles` | Frontend / UI | Motion design framework & AI-slop animation auditor | Distillation of techniques by Emil Kowalski, Jakub Krehel, Jhey Tompkins |
| `docker-stack-dev` | DevOps | Containerized local dev stacks | Native |
| `nest-api-patterns` | Backend | NestJS architecture, DTO validation, and queues | Native |
| `pg-pgvector-ops` | Database | PostgreSQL schema design, indexing, and vector embeddings | Native |
| `playwright-verify` | QA / Testing | Browser automation and visual snapshot verification | Native |
| `static-export-build` | Frontend | SSG build optimization and cache management | Native |
| `ux-researcher` | Product / Design | Usability testing protocols and interview personas | Native |
| `web-accessibility-wizard` | Accessibility | Guided WCAG 2.2 accessibility auditor | [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/) |
| `wordpress-modern` | WordPress | Modern WP 6.8+ asset loading & speculative prefetching | [mikkelkrogsholm/wp-plugins](https://github.com/mikkelkrogsholm) |
| `wp-performance` | WordPress | Query and cache profiling | Native |
| `wp-performance-review` | WordPress | Systematic PHP code-level performance audits | Native |
| `wp-plugin-development` | WordPress | Object-oriented plugin patterns | Native |
| `wp-wpcli-and-ops` | WordPress | WP-CLI automation and database exports | Native |

---

## Contributing

We welcome community contributions!
- Submit new stack profiles in `profiles/<name>.profile`
- Submit new skills in `skills/<skill-name>/`
- Enhance doctor diagnostic rules in `bin/forge`

See [CONTRIBUTING.md](CONTRIBUTING.md) for full guidelines.

---

## License & Third-Party Notice

- Forge is released under the [MIT License](LICENSE). Copyright (c) 2026 umairmsm.
- Skills derived from community works preserve upstream licenses and author attributions (see [CHANGELOG.md](CHANGELOG.md) and individual `SKILL.md` frontmatter).
