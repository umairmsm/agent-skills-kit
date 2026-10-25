# BEST_PRACTICES.md — brand-neutral rules for any Claude Code project

Distilled from production enterprise and web platform engineering workspaces. Everything here is **brand-neutral** — copy it into
a new project, or fold the relevant parts into that project's `AGENTS.md` / `CLAUDE.md`.
Brand-specific rules (design tokens, agent codenames, content-domain rules) are deliberately excluded — write those fresh per application.

Part 1 applies to every profile. Part 2 is WordPress-only. Part 3 covers the
backend + static-export stack. Read Part 1 plus the part matching your profile.

> Convention used below: `prefix_` = your project's chosen code prefix (e.g. `acme_`).

---

# Part 1 — Universal

## 1. Session / workflow discipline
- **Two context files, read first, every session:** `CLAUDE.md` (stable locked rules) +
  `SESSION_LOG.md` (living state — append after each confirmed step). Keep volatile state
  OUT of `CLAUDE.md`.
- **`AGENTS.md` is command-first and < 150 lines**, organised as three tiers:
  **Always do / Ask first / Never do.** It's the highest-leverage file.
- **Decision log table** (`date · decision · reasoning`) so settled choices aren't
  re-litigated.
- Confirm destructive commands; chain non-destructive work without asking.

## 2. Git & repo hygiene
- **Branch → local test → diff → `merge --no-ff` → push.** Never commit straight to the
  default branch.
- **Stage only intended files** — never blanket `git add -A` (it once swept ~6,200 WP-core
  files + wp-config salts into a repo). Use explicit paths.
- **`.gitignore` tracks only YOUR code.** Ignore vendored dependencies, build output,
  generated exports, `uploads/`, `node_modules/`, `_vendor/`, backups, and every
  `.env`/`wp-config*.php`.
- **Conventional commits** (`feat`/`fix`/`perf`/`chore`/`docs`) + a `Co-Authored-By` trailer.
- **Repo = code source of truth; DB = content source of truth.** Keep a dated,
  DB-inclusive `backup.sh`; push to a private remote + one off-machine copy; maintain a
  `DISASTER_RECOVERY.md` + `CREDENTIALS_INVENTORY.md` restore runbook.

## 3. Secrets
- **`${VAR}` placeholders in `.mcp.json`, never plaintext keys.** Export secrets from your
  shell profile (`FIRECRAWL_API_KEY`, `GITHUB_PERSONAL_ACCESS_TOKEN`).
- **`.claude/settings.local.json` is git-ignored** (machine-specific permission allowlist).
  `skills/`, `.mcp.json`, `AGENTS.md` are tracked and shareable.
- **Never commit `.env`.** Keep `.env.example` current — every key the stack needs, with a
  safe placeholder. It is the only record of what was required.

## 4. Claude Code setup
- **Lean MCP set:** playwright · fetch · firecrawl · github. More servers degrade quality.
- **Brand-neutral skills only** in `.claude/skills/`; exclude brand-locked skills.
- Skills = recipes with progressive disclosure. Use `forge init` to scaffold a new
  project rather than copying a `.claude/` folder by hand; use `forge sync` to pull
  library updates back in.
- **Edit skills in the library, never in a project's installed copy** — `forge sync`
  overwrites project copies.
- Audit context overhead periodically with the `context-budget` skill. Descriptions load
  every session; skill bodies only load when the skill fires.

## 5. Accessibility — always-on, WCAG AA
Single `<h1>` per page · one `<main>` landmark per page · AA contrast · keyboard focus
order · `prefers-reduced-motion` support · `alt` on every image · labelled form controls.
Back it with an **axe/Playwright suite** (`test:a11y`, `test:visual`, `test:seo`).

## 6. Content integrity (principle — adapt the specifics)
- **Never fabricate** reviews, ratings, counts, or testimonials. Use real data or dynamic
  comments only.
- **Real photos for any named place/person**; AI/placeholder imagery only if prefixed
  (`ai-`/`temp-ai-`) and logged in a swap manifest.
- **Verify external facts** (safety levels, stats, prices) against a cited source before
  publishing copy.

## 7. Verification discipline
- **Measure a baseline first, re-verify with the same method** (same URL, warmed cache).
- **Distinguish pre-existing from introduced failures** before claiming a regression:
  stash your edits, re-run the suite, compare. (Saved a false alarm on SEO tests once.)
- Prefer WP-CLI / HTTP / logs over UI when a browser isn't available; treat the test
  suite as the source of truth, not vibes.

---

# Part 2 — WordPress (`wordpress-bedrock`)

## 8. WordPress engineering rules
- **WordPress Coding Standards + prefix everything** (`prefix_`). Wire `phpcs`/`phpcbf`
  into DDEV (`ddev phpcs` / `ddev phpcbf`, child-theme scope).
- **No page builders.** **No AI-generated PHP to production** without
  branch → local test → diff → human merge.
- **Self-hosted fonts, zero Google Fonts network calls** (GDPR — verify "0 googleapis
  refs"). Disable the theme's Google Fonts (e.g. Astra) in `functions.php`.
- **One schema owner** (e.g. RankMath); suppress duplicate JSON-LD emitted by other
  plugins (price-less Offers etc. cause Rich Results errors).
- **Dequeue unused plugin assets** when you render custom UI instead of the plugin's
  (measured −80% page weight once).
- **Image delivery:** AVIF/WebP, hero budget < 150 KB desktop / < 100 KB mobile;
  lazy-load + `decoding="async"` on below-fold images, keep the hero eager with
  `fetchpriority="high"`; preload the LCP image. Never ship a heavy PNG where a WebP fits.
- **`.gitignore` for WP specifically:** track only child theme + custom plugins
  (`prefix-*`). Ignore WP core, vendored plugins/themes, `uploads/`, `wp-config*.php`.

## 9. Security pre-launch checklist
WPScan scan (token-free enumeration first) · disable XML-RPC · block `?author=N`
enumeration (301→home) · restrict REST `/users` (401) · empty the generator meta ·
decouple admin login from display name/nicename · stage a `deploy-hardening.md`
(`.htaccess`/nginx blocks, `wp-config` constants `DISABLE_WP_CRON`/`DISALLOW_FILE_EDIT`/
`FORCE_SSL_ADMIN`, real system cron, Cloudflare WAF) to apply at the production handoff.

---

# Part 3 — Backend + static export (`nest-pg-static`, `static-site`)

## 10. The generator is the source
- **Never edit generated export output.** It is gitignored and wiped on every build. Fix
  the generator and regenerate. An edit to the output vanishes at the next build and is
  invisible in code review.
- The export is a **snapshot**. A content fix that is not followed by a rebuild and
  redeploy has not shipped.

## 11. Build correctness
- `SITE_URL` is baked into canonicals, sitemap, robots, and OG tags. Building with the
  wrong value ships canonicals pointing at the wrong host — an SEO incident, not a typo.
- Name the redirect *destination* in `SITE_URL` (if apex 301s to `www`, use `www`).
- **Never build staging without `STAGING=1`.** A staging copy without noindex competes
  with production as duplicate content.
- Root-absolute output **must** be served from a domain root. A subdirectory deploy loads
  unstyled with every internal link broken.

## 12. Database
- **Schema changes are migrations only.** Never enable TypeORM `synchronize`.
- Read generated migration SQL before applying — look for `DROP COLUMN`, `DROP TABLE`,
  and type changes implying a cast. An undetected rename is emitted as drop + add, which
  silently discards data.
- Dump before applying anything against data you care about.
- Store the model name alongside embeddings. Without it, a model swap leaves two vector
  spaces in one column with no way to tell them apart — silent, and it looks like
  "search got worse".

## 13. Deploy
- **Always `rsync --dry-run` first** and read the file list.
- `--delete` erases anything hand-placed in the docroot. `--exclude` what the host owns
  (`.htaccess`, `cgi-bin/`, `.well-known/`).
- The **trailing slash on the rsync source** means "contents of". Omitting it nests the
  build a level deep and every root-absolute asset 404s.
- On shared hosting, scope every change to one docroot — account-level settings (PHP
  version, cron, quota, SSL) affect every site on the account.
- Verify against the real host after deploy: `PW_BASE_URL=<url> npm test`, plus a
  `noindex` grep. That last check catches a staging build deployed to production — the
  highest-cost mistake in this workflow and the least visible in a browser.
- Keep the previous build until the new one is verified. It is the fastest rollback.

---

## What is NOT here (rewrite per client)
Design tokens & visual system · agent codenames · content-domain rules · partner/owner
context · anything naming a specific brand. Start those from the profile's template in
[`templates/`](templates/).
