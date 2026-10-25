---
name: static-export-build
description: Generating a static HTML site from a database-backed backend via an export script — the generator-is-the-source rule, required env vars, staging vs production builds, and root-absolute link constraints. Use when changing exported markup, adding a page type, debugging a wrong or missing URL in the export, or regenerating the site.
---

# Static Export Build

A backend renders the whole catalog to plain HTML/CSS/JS ahead of time. The result needs
no runtime — it is served by any static host.

## The one rule: never edit the export output

The output directory is generated, gitignored, and **wiped on every run**. Editing it
produces a fix that vanishes at the next build and is invisible in code review.

Every change goes into the generator script. Then regenerate.

```bash
make up                     # the generator reads from Postgres — it must be running
cd backend && npm run export:static
```

If the export runs without the database up, it either fails outright or emits an empty
catalog. Check `make ps` first when a build produces a site with no content.

## Build variants

```bash
# production
SITE_URL=https://www.example.com npm run export:static

# staging — adds noindex to every page
STAGING=1 SITE_URL=https://staging.example.com npm run export:static
```

`SITE_URL` is not cosmetic. It is baked into canonical tags, `sitemap.xml`, `robots.txt`,
Open Graph URLs, and any absolute link. Building with the wrong value ships canonicals
pointing at the wrong host, which is an SEO incident, not a typo.

Build for the host the site is actually served from. If the apex 301s to `www`, use the
`www` form — the canonical should name the destination, not the redirector.

**Never build staging without `STAGING=1`.** A staging copy without noindex competes with
production for the same content and can get the real site's rankings suppressed as
duplicate content.

## Root-absolute links

Internal links and assets are emitted root-absolute:

```html
<link rel="stylesheet" href="/styles.css">
<script src="/app.js"></script>
<a href="/courses/intro">…</a>
```

Consequence: **the export must be served from a domain root.** Dropped in a
subdirectory it loads unstyled with every internal link broken. There is no config flag
that fixes this — it is a property of the emitted markup. If a subdirectory deploy is
ever needed, the generator must be changed to emit relative paths or a configurable base.

Test locally at a root, never from a `file://` path:

```bash
npm run serve:export
```

## Adding a page type

1. Add the query and the template to the generator.
2. Emit the page at its final URL path — the URL is decided here, and changing it later
   is a redirect problem, not an edit.
3. Add it to `sitemap.xml` generation in the same change. A page absent from the sitemap
   is a page that may never be crawled.
4. Regenerate and run the link and SEO suites before calling it done.

## Debugging a wrong or missing URL

Work backwards in this order — it is almost always the first one:

1. **Is the record in the database?** The export cannot emit what it cannot query.
2. **Does the generator's query select it?** A `WHERE published = true` filter excludes
   drafts silently.
3. **Is the slug what you expect?** Slug collisions overwrite — two records mapping to
   one path means the second wins and the first disappears with no error.
4. **Was `SITE_URL` set correctly for this build?**

## The export is a snapshot

It reflects the database at build time and goes stale from that moment. Anything that
changes content requires a regenerate-and-redeploy. There is no live query path and no
cache to invalidate — a content fix that is not followed by a rebuild has not shipped.
