---
name: deploy-static-cpanel
description: Deploying a static export to shared cPanel hosting over rsync/SSH — pre-flight checks, safe sync flags, domain-root and multi-site constraints, and post-deploy verification. Use when publishing a build, setting up a deploy target, or diagnosing a deploy that produced an unstyled or stale site.
---

# Deploying a Static Export to cPanel

Shared cPanel hosting has no Docker, no root, and a jailed shell. It can serve static
HTML perfectly well; it cannot run application infrastructure. Only the export goes here.

## Before you sync

1. **Build with the production host.** Canonicals are baked in at build time.
   ```bash
   SITE_URL=https://www.example.com npm run export:static
   ```
   Confirm you did not just build a `STAGING=1` copy — deploying a noindex build to
   production removes the site from search results.

2. **Verify locally.**
   ```bash
   npm run serve:export && npm test
   ```

3. **Know your docroot.** Shared accounts commonly host many unrelated sites. Anything
   account-level — PHP version, cron, quota, SSL — affects all of them. Scope every
   change to one docroot, and confirm the path before writing to it.

## The sync

Dry run first, always. Read the file list before letting it write.

```bash
rsync -avz --dry-run --delete \
  ./static-export/ user@host:~/www.example.com/
```

Then, with the output confirmed:

```bash
rsync -avz --delete \
  ./static-export/ user@host:~/www.example.com/
```

Flag notes:

- The **trailing slash on the source** means "contents of", not "the directory itself".
  Omitting it nests the build one level deep and every root-absolute asset 404s.
- `--delete` removes remote files absent from the build. It is correct for a generated
  site and dangerous everywhere else — it will erase anything hand-placed in that
  docroot. This is exactly why the dry run is not optional.
- Add `--exclude` for anything the host owns and the build does not produce:
  `.htaccess`, `cgi-bin/`, `.well-known/`. Without these, `--delete` removes them.

```bash
rsync -avz --delete \
  --exclude '.htaccess' --exclude 'cgi-bin/' --exclude '.well-known/' \
  ./static-export/ user@host:~/www.example.com/
```

## Domain root is mandatory

The export uses root-absolute paths. It works at a domain or subdomain root only. In a
subdirectory it loads with no CSS and every internal link broken — a deploy that looks
catastrophically wrong but is simply in the wrong place.

## Verify against the real host

Local passing tests do not prove the deploy landed. Run the suites against production:

```bash
PW_BASE_URL=https://www.example.com npm test
```

And spot-check the things only the live host can get wrong:

```bash
curl -sI https://www.example.com | head -20              # 200, content-type
curl -sI https://example.com | grep -i location          # apex → www 301
curl -s https://www.example.com/sitemap.xml | head       # sitemap served
curl -s https://www.example.com | grep -i canonical      # correct host
curl -s https://www.example.com | grep -i noindex        # MUST be empty
```

That last one catches a staging build deployed to production — the highest-cost mistake
in this workflow and the least visible in a browser.

## When the site looks wrong after a deploy

- **Unstyled, links 404** — nesting. The source trailing slash was omitted, or it went to
  a subdirectory. Check the remote tree: `ssh host 'ls ~/www.example.com | head'`.
- **Stale content** — the build was not regenerated, or a CDN/host cache is serving old
  files. Confirm the local build is current before blaming cache.
- **Some pages 404** — the generator did not emit them. That is a build problem, not a
  deploy problem; check the export output locally first.
- **Mixed content warnings** — an absolute `http://` URL baked in at build time. Rebuild
  with the `https://` `SITE_URL`.

## Rollback

Keep the previous build. The fastest rollback for a static site is re-syncing the last
known-good directory — so do not delete it until the new one is verified.

Get explicit sign-off before any production-facing deploy on a shared account. The blast
radius includes sites that are not yours.
