# WP Mirror — Repository Analysis & Industry-Standards Improvement Plan

## 1) Current state (what is already solid)

Based on the current codebase, the plugin already has a strong technical baseline:

- **Background jobs with staged processing** (export, ZIP, deploy, restore), reducing timeout risk in wp-admin.  
- **Static export pipeline** with URL discovery, HTML fetch/rewrite, asset collection/copy, manifest generation, and optional ZIP.  
- **GitHub Pages-oriented deployment** using Git Data API primitives (blob/tree/commit/ref), with rate-limit pause handling and skip-unchanged via manifest.  
- **ZIP lifecycle support** (archive creation/list/download/delete) and **restore from ZIP** using staged extraction and replacement.
- **Security-conscious defaults** like capability checks, nonces, and sanitization in settings/actions.

## 2) Functional-fit assessment vs your requested capabilities

Your requested functional scope:

1. Export to ZIP  
2. Export to GitHub Pages as static HTML, with only needed data and safe thresholds for API limits  
3. Import ZIP back / restore from ZIP  
4. Improve UX/UI to industry best practices and WordPress standards

### Assessment

- **(1) Export to ZIP**: **implemented** and production-usable.
- **(2) GitHub Pages static export**: **implemented**, but threshold strategy can be improved from “reactive pause on limit” to “proactive budgeted deploy”.
- **(3) Restore from ZIP**: **implemented** with background stages, but preflight integrity/safety checks can be expanded.
- **(4) UX/UI best practices**: current UI is functional, but needs WordPress-native information architecture, accessibility, and operational guidance patterns.

## 3) Gap analysis (industry-standard perspective)

## A. Export/Restore reliability gaps

- No visible **archive integrity metadata** (hashes, origin site, WP/plugin version, created-at, export profile).
- Restore workflow would benefit from explicit **preflight validation**:
  - ZIP format and structure checks
  - Maximum file count / size checks
  - Disk-space estimate before extraction
  - Compatibility checks (plugin version schema)
- No explicit **transactional rollback marker** if restore fails mid-way (current approach has backup/replace stages but should surface recoverability guarantees in UI).

## B. GitHub Pages deploy efficiency and API-safety gaps

- Current behavior pauses when rate-limited, but does not enforce a **pre-commit API budget policy**.
- No user-facing **recommended performance profiles** (“Small site”, “Medium”, “Large”) tied to safe API thresholds.
- No explicit handling policy for **oversized files** for Git blobs (should fail early with actionable guidance).

## C. UX/UI + WordPress standards gaps

- Single-screen density is high; users need **task-first workflow** (Configure → Export → Validate → Package → Deploy → Restore).
- Progress/logging is useful but could be upgraded with:
  - stage-specific ETA,
  - actionable remediation cards,
  - downloadable machine-readable job report.
- Accessibility and WP admin consistency should be strengthened:
  - better semantic headings/fieldsets/help text,
  - notice styles and status chips aligned to WP patterns,
  - keyboard and screen-reader flow validation.

## 4) Target product behavior (industry-standard definition)

## 4.1 Export to ZIP (gold standard)

- Always generate:
  - static files,
  - `manifest.json` (path → checksum, size, modified time),
  - `export-metadata.json` (site URL, WP version, plugin version, timestamp, profile, options snapshot sans secrets).
- Optional integrity mode:
  - SHA-256 checksum file per archive,
  - one-click verify in UI before restore.
- Archive naming convention:
  - `wp-mirror-export-YYYYmmdd-HHMMSS-profile.zip`.

## 4.2 GitHub Pages deploy with safe thresholds

- Introduce **deploy guardrails**:
  - hard-stop if projected requests exceed budget,
  - soft warning if remaining API quota below reserve.
- Add **recommended defaults** for authenticated GitHub API usage:
  - reserve floor: **200 requests** (never consume below this),
  - per-run budget cap: **<= 3,000 requests**,
  - per-batch file target: **10–25 files** (adaptive by average file size/error rate),
  - auto-pause when remaining quota `< reserve floor`.
- Include **preflight estimate**:
  - estimated blobs + tree/commit/ref operations,
  - estimated runtime and number of ticks,
  - skipped unchanged files count.
- Keep output minimal for Pages:
  - default `referenced` asset mode,
  - optional `exclude patterns` presets for common WP/cache artifacts,
  - block PHP/system files by policy.

## 4.3 Import/restore from ZIP (production-safe)

- Preflight checks before execution:
  - archive integrity verify (if checksum available),
  - path traversal protection and allowed-path rules,
  - file count and total size thresholds,
  - writable/space checks.
- Safe restore strategy:
  1. backup existing export dir,
  2. extract to temp dir,
  3. validate extracted tree,
  4. atomic swap,
  5. post-restore verification,
  6. rollback automatically on validation failure.
- User-facing restore report with summary and failed file list.

## 4.4 UX/UI to WordPress best practices

- Rebuild admin page into tabs:
  - **Overview**
  - **Export**
  - **Archives (ZIP + Restore)**
  - **GitHub Deploy**
  - **Diagnostics**
- Add guided setup checklist with readiness indicators:
  - export path writable,
  - cron available,
  - ZipArchive available,
  - GitHub token scope check,
  - rate-limit headroom.
- Add profile presets:
  - **Safe/Default** (small batch, conservative API usage),
  - **Balanced**,
  - **Fast** (for higher-resource hosts).
- Improve logs UX:
  - filter by level (info/warn/error),
  - copy/download logs,
  - jump-to-error anchor.

## 5) Implementation roadmap (phased)

## Phase 1 — Foundation hardening (1–2 sprints)

- Add archive/export metadata and checksums.
- Add restore preflight validator service and UI warnings.
- Add deploy preflight estimator and API budget policy.
- Add recommended profile presets mapped to settings fields.

**Exit criteria:** users can see “safe-to-run” checks before export/deploy/restore.

## Phase 2 — UX modernization + WP standards alignment (1–2 sprints)

- Refactor admin screen into tabbed IA.
- Upgrade settings/help text/notices to WordPress admin UX conventions.
- Accessibility pass (labels, aria-live for progress, focus management, contrast).

**Exit criteria:** setup/deploy/restore completed by new user without docs.

## Phase 3 — Operational excellence (1 sprint)

- Add structured job reports (JSON) and downloadable diagnostics bundle.
- Add retry strategies with reason-specific backoff (network/5xx/rate-limit).
- Add “dry run” deploy mode (estimate + diff, no writes).

**Exit criteria:** predictable operations and support-friendly observability.

## 6) Concrete engineering tasks (backlog-ready)

1. **Create `WPMirror_Deploy_Estimator`**
   - input: manifest diff + settings
   - output: projected requests, batches, estimated minutes, risk level
2. **Create `WPMirror_Restore_Preflight`**
   - zip validation, size/count thresholds, path safety, disk checks
3. **Add metadata/checksum writer + verifier**
   - used by ZIP build and restore
4. **Add settings profiles + constraints**
   - single selector writes batch/timeouts defaults
5. **Admin UI IA refactor**
   - tabs + status cards + diagnostics section
6. **Accessibility and i18n review pass**
   - ARIA live regions for progress
   - ensure all user-visible strings are translatable

## 7) Suggested default thresholds (starting point)

These should be configurable, but sensible defaults are:

- `export_batch_urls`: **5** (current default is good)
- `asset_batch_files`: **25** (current default is good)
- `zip_batch_files`: **200** (current default is good)
- `github_batch_files`: **15** (current default is good baseline)
- API reserve floor: **200** requests
- API per-run budget cap: **3000** requests
- Preflight warning if projected requests > **70%** of remaining quota
- Hard stop if projected requests exceed `(remaining - reserve floor)`

## 8) KPIs to measure improvement

- Export success rate (% jobs completed without manual retry)
- Deploy success rate on first run
- Mean deploy duration
- Restore rollback incidence
- User setup completion time (first successful deploy)
- Accessibility audit score for admin screens

---

## Final recommendation

The plugin already implements the core functional set you requested. The best next step is **not feature reinvention**; it is **operational hardening + UX modernization**:

1. Add preflight safety and API budgeting.
2. Add archive integrity metadata and safer restore verification.
3. Refactor admin UX into guided, WordPress-native workflows.

This path delivers enterprise-grade reliability while preserving the current architecture and minimizing regression risk.
