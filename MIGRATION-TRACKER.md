# Documentation Migration Tracker

This document tracks the migration of content from `/documentation` to `/docs` as part of the TrueNAS documentation hub consolidation project.

**Last Updated:** 2026-01-14

---

## Active Branches

Track which branches are currently being used for this migration work:

| Repository | Current Branch | Base Branch | Notes |
|------------|---------------|-------------|-------|
| `documentation` | `PD-2214-build-proposal-and-mockups-for-docs-hub-umbrella-docs-truenas-com` | `master` | Source repository for content migration |
| `connect-docs` | `PD-2214-build-proposal-and-mockups-for-docs-hub-umbrella-docs-truenas-com` | `master` | (Related work) |
| `docs-shared` | `TrueShared2026` | `master` | Shared Hugo module - shortcodes, data files, scripts added |
| `docs` | `main` | `main` | Target repository for migrated content |

---

## Redirects Needed

Pages that have been removed from `/documentation` and moved to `/docs` will need server-side redirects configured to prevent 404 errors.

**Format:** `source_path` → `destination_path` | Notes

### Pending Redirects

_No pages have been migrated yet. This section will be populated as content is moved._

Example format:
```
/docs/scale/gettingstarted/softwarestatus/ → https://www.truenas.com/docs/softwarestatus/ | Software status page moved
```

---

## Migration Progress

### Completed Transfers (2026-01-14)

**From `/documentation` to `/docs-shared`:**
- ✅ 19 shortcodes → `/layouts/shortcodes/`
- ✅ 2 menu files → `/data/menu/`
- ✅ 6 properties files → `/data/properties/`
- ✅ 7 static data files → `/static/data/`
- **Total:** 33 files

**From `/documentation` to `/docs`:**
- ✅ 2 menu files → `/data/menu/`
- ✅ 6 properties files → `/data/properties/`
- ✅ 7 static data files → `/static/data/`
- ✅ 3 scripts → `/scripts/` (modified for /docs)
- **Total:** 17 files + 1 modified script

**Grand Total:** 50 files transferred

### Content Migrations

_Content page migrations will be tracked here as they occur._

---

## Notes

- Data files were copied to both `/docs-shared` (for module sharing) and `/docs` (for software status page)
- Scripts in `/docs` were modified for the new repository structure:
  - `create-software-status-pr.sh`: Updated branch name and repository URLs
  - Other scripts work without modification
- Shortcodes remain in `/documentation` for now (available via Hugo module from docs-shared)
