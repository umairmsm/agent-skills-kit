# AGENTS.md — Agent Contract for `agent-skills-kit`

> **Any AI coding agent working in this repository reads this file first.**
> The pointer files at this root (`agent.md`, `CLAUDE.md`, `GEMINI.md`) are **symlinks to this file**. Never replace them with separate files.

*Last verified against the code: 2026-10-27.* If a statement here disagrees with the code, the code wins: fix this file in the same commit.

---

## 1. What this repository is

`agent-skills-kit` is a **library of agent skills** plus a small CLI, **`forge`**, that installs the right skills, an `AGENTS.md` and an `.mcp.json` into a project. (The GitHub repository used to be called `forge`; the command is still `forge`.)

Skills are grouped into **profiles** (a stack, such as WordPress or a static site). `forge init` interviews the user, picks a profile, and copies that profile's skills and its `AGENTS.md` template into the target project.

---

## 2. What is in the repository (checked 2026-10-27)

| Path | What it is |
|---|---|
| `bin/forge` | The CLI (bash). Commands: `forge init [dir]`, `forge list`, `forge add <skill> [dir]`, `forge sync [dir]`, `forge doctor [--fix]`. |
| `skills/<name>/SKILL.md` | One skill per folder (16 today), with optional `references/` and `scripts/`. `SKILL.md` starts with YAML frontmatter: `name`, `description`, and where relevant `compatibility` and `origin`. |
| `profiles/<name>.profile` | A shell-sourceable file per stack. Today: `nest-pg-static`, `static-site`, `wordpress-bedrock`. Fields: `LABEL`, `BLURB`, `SKILLS` (space-separated skill names), `TEMPLATE`, `TOOLS`. |
| `templates/` | `AGENTS.md.<profile>` (the file installed into projects) and `mcp.json`. |
| `BEST_PRACTICES.md` | Brand-neutral rules that the templates are distilled from. |
| `.github/workflows/ci.yml` | CI: `bash -n` and ShellCheck on `bin/forge`, `forge list`, `bash -n` on every profile. |

---

## 3. Rules for changes

1. **One source of truth.** This file is canonical. Keep `agent.md`, `CLAUDE.md` and `GEMINI.md` as symlinks to it.
2. **Adding or changing a skill.**
   - Create `skills/<name>/SKILL.md` with frontmatter `name` (equal to the folder name) and a `description` that says *when* to use it.
   - **Add an `origin:` line** naming where the content comes from (a URL or `original`), and keep upstream licence and credit intact. Do not guess an origin: if you do not know it, say so in the pull request.
   - Keep skills brand-neutral: no company, client, project, host or person names.
   - Register the skill in the `SKILLS=` list of every profile that should install it.
3. **Adding or changing a profile.**
   - A profile is a plain shell file of `KEY="value"` lines. It must pass `bash -n` and must not run commands.
   - Every skill named in `SKILLS=` must exist in `skills/`, and `TEMPLATE=` must name a file in `templates/`.
4. **Templates.** An installed `AGENTS.md` should be command-first and short (`BEST_PRACTICES.md` recommends under 150 lines). Never put project-specific facts or secrets in a template.
5. **Changing `bin/forge`.** It must stay ShellCheck-clean (0 findings). Test `forge list` before committing, and `forge doctor` if you touched it.

---

## 4. Safety and privacy

* **Never commit** API keys, credentials, private hostnames, IP addresses, client names or personal paths. Examples in docs must be fictional.
* **`forge doctor` reads the local machine**: your Claude settings and every script it finds that starts a headless Claude session. Its output contains real file paths. **Never paste it into a commit, issue or pull request.** `forge doctor --fix` edits local settings, so run it only with the owner's approval.
* Automated agents never run `git push`, publish a release, or change repository visibility. Only the owner does.
* Contributions should be signed off (`git commit -s`). CI does not enforce this. An AI agent must not add a sign-off on a person's behalf.
* Use Conventional Commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`.

---

## 5. How to verify your work

```bash
# 1. Syntax and lint of the CLI (0 findings required)
bash -n bin/forge
shellcheck bin/forge

# 2. The commands CI also runs
./bin/forge list
for p in profiles/*.profile; do bash -n "$p"; done

# 3. Every skill named in a profile exists, and every template exists
for p in profiles/*.profile; do
  ( . "$p"
    for s in $SKILLS; do [ -f "skills/$s/SKILL.md" ] || echo "MISSING skill $s in $p"; done
    [ -f "templates/$TEMPLATE" ] || echo "MISSING template $TEMPLATE in $p" )
done

# 4. Secret scan, working tree and history
gitleaks dir . --no-banner --redact
gitleaks git . --no-banner --redact
```

Step 3 prints nothing when everything is consistent. Maintainers also run a private scrub scan before publishing; it lives outside this repository because its pattern list is not public.

---

## 6. Known gaps

* **Origins are missing.** Only 2 of the 16 skills (`context-budget`, `wordpress-modern`) declare an `origin:` line. The rule in section 3 applies to every skill, so the other 14 need their origin and licence recorded by the maintainer. Do not invent them.
* **No tests for `forge init`, `forge add` or `forge sync`.** CI covers syntax, lint, `forge list` and profile syntax only.
* **`forge doctor` is machine-specific** and is not tested in CI.
* The local folder and some older docs may still say `forge`. Treat `agent-skills-kit` as the repository name and `forge` as the command.

---

## 7. Contributing

See `CONTRIBUTING.md`. Good first areas: recording skill origins, a smoke test for `forge init` in a temp directory, and new profiles for other stacks.
