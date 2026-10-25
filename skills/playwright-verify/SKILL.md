---
name: playwright-verify
description: Playwright verification suites for a static site — accessibility, responsive, links, performance, SEO, and visual regression, runnable against a local build or the live domain. Use when verifying a change before or after deploy, adding a check, updating visual snapshots, or diagnosing a failing suite.
---

# Playwright Verification Suites

Six suites, each answering one question. Run the whole set before a deploy; run one
while iterating.

```bash
npm test                  # everything
npm run test:a11y         # accessibility (axe)
npm run test:responsive   # layout across viewports
npm run test:links        # internal links resolve, no 404s
npm run test:perf         # load timing budgets
npm run test:seo          # titles, canonicals, meta, sitemap
npm run test:visual       # screenshot regression
```

## Target selection

Tests run against a base URL. Default is a local server; point them anywhere with an
env var:

```bash
npm run serve:export                                  # serve the local build
npm test                                              # → local

PW_BASE_URL=https://www.example.com npm test          # → live site
PW_BASE_URL=https://staging.example.com npm test      # → staging
```

Running against live is the real post-deploy check — it exercises the actual host,
redirects, TLS, and caching, none of which a local server reproduces. It is read-only
and safe. Do it after every deploy.

## Visual regression

Snapshots are committed. A visual failure is a diff to look at, not a test to silence.

```bash
npm run test:visual
npm run test:visual:update    # accept the new baseline — ONLY after eyeballing the diff
```

Never run `--update-snapshots` to make a red suite green. It overwrites the record of
what the site is supposed to look like, and the regression ships. Open the diff image in
`test-results/` first and confirm the change was intended.

Snapshots are platform-sensitive: font rendering differs between Linux, macOS, and CI.
Baselines generated on one platform will produce spurious diffs on another. Generate
them where they will be verified.

## Reading a failure

`test-results/` holds the artifacts — screenshots, diffs, traces, and the error context.
Start there rather than re-running with more logging.

```bash
npx playwright show-report      # HTML report
npx playwright test --headed    # watch it run
npx playwright test --debug     # step through with the inspector
```

Failures cluster into recognizable kinds:

- **a11y** — axe names the rule and the offending selector. Fix the markup in the
  generator, not the test.
- **links** — a 404 usually means the generator emitted a link to a page it did not
  emit, or a slug changed without a redirect.
- **seo** — a missing canonical or a wrong host almost always means the build used the
  wrong `SITE_URL`. Rebuild before investigating the test.
- **perf** — check whether the budget is genuinely exceeded or the machine is loaded.
  Perf assertions on a busy laptop are noisy; confirm against the live site.
- **visual** — inspect the diff. Anti-aliasing noise means the threshold needs raising;
  a shifted layout means the CSS changed.

## Adding a check

Put it in the suite that matches the question it answers, so it runs with that suite's
targeted command. A check that lives in the wrong suite gets skipped by everyone
iterating on the right one.

Assert on user-visible outcomes — rendered text, resolved URLs, computed styles — not on
internal markup structure. Tests coupled to DOM shape break on every refactor and teach
people to ignore red.

## Flake

Before assuming flake, run the single test three times. Real flake is almost always a
missing wait: assert on a condition (`toBeVisible`, `toHaveURL`) rather than a fixed
timeout. Never add `waitForTimeout` to stabilize a test — it slows the suite and hides
the actual race.
