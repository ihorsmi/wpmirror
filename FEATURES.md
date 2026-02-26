# WP Mirror capability analysis and improvement plan

This document summarizes the current plugin capabilities and the improvements added to better support practical static publishing workflows.

## 1) Static generation + publishing options

### Current capabilities

- ✅ One-click export from WP Admin (`Generate Static Export`).
- ✅ ZIP archive generation and secure download from admin UI.
- ✅ Local directory publishing by writing the static output to a configured export directory.
- ✅ GitHub API deploy flow (compatible with GitHub Pages and Git-based hosting pipelines).

### Improvements included in this update

- ✅ Added a machine-readable capabilities manifest emitted on each export:
  - `.wp-mirror-capabilities.json`
  - Documents first-party and integration-friendly publish targets.
- ✅ Manifest now documents common GitHub-driven hosting paths:
  - GitHub Pages, Cloudflare Pages, Netlify (via Git integration).
- ✅ Manifest now documents common static/CDN destinations as deployment patterns:
  - AWS S3, Bunny CDN, DigitalOcean App Platform, Kinsta Static Site Hosting, Tiiny.host.

### Practical deployment notes

- ZIP export: share/upload archive to any host that accepts static bundles.
- Local directory: serve directly from Nginx/Apache/CDN sync jobs.
- SFTP: upload the export directory with your preferred SFTP client/automation.
- GitHub-driven hosts: use WP Mirror deploy to push a static branch, then connect host to that repo.

## 2) "Static but still functional" (dynamic feature patterns)

### Current capabilities

- WP Mirror focuses on static rendering/export/deploy and avoids server runtime coupling.
- Integrations for comments/forms/search were not previously generated as first-class output artifacts.

### Improvements included in this update

- ✅ Added generated search index output on export:
  - `search-index.json` contains title/url/excerpt/date/type for posts and pages.
  - Enables client-side search (Fuse.js) and can also be used as an indexing feed.
- ✅ Added generated capabilities guidance output:
  - `.wp-mirror-capabilities.json` now records integration patterns for comments/forms/search.

### Recommended implementation patterns

- Comments:
  - Use external embeddable providers (script embed) on exported pages.
- Forms:
  - Post submissions to an external endpoint (Formspree/webhooks/etc.).
  - Works with embedded frontend forms from plugins/builders when action targets external service.
- Search:
  - Fuse.js: load `search-index.json` client-side for instant local search.
  - Algolia: feed/export content into Algolia indices and query via hosted search API.

## Example: Fuse.js integration sketch

1. Include Fuse.js in your static theme/template.
2. Fetch `/search-index.json`.
3. Build a Fuse index with keys like `title`, `excerpt`, `url`.
4. Render result links to exported URLs.

