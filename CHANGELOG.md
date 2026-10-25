# Changelog

All notable changes to `forge` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-10-26

### Added
- **Core CLI (`bin/forge`)**:
  - `forge init`: Interactive interview to install stack-specific skills, `AGENTS.md`, and `.mcp.json`.
  - `forge list`: Discover available profiles, descriptions, skills, and tooling prerequisites.
  - `forge add <skill>`: Install individual skills into an existing project.
  - `forge sync`: Update project skills from the central library without overwriting custom edits.
  - `forge doctor`: Audits headless CLI invocations, catches missing `--no-session-persistence` flags, and cleans zero-message transcript bloat.
- **Profiles**:
  - `static-site`: Lightest footprint for static web apps.
  - `nest-pg-static`: Full-stack profile for NestJS, PostgreSQL/pgvector, and static export pipelines.
  - `wordpress-bedrock`: Composer/Bedrock, DDEV, and WP-CLI development stacks.
- **Skill Library (16 skills)**:
  - `agent-interop`: Universal multi-agent context bridge (`AGENTS.md` + pointers).
  - `context-budget`: Token consumption auditor for Claude Code sessions (attributed to ECC).
  - `deploy-static-cpanel`: Pre-flight checks and safe rsync deployment.
  - `design-motion-principles`: Physics-based motion design framework and AI-slop auditor.
  - `docker-stack-dev`: Containerized development workflows.
  - `nest-api-patterns`: Architecture, DTO validation, and BullMQ queue patterns.
  - `pg-pgvector-ops`: Schema design, indexing, and vector similarity operations.
  - `playwright-verify`: Headless browser testing and snapshot assertions.
  - `static-export-build`: Static build pipelines and caching best practices.
  - `ux-researcher`: User interview and heuristic evaluation personas.
  - `web-accessibility-wizard`: Guided WCAG 2.2 accessibility auditor.
  - `wordpress-modern`: WordPress 6.8+ modern asset loading and performance APIs.
  - `wp-performance`: Query and cache profiling.
  - `wp-performance-review`: PHP code-level performance audits.
  - `wp-plugin-development`: Object-oriented WordPress plugin architecture.
  - `wp-wpcli-and-ops`: WP-CLI operations, database exports, and multisite administration.
- **Templates**:
  - Multi-agent contract templates (`AGENTS.md.static-site`, `AGENTS.md.nest-pg-static`, `AGENTS.md.wordpress-bedrock`).
  - Generic Model Context Protocol (`mcp.json`) server configuration template.
