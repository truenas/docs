# Documentation Migration Tracker

This document tracks the migration of content from `/documentation` to `/docs` as part of the TrueNAS documentation hub consolidation project.

**Last Updated:** 2026-01-15

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

| Source Path | Destination Path | Status | Notes |
|------------|------------------|--------|-------|
| `/SoftwareStatus/` (in documentation) | `https://www.truenas.com/docs/software-status/` | Pending | Software Status page migrated to /docs on 2026-01-15 |

---

## Migration Progress

### Completed Transfers

#### Initial Migration (2026-01-14)

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

#### Software Status Page Migration (2026-01-15)

**From `/documentation` to `/docs-shared`:**
- ✅ 3 JavaScript files → `/static/js/`
  - `linkable-tabs.js` (1418 lines - tab functionality with Mermaid support)
  - `linkable-tabs-init.js` (Hugo tab initialization)
  - `jump-to-button-fix.js` (Scroll enhancement for jump-to buttons)
- ✅ 1 CSS file → `/static/css/`
  - `custom.css` (Table styles and section box styling)
- **Total:** 4 files

**From `/documentation` to `/docs`:**
- ✅ 1 content page → `/content/en/software-status/_index.md`
- ✅ 1 favicon → `/static/favicon/software-status.png`
- ✅ 1 script modification → `update-software-status.py` (Fixed data file path to use /data/properties/)
- **Total:** 2 files + 1 script fix

**Already in Place (from previous migrations):**
- ✅ 5 shortcodes (software-status-table, user-expectations-table, deprecation-status, releaselist, tabbox)
- ✅ 4 data files (software_status_config.yaml, scale-releases.yaml, truenas-downloads.yaml, component_versions.yaml)
- ✅ 2 automation scripts (update-software-status.py, create-software-status-pr.sh)

**Grand Total:** 56 files transferred (50 initial + 6 software status)

### Content Migrations

| Page | Source | Destination | Date | Status | Notes |
|------|--------|-------------|------|--------|-------|
| Software Status | `/documentation/content/SoftwareStatus/_index.md` | `/docs/content/en/software-status/_index.md` | 2026-01-15 | ✅ Complete | Includes all dependencies (shortcodes, JS, CSS, data files). Automation scripts verified. Landing page updated to link to local path. |

---

## Notes

### Hugo Module Configuration for Production

**IMPORTANT:** Before deploying to production, remove local module replace directives:

1. **In `go.mod` (line 14):** Comment out or remove:
   ```
   replace github.com/truenas/docs-shared => ../docs-shared
   ```

2. **In `hugo.toml` (line 50):** Remove the replace line:
   ```
   replace = "../docs-shared"
   ```

3. **Update module to remote branch:**
   ```bash
   hugo mod get -u github.com/truenas/docs-shared@<branch-name>
   hugo mod tidy
   ```

These local replace directives are used for development only. Production builds must fetch the module from GitHub.

### Initial Migration (2026-01-14)
- Data files were copied to both `/docs-shared` (for module sharing) and `/docs` (for software status page)
- Scripts in `/docs` were modified for the new repository structure:
  - `create-software-status-pr.sh`: Updated branch name and repository URLs
  - Other scripts work without modification
- Shortcodes remain in `/documentation` for now (available via Hugo module from docs-shared)

### Software Status Page Migration (2026-01-15)
- **Complete Feature Migration**: All components needed for the Software Status page are now in `/docs` or `/docs-shared`
- **JavaScript Dependencies**:
  - `linkable-tabs.js` provides interactive tab functionality with URL hash support and theme-aware Mermaid diagram rendering
  - `linkable-tabs-init.js` initializes tabs from Hugo-processed content
  - `jump-to-button-fix.js` enhances scroll behavior for jump-to buttons
- **CSS Styling**: Custom table styles (`.truetable`) and section boxes (`.section-box`) with dark mode support
- **Automation Scripts**:
  - `update-software-status.py` fixed to use correct data path: `/data/properties/software_status_config.yaml`
  - `create-software-status-pr.sh` already configured to target `main` branch
  - Scripts continue to work via GitHub Actions/Jenkins for automated version updates
- **Content Structure**:
  - Page uses lowercase directory name (`/software-status/`) for consistency with `/docs` conventions
  - Landing page (`_index.md`) updated to link to Software Status with local path (`/software-status/`)
  - All shortcode references verified and working
- **Data Flow**: TrueNAS Update API → Python script → YAML file → Hugo build → Static page
- **Migration Strategy**: Shortcodes and data files were already migrated previously, only needed to add page-specific JavaScript, CSS, content, and fix script path
