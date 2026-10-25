---
name: agent-interop
description: Sets up repositories for collaborative work between Claude Code and Antigravity/Gemini, audits context-file drift, diagnoses why Gemini does not know facts from CLAUDE.md, guides adding AGENTS.md, and reconciles divergent CLAUDE.md and GEMINI.md files.
---

# Agent Interop

Ensure frictionless collaboration between Claude Code and Antigravity/Gemini in shared repositories. Eliminate silent context drift, prevent duplicate sources of truth, and maintain clean separation between shared project facts and agent-specific role instructions.

## Overview

Claude Code and Antigravity/Gemini often work on the same codebase, but each platform reads context differently:
- **Claude Code** reads `CLAUDE.md` and `AGENTS.md` natively in the repo root.
- **Antigravity/Gemini** reads `GEMINI.md` in the repo root.

When repositories maintain separate `CLAUDE.md` and `GEMINI.md` files without shared coordination, they inevitably drift — one agent learns critical constraints, architecture decisions, or build rules that the other never sees.

The architecture to prevent this:
1. **Shared project truth lives once in `AGENTS.md`**: Canonical project architecture, core rules, authoritative sources, locked standards, handoff channels, and read orders.
2. **Tool files carry only role briefs or tool-specific machinery**: `CLAUDE.md` and `GEMINI.md` hold only what is unique to that tool or persona.
3. **Existing coordination homes remain authoritative**: If a repo already has a coordination home (`.team/`, `00-Conventions.md`, docs under `3_Areas_Operations/`), `AGENTS.md` must point to it, never duplicate it.

## When to Use

Trigger this skill whenever you need to:
- Set up a repository for both Claude Code and Antigravity/Gemini.
- Audit a repository for context-file drift or divergence.
- Answer "why does Gemini not know X?" when X is documented in `CLAUDE.md`.
- Add an `AGENTS.md` single source of truth to an existing project.
- Reconcile or split existing `CLAUDE.md` and `GEMINI.md` files that disagree.

## Reference Implementations

Do not re-derive the interop pattern from scratch. Consult these live implementations on disk:

- **`projects/e-commerce-app` (Symlink form)**: `GEMINI.md` is a symlink pointing directly to `AGENTS.md` (`GEMINI.md -> AGENTS.md`). `CLAUDE.md` contains only Claude-specific machinery (custom subagents, custom skills, and command harnesses) and explicitly opens by pointing back to `AGENTS.md`.
- **`projects/knowledge-base` (Pointer form with role split)**: Genuine role split where one agent is PM and another is Executor. `AGENTS.md` holds shared facts once. `GEMINI.md` is a dedicated PM brief that opens with `@AGENTS.md` followed by an explicit imperative to read `AGENTS.md` first. `CLAUDE.md` is an executor brief.
- **`projects/booking-portal` (Defers to existing `.team/` protocol)**: Live coordination already exists in `.team/` (`TEAM_STATE.md`, `TASK_QUEUE.md`). `AGENTS.md` is deliberately short, establishing rules and pointing agents to `.team/` without restating it. `GEMINI.md` is a symlink to `AGENTS.md`. No `CLAUDE.md` exists because no Claude-only tooling was needed.
- **`projects/web-platform` (Pre-existing multi-tool symlinks)**: Multi-tool environment linking `.cursorrules`, `.clinerules`, `.windsurfrules`, and `GEMINI.md` back to `AGENTS.md`.

## Core Principle: Model Judgement Over Scripts

Splitting an already-drifted pair of context files is a **judgement task for the model, not a script**.

Whole-file diffs produce noisy, un-mergeable clutter. Determining which facts belong in `AGENTS.md` (shared law) versus `CLAUDE.md`/`GEMINI.md` (tool machinery or role brief) requires semantic understanding of the repository's workflow, agent assignments, and existing coordination documentation.

This skill provides the systematic procedure, heuristics, and verification checks for the model to execute. It does not attempt to automate merges with a blind script.

---

## Mode A — Audit (Default)

**Mode A is read-only. It inspects the repository, reports findings, and modifies nothing.**

Run this mode by default when checking a repository, troubleshooting missing agent context, or auditing for drift.

### Audit Inspection Steps

Execute these checks in order:

#### 1. Real File Collision Check
Check whether both `CLAUDE.md` and `GEMINI.md` exist as regular files (not symlinks):
```bash
[ -f CLAUDE.md ] && [ ! -L CLAUDE.md ] && [ -f GEMINI.md ] && [ ! -L GEMINI.md ]
```
If both exist as real files, flag this as **Candidate Drift**. Report each file's size (in bytes) and last modification timestamp (`mtime`).

#### 2. Factual Drift Detection
Compare the substantive content between tool files (`CLAUDE.md` vs `GEMINI.md`, or `AGENTS.md` vs tool files):
- Do **not** use a raw `diff -u` (too noisy due to structural formatting).
- Extract and compare **markdown section headings** (`## ...`).
- Extract and cross-reference **distinctive tokens**:
  - File paths and directories (e.g. `scripts/`, `backend/`, `src/`)
  - Command invocations (e.g. build, test, deploy commands)
  - Locked metrics, numeric standards, and business constants
  - Environment variable names and operational policies
- Surface every fact present in one file but omitted from the other (the actual drift).

#### 3. Stale Copy Check
Check if `GEMINI.md` is a static **copy** of `AGENTS.md` rather than a symbolic link or a pointer:
```bash
if [ -f GEMINI.md ] && [ ! -L GEMINI.md ] && [ -f AGENTS.md ]; then
  # Check if GEMINI.md merely duplicates AGENTS.md without an @AGENTS.md pointer
  ...
fi
```
If `GEMINI.md` contains duplicate text of `AGENTS.md` instead of being a symlink (`GEMINI.md -> AGENTS.md`) or a role brief opening with `@AGENTS.md`, report it as a stale copy. Any edit to `AGENTS.md` will leave `GEMINI.md` behind.

#### 4. Missing Context Check
Check if a repository where both tools operate lacks context files entirely:
- Detect if both tools have worked here (e.g. git branches matching `claude/*` and `ag/*`, or commits co-authored by Claude and Antigravity).
- Check if neither `AGENTS.md`, `CLAUDE.md`, nor `GEMINI.md` exists.
- Report missing context baseline.

#### 5. Duplication Against Existing Coordination Home (CRITICAL)
**This is the check that matters most. Flag this loudly with highest severity if found.**
- Look for existing coordination files:
  - `.team/` directory (e.g. `TEAM_STATE.md`, `TASK_QUEUE.md`, `AUDIT_LOG.md`)
  - `00-Conventions.md`
  - Documents under `3_Areas_Operations/`
  - `PROJECT.md`
- Inspect `AGENTS.md` and tool files for facts restated from these coordination homes (e.g. re-authoring workflow sequences, duplicating task queues, or rewriting conventions).
- Re-authoring existing coordination state in `AGENTS.md` recreates the exact drift the pattern exists to eradicate.

#### 6. Branch Convention Check (lowest severity — read the caveat first)
The convention is:
- Claude Code: `claude/<topic>`
- Antigravity: `ag/<topic>`
- Neither commits directly to the default branch unless explicitly directed by the owner.

**A fast-forward merge is indistinguishable from a direct commit.** `git log` on the
default branch shows both identically, so a plain log scan reports false positives for
every repo that branched correctly and then fast-forwarded — which is the *recommended*
workflow, not a violation. Do not flag on `git log` alone.

Only report this when there is real evidence of a bypass:

```bash
# Was the commit ever on a side branch? A merge commit or a reflog entry says yes.
git log -n 25 --format='%h %an %s' --first-parent
git reflog show --date=short | head -30    # branch checkouts and merges leave traces here
```

Treat as a finding only if **both** hold: the commit is authored by an agent (`claude`
or an Antigravity identity), **and** the reflog shows it was committed while the default
branch was checked out. If the reflog has aged out or is unavailable, say the check is
**inconclusive** rather than reporting a violation.

Never raise this above `[LOW]`, and never raise it at all on a repo whose history is
entirely fast-forwarded — silence is the correct output when the evidence cannot
distinguish the two cases.

### Audit Output Format

Emit a short, prioritized findings list ranked from highest to lowest severity. Name the affected file(s) and provide the exact, concrete fix. **Do not write a prose essay.**

Example output:
```markdown
### Agent Interop Audit Findings

1. [CRITICAL] Duplication against existing coordination home: `AGENTS.md:14-38` restates workflow rules from `.team/TEAM_STATE.md`.
   Fix: Replace lines 14-38 with a table referencing `.team/TEAM_STATE.md` as the authority.

2. [HIGH] Real file collision / Factual drift: `CLAUDE.md` (3.4 KB, modified 2026-09-10) and `GEMINI.md` (2.1 KB, modified 2026-09-02) are both real files.
   - Distinctive facts in CLAUDE.md missing from GEMINI.md: `npm run test:e2e`, `scripts/export.sh`, `API_RATE_LIMIT=60`.
   - Distinctive facts in GEMINI.md missing from CLAUDE.md: `DOCKER_PORT=8080`.
   Fix: Extract shared facts to `AGENTS.md`. Symlink `GEMINI.md -> AGENTS.md` (or convert to pointer if role split exists).

3. [MEDIUM] Stale copy: `GEMINI.md` is a regular file duplicating `AGENTS.md` rather than a symlink.
   Fix: `rm GEMINI.md && ln -s AGENTS.md GEMINI.md`.

4. [LOW] Direct commits on default branch: `d4a8f12` authored by claude, and the reflog shows it was committed with `master` checked out (not a fast-forward).
   Fix: Enforce the `claude/<topic>` branch rule for future tasks. (If the reflog had aged out, report this as inconclusive instead.)
```

If no issues are found, state cleanly:
```markdown
### Agent Interop Audit Findings
✓ No drift detected. Context files, symlinks, coordination homes, and branch conventions are fully compliant.
```

---

## Mode B — Bootstrap

**Mode B sets up a repository for clean dual-agent operation.**

Follow these steps in strict sequence. **Step 1 is mandatory and non-negotiable:**

### Step 1: Look for an Existing Coordination Home First

Before writing any context file, scan for an existing coordination authority:
- `.team/` directory (e.g. `TEAM_STATE.md`, `TASK_QUEUE.md`)
- `00-Conventions.md`
- `PROJECT.md`
- Docs under `3_Areas_Operations/`
- Pre-existing `AGENTS.md`

**If a coordination home exists:**
`AGENTS.md` must **reference** it, NEVER restate it. For example, in `projects/booking-portal/AGENTS.md`:
```markdown
## Coordination lives in `.team/` — read it, don't restate it

This repo already runs a dual-agent protocol. Read these before acting; they are the authority, not this file:
| File | Holds |
|---|---|
| `.team/TEAM_STATE.md` | Current status, live deployment, roles, collaboration sequence |
| `.team/TASK_QUEUE.md` | What to work on next, specs, constraints |
```
Restating rules from `.team/` in `AGENTS.md` creates a second source of truth that will immediately drift.

### Step 2: Decide Layout (Explain Choice Before Writing)

State which layout you are applying and why before touching files on disk:

#### Option 1: No Role Split (Default for Product Repos)
Use when both agents work as general engineers or implementers on the codebase.
- `AGENTS.md`: Real file holding all shared knowledge (architecture, standards, build/test commands, branch rules, hard guardrails).
- `GEMINI.md`: Symlink to `AGENTS.md`:
  ```bash
  ln -s AGENTS.md GEMINI.md
  ```
- `CLAUDE.md`: Created **only** if genuinely Claude-specific machinery exists (e.g. subagents in `~/.claude/agents/`, custom slash commands, or MCP harness rules). If no Claude-only tooling exists, do **not** create `CLAUDE.md`. If `CLAUDE.md` is needed, it opens with:
  ```markdown
  > **Project law lives in [`AGENTS.md`](AGENTS.md)**. Claude Code loads it automatically
  > and it binds you fully. This file lists only Claude-specific machinery.
  ```

#### Option 2: Genuine Role Split (PM vs Executor / Architect vs Implementer)
Use when agents hold distinct, complementary roles (e.g. where Antigravity is PM and Claude is Executor, or where one agent is Architect and another is Implementer).
- `AGENTS.md`: Shared brief authored once (read order, architecture, source of truth law, handoff channels, shared guardrails).
- `CLAUDE.md`: Short role brief for the executor or implementer.
- `GEMINI.md`: Short role brief for the PM or architect.
  **MANDATORY FORMAT FOR `GEMINI.md` POINTER**:
  `GEMINI.md` must open with an `@AGENTS.md` directive **and** an explicit imperative reading block:
  ```markdown
  # GEMINI.md — Antigravity role brief (<project-name>)

  @AGENTS.md

  > **STOP — read [`AGENTS.md`](AGENTS.md) in this directory before doing anything else.**
  > It is the shared brief for both agents: read order, roles, source-of-truth law,
  > handoff channels, and guardrails. If the import line above did not pull it in
  > automatically, open and read it now.
  >
  > This file adds only what is specific to *you*. Nothing here is duplicated from
  > `AGENTS.md`; if a fact belongs to both agents, it is authored there, once.
  ```
  *(Verified working 2026-09-13: Antigravity follows the `@AGENTS.md` pointer and cites `AGENTS.md` line ranges unprompted).*

### Step 3: Ask for Role Assignment — NEVER Guess

**Never assume or guess agent roles.** Role assignments are not constant across repositories:
- Example A: Antigravity is PMO Orchestrator; Claude Code is the Executor.
- Example B: Antigravity is Lead Architect & Security Auditor; Claude Code is Full-Stack Implementer.

Ask the repository owner / user:
> "What are the intended roles for Claude Code and Antigravity in this repository? (e.g. Equal peers, PM + Executor, Architect + Implementer?)"

Record the confirmed role assignment explicitly in `AGENTS.md` under a `## Roles` section.

### Step 4: Record Branch Conventions

Record the non-negotiable branch rules in `AGENTS.md`:
```markdown
## Branch Convention

- Claude Code works on `claude/<topic>` branches.
- Antigravity works on `ag/<topic>` branches.
- Neither agent commits directly to the default branch (`main` or `master`) unless explicitly instructed by the repository owner.
- Use small, atomic commits describing intent, not mechanics.
```

### Step 5: Security & Secret Hygiene (Grep Before Finishing)

`AGENTS.md` and context files are tracked in git and frequently land in public repositories.

**Never copy secrets, live deployment URLs, webhook credentials, or API keys into `AGENTS.md`.**
- Use placeholder environment variable names (e.g. `CLOUDFLARE_API_KEY`, `DATABASE_URL`).
- Refer to credentials by their configuration location (e.g. "Stored in Script Properties", "Configured in `.env.local`").

**Mandatory verification step before finishing bootstrap**:
Run a regex grep across newly authored context files to ensure no sensitive credentials or live webhook endpoints were pasted:
```bash
grep -E '(sk-[a-zA-Z0-9]{20,}|key-[a-zA-Z0-9]+|gh[p]_[a-zA-Z0-9]+|https://script\.google\.com/macros/s/|https://hooks\.slack\.com)' AGENTS.md CLAUDE.md GEMINI.md 2>/dev/null || true
```
Ensure zero secret leakage before concluding.

---

## Reconciliation Heuristics (Reconciling Drifted Repos)

When migrating an existing drifted repository:
1. **Read both files thoroughly** (`CLAUDE.md` and `GEMINI.md`).
2. **Cluster extracted facts into three categories**:
   - *Category 1: Universal project facts* (commands, architecture, coding guidelines, testing standards, branch rules) → Move to `AGENTS.md`.
   - *Category 2: Claude-only machinery* (subagents, custom skills, slash commands, Archon workflows) → Keep in `CLAUDE.md`.
   - *Category 3: Gemini/Antigravity-only responsibilities* (PM duties, audit workflows) → Keep in `GEMINI.md` (if role split) or merge into `AGENTS.md` (if no role split).
3. **If facts contradict** (e.g. one file says `node 18`, other says `node 20`): Check the codebase reality (`package.json`, `.nvmrc`, Dockerfile, CI workflow) to determine truth. If ambiguous, pause and ask the user.
4. Apply the chosen layout (Symlink or Pointer) and verify that `GEMINI.md` and `CLAUDE.md` contain zero duplicated facts from `AGENTS.md`.
