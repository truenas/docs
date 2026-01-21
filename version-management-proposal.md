# TrueNAS Documentation Version Management Strategy

**Status:** Proposal
**Last Updated:** 2026-01-21
**Target Implementation:** TrueNAS 26 Release

---

## Executive Summary

### Current Problem Statement

The TrueNAS documentation currently serves nightly/development content at the root URL (`truenas.com/docs/`), creating confusion for users who cannot determine which version they're viewing or access documentation for specific stable releases. With no clear version navigation and complex Jenkins symlink configurations, both users and maintainers face significant pain points.

**Key Issues:**
- Users viewing nightly docs by default without realizing it
- No version navigation or discoverability
- Complex Jenkins builds with hard-to-maintain symlink configurations
- Difficult to determine which docs match deployed TrueNAS versions

### Proposed Solution Overview

Implement a **branch-per-version architecture** starting with TrueNAS 26, with versioned URLs under the `/tn/` namespace:
- Stable versions at `docs.truenas.com/tn/26/`, `/tn/27/`, etc.
- Nightly builds at `docs.truenas.com/tn/nightly/`
- Symlink at `/tn/current/` pointing to latest stable
- Root redirect (`/docs/`) → `/docs/tn/current/` for discoverability
- Branch-per-version preserves existing backport automation

### Key Benefits

- ✅ **User Clarity**: Clear version indicators and navigation
- ✅ **Discoverability**: Root URL redirects to current stable version
- ✅ **Maintainability**: Symlink approach minimizes nginx configuration changes
- ✅ **Preservation**: Existing backport automation continues to work
- ✅ **Scalability**: Archive strategy manages build times as versions accumulate
- ✅ **Consistency**: Follows industry standards (Kubernetes, Django, etc.)

---

## Current State Analysis

### How `/documentation` Repo Handles Versions

**Current Architecture:**
```
/documentation/
├── master branch → builds to docs.truenas.com/docs/ (nightly)
├── 24.10.1 branch → builds to enterprise.truenas.com/docs/ (via symlink)
├── 24.10 branch → (maintenance)
├── 24.04.2 branch → builds to core.truenas.com/docs/ (via symlink)
└── jenkins/ → complex build configurations
```

**Current URL Structure:**
- `truenas.com/docs/*` → nightly/development content
- `enterprise.truenas.com/docs/*` → TrueNAS 24.10.1 (Enterprise)
- `core.truenas.com/docs/*` → TrueNAS 24.04.2 (Community Edition)

**Current Jenkins Approach:**
- Multiple Jenkins jobs per branch
- Symlinks managed via Jenkins pipelines
- Complex deploy.py scripts with hard-coded paths
- Builds scattered across domain subfolders

### Pain Points and User Confusion

| Issue | Impact | Severity |
|-------|--------|----------|
| Root URL serves nightly docs | Users on stable releases read incorrect documentation | ⚠️ **High** |
| No version selector | Users cannot navigate between versions | ⚠️ **High** |
| No version indicators | Users don't know what version they're viewing | ⚠️ **High** |
| Domain-based versioning (core vs enterprise) | Confusing naming as products converge | ⚠️ **Medium** |
| Complex symlink configurations | Maintenance burden, error-prone | ⚠️ **Medium** |
| Multiple Jenkins builds per branch | Resource intensive, slow iteration | ⚠️ **Medium** |

### Maintenance Burdens

**For Documentation Team:**
- Must remember which branch builds to which domain
- Backporting requires awareness of branch-to-URL mappings
- No clear process for archiving old versions
- Difficult to test version-specific content

**For IT/DevOps:**
- Jenkins configuration changes require deep system knowledge
- Symlink management across domains prone to errors
- Nginx configurations scattered across multiple files
- Difficult to add new versions without breaking existing builds

---

## Proposed Architecture

### Branch-Per-Version Structure

Starting with TrueNAS 26, each major version gets its own branch:

```
/docs/
├── main branch → /tn/nightly/ (development)
├── tn-26 branch → /tn/26/ (stable)
├── tn-27 branch → /tn/27/ (when released)
├── tn-28 branch → /tn/28/ (when released)
└── archived branches (no longer build)
```

**Branch Naming Convention:**
- `main` → nightly/development builds
- `tn-26`, `tn-27`, `tn-28` → stable version branches
- Format: `tn-<major-version>` (not `tn-26.04`, just `tn-26`)

**Version Release Cadence:**
- Yearly major versions: 26, 27, 28 (not 26.04, 26.10)
- Point releases (26.1, 26.2) update the `tn-26` branch
- Documentation versioned by major version only

### URL Scheme with /tn/ Prefix

All versioned documentation lives under the `/tn/` namespace for clean organization:

```
https://docs.truenas.com/tn/26/          → TrueNAS 26 (stable)
https://docs.truenas.com/tn/27/          → TrueNAS 27 (stable)
https://docs.truenas.com/tn/nightly/     → Development/nightly
https://docs.truenas.com/tn/current/     → Symlink to latest stable
```

**Benefits of `/tn/` Namespace:**
- Clear separation from other content (API docs, Apps docs, etc.)
- Room for future product lines without URL conflicts
- Follows established patterns (e.g., Kubernetes uses `/docs/v1.x/`)
- Makes version explicit in URL for support troubleshooting

### Symlink Strategy for /current/

Instead of hard-coding nginx configurations for "current version," use a filesystem symlink:

```bash
/var/www/docs.truenas.com/tn/
├── 26/            (directory)
├── 27/            (directory)
├── nightly/       (directory)
└── current -> 27  (symlink)
```

**Updating "Current" Version:**
```bash
cd /var/www/docs.truenas.com/tn/
ln -sfn 27 current  # Update to new version
```

**Nginx Configuration:**
```nginx
# No special configuration needed - symlink is transparent
location /tn/ {
    try_files $uri $uri/ =404;
}
```

**Benefits:**
- No nginx reloads required when updating current version
- IT can update current version without code changes
- Zero downtime updates
- Same approach works for all environments (dev, staging, prod)

### Root Redirect Configuration

Solve the discoverability problem by redirecting root docs URL to current stable version:

```nginx
# Redirect root to current stable version
location = /docs/ {
    return 302 /docs/tn/current/;
}

# Redirect common unversioned paths to current version
location ~ ^/docs/(getting-started|installation|administration|troubleshooting)(/.*)?$ {
    return 302 /docs/tn/current/$1$2;
}

# Versioned paths work as-is
location /docs/tn/ {
    try_files $uri $uri/ =404;
}
```

**User Experience:**
1. User visits `truenas.com/docs/` → Redirected to `truenas.com/docs/tn/current/` → Sees latest stable
2. User visits `truenas.com/docs/tn/26/` → Sees TrueNAS 26 docs
3. User visits `truenas.com/docs/tn/nightly/` → Sees development docs

**Old URL Handling:**
- Old unversioned URLs redirect to current stable version
- Explicit version URLs (if linked from old sources) can use redirects
- Gradual migration, not big-bang cutover

---

## Implementation Details

### Jenkins Build Configuration Per Branch

Each branch gets a dedicated Jenkins build job:

**Jenkins Job Structure:**
```
TrueNAS-Docs-Build-TN26      → tn-26 branch
TrueNAS-Docs-Build-TN27      → tn-27 branch
TrueNAS-Docs-Build-Nightly   → main branch
```

**Build Command (Example for TN26):**
```bash
#!/bin/bash
set -e

# Update Hugo modules
hugo mod get -u
hugo mod tidy

# Build with version-specific baseURL
hugo --gc --minify --cleanDestinationDir \
     --baseURL="https://docs.truenas.com/tn/26/" \
     --destination="/var/www/docs.truenas.com/tn/26/"

# Optional: Clear CDN cache for this version
curl -X POST "https://cdn.example.com/purge/docs/tn/26/"
```

**Environment Variables Per Job:**
```bash
VERSION="26"
BASE_URL="https://docs.truenas.com/tn/26/"
DEPLOY_PATH="/var/www/docs.truenas.com/tn/26/"
BRANCH="tn-26"
```

**Jenkinsfile Template:**
```groovy
pipeline {
    agent any

    environment {
        VERSION = "26"
        BASE_URL = "https://docs.truenas.com/tn/26/"
        DEPLOY_PATH = "/var/www/docs.truenas.com/tn/26/"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'tn-26', url: 'https://github.com/truenas/docs'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    hugo mod get -u
                    hugo mod tidy
                    hugo --gc --minify --cleanDestinationDir \
                         --baseURL="${BASE_URL}" \
                         --destination="${DEPLOY_PATH}"
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    # Deployment already done by Hugo destination
                    # Optional: Set permissions, clear caches, etc.
                    chmod -R 755 "${DEPLOY_PATH}"
                '''
            }
        }
    }

    post {
        success {
            echo "TrueNAS ${VERSION} documentation built successfully"
        }
        failure {
            emailext to: 'docs-team@truenas.com',
                     subject: "Jenkins: TN${VERSION} docs build failed",
                     body: "Build failed. Check Jenkins for details."
        }
    }
}
```

### Nginx Configuration with Symlinks

**Complete Nginx Configuration Block:**
```nginx
# TrueNAS Documentation
server {
    listen 443 ssl http2;
    server_name docs.truenas.com;

    # SSL configuration
    ssl_certificate /etc/ssl/certs/truenas.crt;
    ssl_certificate_key /etc/ssl/private/truenas.key;

    root /var/www/docs.truenas.com;
    index index.html;

    # Redirect root to current stable version
    location = /docs/ {
        return 302 /docs/tn/current/;
    }

    # Redirect old unversioned paths to current
    location ~ ^/docs/(getting-started|installation|administration|troubleshooting|references)(/.*)?$ {
        return 302 /docs/tn/current/$1$2;
    }

    # Serve versioned documentation (symlinks work transparently)
    location /docs/tn/ {
        try_files $uri $uri/ $uri/index.html =404;

        # Optional: Add version header for debugging
        add_header X-Docs-Version $uri;
    }

    # Legacy domain redirects (maintain old links)
    location /docs/ {
        try_files $uri $uri/ $uri/index.html =404;
    }

    # API docs continue to work
    location /api/ {
        try_files $uri $uri/ $uri/index.html =404;
    }

    # Apps docs continue to work
    location /apps/ {
        try_files $uri $uri/ $uri/index.html =404;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name docs.truenas.com;
    return 301 https://$server_name$request_uri;
}
```

**Symlink Management Script:**
```bash
#!/bin/bash
# update-current-version.sh
# Updates the /tn/current/ symlink to point to the latest stable version

set -e

VERSION="${1}"
DOCS_ROOT="/var/www/docs.truenas.com/tn"

if [ -z "$VERSION" ]; then
    echo "Usage: $0 <version>"
    echo "Example: $0 27"
    exit 1
fi

if [ ! -d "${DOCS_ROOT}/${VERSION}" ]; then
    echo "Error: Version directory ${DOCS_ROOT}/${VERSION} does not exist"
    exit 1
fi

cd "$DOCS_ROOT"

# Create symlink (force overwrite if exists)
ln -sfn "$VERSION" current

echo "✅ Updated /tn/current/ → ${VERSION}"
echo "Current version is now TrueNAS ${VERSION}"
```

### Version Dropdown Configuration

**Hugo Configuration (hugo.toml):**
```toml
[params]
  # Current version of these docs
  version = "26"

  # Version dropdown configuration
  [[params.versions]]
    version = "Nightly (27-dev)"
    url = "/tn/nightly/"
    lifecycle = "next"

  [[params.versions]]
    version = "27 (Current)"
    url = "/tn/27/"
    lifecycle = "current"

  [[params.versions]]
    version = "26 (Previous)"
    url = "/tn/26/"
    lifecycle = "previous"

  [[params.versions]]
    version = "25 (Archived)"
    url = "/tn/25/"
    lifecycle = "archived"
```

**Version Selector Shortcode (layouts/shortcodes/version-selector.html):**
```html
{{ $currentVersion := .Site.Params.version }}
{{ $versions := .Site.Params.versions }}

<div class="version-selector">
  <div class="version-dropdown">
    <button class="version-button" aria-haspopup="true" aria-expanded="false">
      <span class="version-icon">📖</span>
      <span class="version-text">Version: {{ $currentVersion }}</span>
      <span class="dropdown-icon">▼</span>
    </button>

    <div class="version-menu" role="menu">
      {{ range $versions }}
        <a href="{{ .url }}"
           class="version-item version-{{ .lifecycle }}"
           role="menuitem">
          <span class="version-name">{{ .version }}</span>
          <span class="version-badge badge-{{ .lifecycle }}">
            {{ if eq .lifecycle "current" }}Current{{ end }}
            {{ if eq .lifecycle "previous" }}Previous{{ end }}
            {{ if eq .lifecycle "next" }}Development{{ end }}
            {{ if eq .lifecycle "archived" }}Archived{{ end }}
          </span>
        </a>
      {{ end }}
    </div>
  </div>
</div>

<style>
.version-selector {
  position: relative;
  display: inline-block;
}

.version-button {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: #485d6b;
  color: #f1f3f4;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 500;
  transition: background 0.2s;
}

.version-button:hover {
  background: #5a7080;
}

.version-menu {
  display: none;
  position: absolute;
  top: 100%;
  left: 0;
  margin-top: 4px;
  min-width: 200px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 6px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  z-index: 1000;
}

.version-dropdown:hover .version-menu,
.version-dropdown:focus-within .version-menu {
  display: block;
}

.version-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 16px;
  color: #333;
  text-decoration: none;
  border-bottom: 1px solid #eee;
  transition: background 0.2s;
}

.version-item:last-child {
  border-bottom: none;
}

.version-item:hover {
  background: #f5f5f5;
}

.version-badge {
  font-size: 0.75rem;
  padding: 2px 8px;
  border-radius: 12px;
  font-weight: 600;
}

.badge-current {
  background: #73bf44;
  color: white;
}

.badge-previous {
  background: #aeaeae;
  color: white;
}

.badge-next {
  background: #0795d3;
  color: white;
}

.badge-archived {
  background: #6c757d;
  color: white;
}

/* Dark mode support */
@media (prefers-color-scheme: dark) {
  .version-menu {
    background: #2d3748;
    border-color: #4a5568;
  }

  .version-item {
    color: #e2e8f0;
    border-bottom-color: #4a5568;
  }

  .version-item:hover {
    background: #4a5568;
  }
}
</style>

<script>
document.addEventListener('DOMContentLoaded', function() {
  const versionButton = document.querySelector('.version-button');
  const versionMenu = document.querySelector('.version-menu');

  if (versionButton) {
    versionButton.addEventListener('click', function(e) {
      e.stopPropagation();
      const expanded = this.getAttribute('aria-expanded') === 'true';
      this.setAttribute('aria-expanded', !expanded);
      versionMenu.style.display = expanded ? 'none' : 'block';
    });
  }

  // Close dropdown when clicking outside
  document.addEventListener('click', function() {
    if (versionMenu) {
      versionMenu.style.display = 'none';
    }
    if (versionButton) {
      versionButton.setAttribute('aria-expanded', 'false');
    }
  });
});
</script>
```

### Version Awareness Banners

**Outdated Version Banner (layouts/partials/version-banner.html):**
```html
{{ $currentVersion := .Site.Params.version }}
{{ $versions := .Site.Params.versions }}

{{/* Find if this is the current version */}}
{{ $isCurrent := false }}
{{ $isArchived := false }}
{{ $latestVersion := "" }}

{{ range $versions }}
  {{ if eq .lifecycle "current" }}
    {{ $latestVersion = .version }}
    {{ if eq $currentVersion .version }}
      {{ $isCurrent = true }}
    {{ end }}
  {{ end }}
  {{ if and (eq .lifecycle "archived") (eq $currentVersion .version) }}
    {{ $isArchived = true }}
  {{ end }}
{{ end }}

{{/* Show banner if viewing non-current version */}}
{{ if and (not $isCurrent) $latestVersion }}
  <div class="version-warning-banner {{ if $isArchived }}archived{{ else }}outdated{{ end }}">
    <div class="banner-content">
      {{ if $isArchived }}
        <span class="banner-icon">🗄️</span>
        <span class="banner-text">
          You're viewing archived documentation for TrueNAS {{ $currentVersion }}.
          This version is no longer supported.
        </span>
      {{ else }}
        <span class="banner-icon">⚠️</span>
        <span class="banner-text">
          You're viewing documentation for TrueNAS {{ $currentVersion }}.
          The current stable version is {{ $latestVersion }}.
        </span>
      {{ end }}
      <a href="/tn/current/" class="banner-link">View Current Docs →</a>
    </div>
  </div>
{{ end }}

<style>
.version-warning-banner {
  padding: 12px 20px;
  background: #fff3cd;
  border-bottom: 2px solid #ffc107;
  color: #856404;
}

.version-warning-banner.archived {
  background: #f8d7da;
  border-bottom-color: #dc3545;
  color: #721c24;
}

.banner-content {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.banner-icon {
  font-size: 1.2rem;
}

.banner-text {
  flex: 1;
  font-weight: 500;
}

.banner-link {
  padding: 6px 16px;
  background: white;
  color: #0795d3;
  text-decoration: none;
  border-radius: 4px;
  font-weight: 600;
  transition: background 0.2s;
}

.banner-link:hover {
  background: #f0f0f0;
}

@media (max-width: 768px) {
  .banner-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
}
</style>
```

---

## User Experience Improvements

### Version Navigation and Discovery

**Primary Navigation:**
1. **Version Selector Dropdown**: Always visible in header, shows current version
2. **Version Banner**: Warning when viewing non-current version
3. **Footer Version Info**: Displays version and last updated date
4. **Root Redirect**: Default entry point shows current stable version

**Navigation Flow:**
```
User Entry Points:
├── truenas.com/docs/ → Redirect → /tn/current/ (discovers latest stable)
├── truenas.com/docs/tn/26/ → Direct version access (bookmarks, search results)
├── truenas.com/docs/tn/nightly/ → Development docs (contributors, early adopters)
└── Version dropdown → Quick switching between versions
```

**Benefits:**
- Users default to stable docs unless explicitly choosing otherwise
- Clear version context at all times
- Easy switching between versions
- Bookmarkable version-specific URLs

### Visual Indicators for Version Context

**Header Treatment:**
```
┌─────────────────────────────────────────────────────┐
│  🏠 TrueNAS Docs Hub  │  📖 Version: 26 ▼  │  🔍  │
├─────────────────────────────────────────────────────┤
│  ⚠️ You're viewing docs for TrueNAS 26. Current     │
│  stable version is 27. [View Current Docs →]       │
└─────────────────────────────────────────────────────┘
```

**Version Badge in Search Results:**
- Each search result shows version badge
- Filter search by version (optional)
- Version-aware search ranking (prefer current version)

**Footer Treatment:**
```
─────────────────────────────────────────────────────
TrueNAS 26 Documentation | Last Updated: 2026-01-15
Generated from branch: tn-26 | Commit: abc123d
```

### Search Integration Considerations

**Current Search Implementation:**
- Uses custom search with version filtering
- `showVersionFilter = true` in hugo.toml
- `siteKey = "docs"` differentiates from other sites

**Version-Aware Search Strategy:**

**Option A: Single Search Index (Recommended)**
- Index all versions in one search database
- Add `version` field to each document
- Enable version filter in search UI
- Rank current version results higher by default

**Option B: Separate Indexes Per Version**
- Each version has its own search index
- Search only within current version by default
- Option to "Search all versions"
- More complex, higher maintenance

**Implementation (Option A):**
```javascript
// Add version to search index
const searchConfig = {
  indexConfig: {
    fields: {
      title: { boost: 2 },
      content: { boost: 1 },
      version: { boost: 0 }, // For filtering, not ranking
      tags: { boost: 1.5 }
    }
  },

  // Default filter to current version
  defaultFilter: {
    version: getCurrentVersion() // Returns "27" for current
  },

  // Allow "all versions" toggle
  allowCrossVersionSearch: true
};
```

**Search Result Display:**
```html
<div class="search-result">
  <span class="version-badge badge-current">v27</span>
  <h3>Installing TrueNAS</h3>
  <p>Step-by-step guide to installing TrueNAS...</p>
  <a href="/tn/27/getting-started/install/">Read more →</a>
</div>
```

---

## Maintenance Workflows

### Releasing New Versions (Make it "Current")

**Timeline: TrueNAS 27 Release**

**Week Before Release:**
1. **Create release branch** from `main`:
   ```bash
   git checkout main
   git pull origin main
   git checkout -b tn-27
   git push origin tn-27
   ```

2. **Set up Jenkins build** for `tn-27`:
   - Clone "TrueNAS-Docs-Build-Nightly" job
   - Rename to "TrueNAS-Docs-Build-TN27"
   - Update branch to `tn-27`
   - Update environment variables (VERSION=27, etc.)
   - Test build to `/var/www/docs.truenas.com/tn/27/`

3. **Update version config** in `tn-27` branch:
   ```toml
   # hugo.toml
   [params]
     version = "27"

     [[params.versions]]
       version = "Nightly (28-dev)"
       url = "/tn/nightly/"
       lifecycle = "next"

     [[params.versions]]
       version = "27 (Current)"
       url = "/tn/27/"
       lifecycle = "current"

     [[params.versions]]
       version = "26 (Previous)"
       url = "/tn/26/"
       lifecycle = "previous"
   ```

**Release Day:**
1. **Verify build** at `/tn/27/` is complete and correct
2. **Update symlink** on production server:
   ```bash
   ssh docs@docs.truenas.com
   cd /var/www/docs.truenas.com/tn/
   ln -sfn 27 current
   ```
3. **Update version configs** in other branches (tn-26, main) to reflect new lifecycle
4. **Test redirect**: Verify `docs.truenas.com/docs/` → `/docs/tn/current/` → `/docs/tn/27/`
5. **Announce** new docs version to team and users

**Post-Release:**
1. **Monitor** 404 errors and user feedback
2. **Update** search indexes if using separate indexes per version
3. **Review** old version (tn-26) for archival eligibility

### Archiving EOL Versions

**When to Archive:**
- Version reaches end-of-life (EOL)
- Typically 3 active versions maintained simultaneously
- Special consideration for LTS versions (may maintain longer)

**Example: Archiving TrueNAS 25 when TrueNAS 28 releases**

**Steps:**

1. **Announce archival** in release notes:
   ```markdown
   ## Documentation Updates

   - TrueNAS 28 documentation now available
   - TrueNAS 25 documentation archived (EOL)
   - Active versions: 26, 27, 28, nightly
   ```

2. **Update version configs** across all active branches:
   ```toml
   # Remove from main dropdown, add to "Archived Versions" section
   [[params.versions]]
     version = "25 (Archived)"
     url = "https://docs-archive.truenas.com/tn/25/"
     lifecycle = "archived"
   ```

3. **Disable Jenkins build** for `tn-25`:
   - Archive job configuration (don't delete)
   - Stop scheduled builds
   - Keep build artifacts for reference

4. **Move to archive server** (optional):
   ```bash
   # On production server
   cd /var/www/docs.truenas.com/tn/
   tar -czf tn-25-archive-$(date +%Y%m%d).tar.gz 25/

   # Copy to archive server
   scp tn-25-archive-*.tar.gz archive@docs-archive.truenas.com:/archives/

   # Extract on archive server
   ssh archive@docs-archive.truenas.com
   cd /var/www/docs-archive.truenas.com/tn/
   tar -xzf /archives/tn-25-archive-*.tar.gz
   ```

5. **Update nginx** to redirect archived version:
   ```nginx
   # Redirect archived versions to archive server
   location /docs/tn/25/ {
       return 301 https://docs-archive.truenas.com/tn/25$request_uri;
   }
   ```

6. **Update branch README** in `tn-25`:
   ```markdown
   # TrueNAS 25 Documentation (ARCHIVED)

   This version has reached end-of-life and is no longer maintained.

   Archived docs: https://docs-archive.truenas.com/tn/25/
   Current docs: https://docs.truenas.com/docs/
   ```

**Archive Server Benefits:**
- Reduces build time for active versions (only build 3-4 versions)
- Maintains historical record without main server overhead
- Can use cheaper storage for infrequently accessed content
- Clear separation between active and archived content

### Backporting (Preserved Process)

**Current backporting automation continues to work unchanged.**

**Workflow:**
1. Make change in `main` (nightly)
2. Backport script identifies eligible changes
3. Create backport PR to `tn-27`, `tn-26`, etc.
4. Review and merge backport PRs
5. Jenkins automatically rebuilds affected versions

**Example: Fixing a typo in installation docs**
```bash
# Fix made in main branch
git checkout main
# Edit content/en/getting-started/install/_index.md
git commit -m "Fix typo in installation instructions"
git push origin main

# Automated backport (or manual)
git checkout tn-27
git cherry-pick abc123d
git push origin tn-27

# Jenkins auto-builds /tn/27/ with fix
```

**No changes needed** to existing backport automation scripts - they continue to work with branch names.

### Version Dropdown Updates

**When to Update:**
- New version released
- Version reaches EOL/archived status
- Lifecycle status changes (e.g., previous → archived)

**Process:**

1. **Update `hugo.toml`** in ALL active branches:
   ```bash
   # Script: update-all-version-configs.sh
   #!/bin/bash

   BRANCHES=("main" "tn-26" "tn-27" "tn-28")

   for branch in "${BRANCHES[@]}"; do
       git checkout "$branch"
       git pull origin "$branch"

       # Edit hugo.toml to update [[params.versions]] section
       # (Manual edit or use sed/awk for automation)

       git add hugo.toml
       git commit -m "Update version dropdown for TrueNAS 28 release"
       git push origin "$branch"
   done
   ```

2. **Verify dropdown** appears correctly on all versions:
   - Visit `/tn/26/` - should show correct versions
   - Visit `/tn/27/` - should show correct versions
   - Visit `/tn/nightly/` - should show development status

3. **Test version switching**:
   - Click each version in dropdown
   - Verify URLs are correct
   - Check that lifecycle badges display properly

**Automation Opportunity:**
- Store version configuration in a central data file
- Script to update all branches automatically
- CI/CD pipeline to verify consistency across branches

---

## Archive Strategy

### Build Time Management

**Current Problem:**
- As versions accumulate, total build time increases linearly
- 10 versions × 5 min build = 50 minutes total build time
- Resource contention on Jenkins server

**Solution: Active + Archived Approach**

**Active Versions (always build):**
- Current stable (e.g., TrueNAS 27)
- Previous stable (e.g., TrueNAS 26)
- Next development (nightly)
- Optional: LTS version (if designated)

**Total: 3-4 builds** × 5 min = 15-20 minutes

**Archived Versions (no longer build):**
- Static HTML preserved on archive server
- No CI/CD overhead
- Accessible for historical reference

### When to Archive

**Criteria for Archiving:**

| Criterion | Description | Example |
|-----------|-------------|---------|
| **EOL Reached** | Version officially end-of-life | TrueNAS 25 EOL'd when 28 releases |
| **2+ Versions Old** | More than 2 major versions behind current | Archive 25 when 27 is current |
| **LTS Exception** | LTS versions maintained longer (e.g., 3-4 versions) | LTS version stays active beyond 2-version rule |
| **Low Traffic** | Analytics show <1% of doc views | Archive when users have moved on |

**Typical Timeline:**
```
2026: TrueNAS 26 releases → Active: [26, nightly]
2027: TrueNAS 27 releases → Active: [26, 27, nightly]
2028: TrueNAS 28 releases → Active: [26, 27, 28, nightly] → Archive 25
2029: TrueNAS 29 releases → Active: [27, 28, 29, nightly] → Archive 26
```

### Docs-Archive Repo Concept

**Approach: Separate Archive Infrastructure**

**Repository Structure:**
```
docs-archive/
├── tn/
│   ├── 20/         Static HTML for TrueNAS 20
│   ├── 21/         Static HTML for TrueNAS 21
│   ├── 22/         Static HTML for TrueNAS 22
│   ├── 23/         Static HTML for TrueNAS 23
│   ├── 24/         Static HTML for TrueNAS 24
│   └── 25/         Static HTML for TrueNAS 25
├── README.md       Archive documentation
└── .htaccess       Redirects and access rules
```

**Benefits:**
- Separate from active docs infrastructure
- Can use different hosting (cheaper, CDN, S3, etc.)
- No build overhead
- Clear separation of active vs. archived
- Can implement read-only policies

**Alternative: Git Branches**
- Keep archived versions in Git branches (no builds)
- Users can clone and build locally if needed
- Lower hosting cost (no static HTML storage)
- Requires users to have Hugo to view archived docs

### Hosting Options

**Option A: Same Server, Different Path**
- Host at `docs.truenas.com/archive/tn/25/`
- Simple nginx configuration
- Uses existing infrastructure
- **Cost:** Included in current hosting

**Option B: Subdomain**
- Host at `docs-archive.truenas.com/tn/25/`
- Clear separation from active docs
- Can implement different caching policies
- **Cost:** Minimal (DNS + nginx config)

**Option C: Static Storage (S3/CloudFlare)**
- Upload archived HTML to object storage
- Serve via CDN for performance
- Very low cost per GB
- Reduce load on main servers
- **Cost:** ~$0.023/GB/month (S3) or free (CloudFlare Pages)

**Option D: GitHub Pages**
- Publish archived docs to GitHub Pages
- Free hosting for public repos
- Good performance via CDN
- Easy to automate deployment
- **Cost:** Free

**Recommendation: Option B (Subdomain) → Option C (S3/CDN)**
- Start with subdomain (docs-archive.truenas.com) using existing server
- Migrate to S3/CDN as archive grows
- Best balance of cost, performance, and maintenance

---

## Migration Path

### Timeline and Phases

**Phase 1: Preparation (4-6 weeks before TN26 release)**

**Week 1-2: Planning & Setup**
- ✅ Document strategy (this document)
- ✅ Create branch naming conventions
- ✅ Design version selector UI/UX
- ✅ Update CLAUDE.md with versioning guidance

**Week 3-4: Development**
- Implement version selector shortcode
- Create version banner partial
- Update theme/layout templates
- Test locally with multiple version configs

**Week 5-6: Jenkins & Infrastructure**
- Create Jenkins job templates for versioned builds
- Configure nginx redirects (staging environment)
- Test symlink strategy
- Set up monitoring and alerting

**Phase 2: TrueNAS 26 Release (Release Week)**

**Pre-Release (3 days before):**
- Create `tn-26` branch from `main`
- Set up "TrueNAS-Docs-Build-TN26" Jenkins job
- Build and deploy to `/tn/26/` on staging
- Test all redirects and version switching on staging

**Release Day:**
- Deploy nginx configuration to production
- Update `/tn/current/` symlink to `26`
- Trigger production builds for all active versions
- Monitor logs for errors and 404s

**Post-Release (1 week after):**
- Gather user feedback
- Fix any issues with version navigation
- Update documentation and runbooks
- Train team on new workflows

**Phase 3: Iteration (TN26 → TN27)**

**During TN26 Lifecycle:**
- Refine version dropdown based on user feedback
- Optimize Jenkins builds and deployment
- Document lessons learned
- Prepare for TN27 release

**TN27 Release (same process):**
- Create `tn-27` branch
- Set up Jenkins job
- Update version configs across all branches
- Update `/tn/current/` symlink
- Update TN26 lifecycle to "previous"

**Phase 4: Archival (TN25 EOL)**

**When TN28 Releases:**
- Archive TN25 following documented process
- Test redirect to archive server
- Verify all active versions still work correctly
- Update team documentation

### IT Coordination Requirements

**Required IT/DevOps Tasks:**

**Infrastructure:**
- [ ] Nginx configuration changes (provide config blocks)
- [ ] Filesystem structure setup (`/var/www/docs.truenas.com/tn/`)
- [ ] Jenkins job creation and configuration
- [ ] Symlink management scripts and permissions
- [ ] Archive server setup (if using separate server)

**Deployment:**
- [ ] Staging environment testing
- [ ] Production deployment planning (change window)
- [ ] Rollback plan if issues occur
- [ ] Monitoring and alerting configuration

**Ongoing:**
- [ ] Jenkins job maintenance
- [ ] Symlink updates for new releases
- [ ] Archive server management
- [ ] SSL certificate renewal (if new subdomains)

**Communication:**
- Regular sync meetings during Phase 1
- Dedicated Slack channel for coordination
- Shared documentation and runbooks
- On-call contact during releases

### Testing Approach

**Test Scenarios:**

**1. Version Navigation**
- [ ] Visit root URL → Redirects to `/tn/current/`
- [ ] Click version dropdown → Shows all versions
- [ ] Select different version → Navigates correctly
- [ ] Verify version badge shows correct lifecycle

**2. Redirects**
- [ ] Old unversioned URLs redirect to current version
- [ ] Archived version URLs redirect to archive server
- [ ] Invalid version URLs return 404 (not 500)

**3. Search**
- [ ] Search within current version works
- [ ] Version filter functions correctly
- [ ] Cross-version search option available
- [ ] Results show version badges

**4. Build Process**
- [ ] Jenkins builds complete successfully
- [ ] Correct baseURL in generated HTML
- [ ] Assets (CSS, JS, images) load correctly
- [ ] Hugo modules resolve properly

**5. Symlink Management**
- [ ] Symlink update script works
- [ ] No downtime during symlink update
- [ ] Old symlink doesn't break existing links

**6. Mobile/Accessibility**
- [ ] Version dropdown works on mobile
- [ ] Banner displays correctly on small screens
- [ ] Keyboard navigation functional
- [ ] Screen reader compatibility

**Testing Tools:**
- **Local:** Hugo serve with different version configs
- **Staging:** Full deployment with production-like nginx
- **Monitoring:** Sentry for errors, Google Analytics for user behavior
- **Load Testing:** Verify performance with multiple versions

---

## Risks and Mitigations

### Build Time Concerns

**Risk:** Build times increase as versions accumulate
- **Severity:** Medium
- **Impact:** Slower deployments, delayed feedback
- **Mitigation:**
  - Limit active builds to 3-4 versions
  - Archive EOL versions promptly
  - Optimize Hugo build performance
  - Consider incremental builds for minor updates
  - Use parallel builds across Jenkins nodes

### User Confusion During Transition

**Risk:** Users accustomed to old URL structure get confused
- **Severity:** Medium
- **Impact:** Support burden, user frustration
- **Mitigation:**
  - Prominent banner explaining new versioning
  - Blog post announcing changes
  - Redirect old URLs to new structure
  - Update external links (forums, wikis, etc.)
  - FAQ section addressing common questions

### Symlink Management Errors

**Risk:** Incorrect symlink breaks `/current/` redirect
- **Severity:** High
- **Impact:** Users see wrong version or 404 errors
- **Mitigation:**
  - Automated script with validation checks
  - Test symlink before updating production
  - Monitor `/current/` URL in uptime checks
  - Documented rollback procedure
  - Backup of previous symlink target

### IT Coordination Dependencies

**Risk:** IT team has limited bandwidth for infrastructure changes
- **Severity:** Medium
- **Impact:** Delayed implementation, missed release dates
- **Mitigation:**
  - Early engagement with IT team
  - Phased implementation (can ship incrementally)
  - Provide complete nginx configs (copy-paste ready)
  - Offer to assist with setup and testing
  - Flexible timeline with buffer for delays

### Search Degradation

**Risk:** Version-aware search performs poorly or confuses users
- **Severity:** Low-Medium
- **Impact:** Users can't find content, search traffic drops
- **Mitigation:**
  - Default to current version (most common use case)
  - Make "all versions" search easily discoverable
  - Test search performance with multiple versions indexed
  - Monitor search analytics for drop in usage
  - Provide feedback mechanism for search issues

### Backport Automation Breaks

**Risk:** Existing backport scripts break with new branch structure
- **Severity:** Low
- **Impact:** Manual backporting required temporarily
- **Mitigation:**
  - Branch naming follows existing pattern (just adds `tn-` prefix)
  - Test backport scripts with new branches before release
  - Document manual backport process as backup
  - Gradual migration (can keep old branches temporarily)

---

## Open Questions

Items requiring team discussion and decision:

### 1. LTS Version Strategy

**Question:** Should TrueNAS designate certain versions as "Long Term Support" (LTS)?

**Options:**
- **A)** Every 2-3 major versions is LTS (e.g., 26, 29, 32)
- **B)** No formal LTS, just maintain 2 most recent versions
- **C)** Let product team decide LTS per release

**Considerations:**
- LTS versions would stay in "active builds" longer
- Impacts archive timeline (LTS versions archived later)
- Aligns with product support lifecycle

**Decision Needed By:** Before TN26 release
**Owner:** Product Management + Docs Team

### 2. Archive Server Timeline

**Question:** When should we set up the archive server?

**Options:**
- **A)** Immediately (before TN26 release) - be prepared
- **B)** When TN25 reaches EOL (1-2 years from now)
- **C)** When we hit 5+ archived versions (deferred)

**Considerations:**
- Option A requires IT bandwidth upfront
- Option B is "just in time" but may feel rushed
- Option C saves cost but builds up technical debt

**Decision Needed By:** Phase 1 planning
**Owner:** Docs Team Lead + IT Manager

### 3. Version Numbering Format

**Question:** How should we label pre-release and nightly versions?

**Current Proposal:** "Nightly (28-dev)"

**Alternatives:**
- "TrueNAS 28 (Development)"
- "TrueNAS Nightly"
- "Latest (Unreleased)"
- "28.0-beta" (when in beta phase)

**Considerations:**
- Should match product versioning terminology
- Need to differentiate nightly vs. beta vs. RC
- User clarity most important

**Decision Needed By:** Before implementing version dropdown UI
**Owner:** Docs Team + Product Team

### 4. Search Implementation

**Question:** Single search index or separate indexes per version?

**Options:**
- **A)** Single index with version filtering (recommended)
- **B)** Separate indexes, search current version by default

**Considerations:**
- Option A simpler to maintain, better cross-version search
- Option B better performance, cleaner separation
- Depends on search technology used

**Decision Needed By:** Phase 1 development
**Owner:** Docs Team + Web Developer

### 5. Root URL Behavior

**Question:** Should root URL (`/docs/`) redirect permanently (301) or temporarily (302)?

**Current Proposal:** 302 (temporary redirect)

**Rationale:**
- `/docs/` always redirects to current version
- Current version changes over time (26 → 27 → 28)
- 302 prevents search engines from indexing the redirect target
- Users landing on old bookmarks get latest version

**Alternatives:**
- 301 if we want search engines to index `/tn/current/` directly
- No redirect, show version picker landing page

**Decision Needed By:** Phase 1 nginx configuration
**Owner:** Docs Team + SEO consideration

### 6. Branch Protection Rules

**Question:** How should version branches be protected from accidental changes?

**Considerations:**
- `tn-26`, `tn-27` are stable release branches
- Should have stricter protection than `main`
- May need hotfix process for critical doc fixes
- Backport workflow must still function

**Proposal:**
- Require PR reviews for all version branches
- Limit direct push to release managers
- Allow automated backport commits from approved PRs

**Decision Needed By:** Before creating first version branch
**Owner:** Docs Team Lead + GitHub Admin

### 7. Version Lifecycle Labels

**Question:** What labels should we use for version states?

**Current Proposal:**
- **Current**: Latest stable release
- **Previous**: One version behind current
- **Next/Development**: Nightly/pre-release
- **Archived**: EOL versions

**Alternatives:**
- "Stable" instead of "Current"
- "Maintenance" instead of "Previous"
- "Preview" instead of "Next"
- "Legacy" instead of "Archived"

**Decision Needed By:** Before implementing version dropdown
**Owner:** Docs Team (consistency with other terminology)

---

## Success Metrics

### User Metrics

**Engagement:**
- [ ] **Time on page** increases (users reading correct version)
- [ ] **Bounce rate** decreases (users find relevant content)
- [ ] **Version dropdown clicks** tracked (usage of version navigation)
- [ ] **Search satisfaction** maintained or improved

**Awareness:**
- [ ] **Survey:** "Were you aware of which TrueNAS version these docs were for?"
  - Target: >90% answer "yes" (up from current unknown baseline)
- [ ] **404 errors** decrease after implementing redirects
  - Target: <0.5% of requests result in 404

**Behavior:**
- [ ] **Version-specific traffic** distributed appropriately
  - Current version: 70-80% of traffic
  - Previous version: 15-20% of traffic
  - Nightly: 3-5% of traffic
  - Archived: <2% of traffic

### Operational Metrics

**Build Performance:**
- [ ] **Total build time** for active versions: <20 minutes
  - Baseline: Current multi-branch builds
  - Target: 3-4 versions × 5 min each
- [ ] **Build success rate**: >95%
- [ ] **Time to deploy new version**: <30 minutes

**Maintenance:**
- [ ] **Symlink update time**: <5 minutes (includes testing)
- [ ] **Version dropdown update** across branches: <15 minutes
- [ ] **Time to archive EOL version**: <2 hours

**Reliability:**
- [ ] **Uptime** maintained at >99.9%
- [ ] **Zero downtime** during version releases
- [ ] **Redirect errors**: <0.1% of requests

### Team Metrics

**Efficiency:**
- [ ] **Time to backport fix** unchanged or improved
- [ ] **Support tickets** about version confusion decrease by >50%
- [ ] **Docs team** spends <5% of time on versioning maintenance

**Knowledge:**
- [ ] **Team members** trained on new workflows: 100%
- [ ] **Runbooks** created for all operational tasks
- [ ] **IT coordination** satisfaction: "easy" or "straightforward"

### Measurement Plan

**Tools:**
- **Google Analytics:** Traffic, engagement, version distribution
- **Sentry:** Error tracking for 404s, build failures
- **Jenkins:** Build time and success rate metrics
- **User Surveys:** Quarterly survey with versioning questions
- **Support Ticket Analysis:** Track version-related confusion

**Review Cadence:**
- **Weekly:** Build times and error rates
- **Monthly:** Traffic distribution and user behavior
- **Quarterly:** Comprehensive review with team survey
- **Yearly:** Full retrospective and strategy adjustment

---

## Appendix

### Technical Reference

**Key Technologies:**
- **Static Site Generator:** Hugo Extended (v0.146+)
- **Theme:** GeekDocs (custom-modified)
- **Build Tool:** Jenkins CI/CD
- **Web Server:** Nginx
- **Version Control:** Git (GitHub)
- **Search:** Custom search implementation (version-aware)

**Repository Locations:**
- **Docs Repo:** `github.com/truenas/docs`
- **Shared Module:** `github.com/truenas/docs-shared`
- **Archive Repo (future):** `github.com/truenas/docs-archive`

**Server Paths:**
- **Production:** `/var/www/docs.truenas.com/tn/`
- **Staging:** `/var/www/staging.docs.truenas.com/tn/`
- **Archive:** `/var/www/docs-archive.truenas.com/tn/`

### Nginx Configuration Blocks

**Complete Production Configuration:**
```nginx
# TrueNAS Documentation - Version Management
# /etc/nginx/sites-available/docs.truenas.com

server {
    listen 443 ssl http2;
    server_name docs.truenas.com;

    ssl_certificate /etc/ssl/certs/truenas.crt;
    ssl_certificate_key /etc/ssl/private/truenas.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    root /var/www/docs.truenas.com;
    index index.html;

    # Logging
    access_log /var/log/nginx/docs.truenas.com.access.log combined;
    error_log /var/log/nginx/docs.truenas.com.error.log warn;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Redirect root to current stable version
    location = /docs/ {
        return 302 /docs/tn/current/;
    }

    # Redirect common unversioned paths to current version
    location ~ ^/docs/(getting-started|installation|administration|troubleshooting|references|tutorials)(/.*)?$ {
        return 302 /docs/tn/current/$1$2;
    }

    # Serve versioned documentation (symlinks work transparently)
    location /docs/tn/ {
        try_files $uri $uri/ $uri/index.html =404;

        # Cache static assets
        location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
            expires 7d;
            add_header Cache-Control "public, immutable";
        }
    }

    # API documentation (separate from versioned docs)
    location /docs/api/ {
        try_files $uri $uri/ $uri/index.html =404;
    }

    # Apps documentation (separate from versioned docs)
    location /docs/apps/ {
        try_files $uri $uri/ $uri/index.html =404;
    }

    # Legacy fallback for any other /docs/ paths
    location /docs/ {
        try_files $uri $uri/ $uri/index.html =404;
    }

    # Archived versions redirect to archive server
    location /docs/tn/25/ {
        return 301 https://docs-archive.truenas.com/tn/25$request_uri;
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name docs.truenas.com;
    return 301 https://$server_name$request_uri;
}
```

**Archive Server Configuration:**
```nginx
# TrueNAS Documentation Archive
# /etc/nginx/sites-available/docs-archive.truenas.com

server {
    listen 443 ssl http2;
    server_name docs-archive.truenas.com;

    ssl_certificate /etc/ssl/certs/truenas.crt;
    ssl_certificate_key /etc/ssl/private/truenas.key;

    root /var/www/docs-archive.truenas.com;
    index index.html;

    # Aggressive caching for archived content (rarely changes)
    expires 30d;
    add_header Cache-Control "public, immutable";

    # Add archive notice header
    add_header X-Archive-Status "This is archived documentation" always;

    location / {
        try_files $uri $uri/ $uri/index.html =404;
    }
}
```

### Scripts and Automation

**update-current-version.sh:**
```bash
#!/bin/bash
# Updates /tn/current/ symlink to point to latest stable version
# Usage: ./update-current-version.sh 27

set -euo pipefail

VERSION="${1:-}"
DOCS_ROOT="/var/www/docs.truenas.com/tn"
BACKUP_DIR="/var/backups/docs-symlinks"

if [ -z "$VERSION" ]; then
    echo "❌ Error: Version number required"
    echo "Usage: $0 <version>"
    echo "Example: $0 27"
    exit 1
fi

if [ ! -d "${DOCS_ROOT}/${VERSION}" ]; then
    echo "❌ Error: Version directory ${DOCS_ROOT}/${VERSION} does not exist"
    echo "Available versions:"
    ls -1 "${DOCS_ROOT}" | grep -E '^[0-9]+$'
    exit 1
fi

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Backup current symlink target
if [ -L "${DOCS_ROOT}/current" ]; then
    CURRENT_TARGET=$(readlink "${DOCS_ROOT}/current")
    echo "📋 Current symlink points to: ${CURRENT_TARGET}"
    echo "${CURRENT_TARGET}" > "${BACKUP_DIR}/current-$(date +%Y%m%d-%H%M%S).txt"
fi

# Update symlink
cd "$DOCS_ROOT"
ln -sfn "$VERSION" current

# Verify symlink
NEW_TARGET=$(readlink current)
if [ "$NEW_TARGET" = "$VERSION" ]; then
    echo "✅ Successfully updated /tn/current/ → ${VERSION}"
    echo ""
    echo "Next steps:"
    echo "1. Test: curl -I https://docs.truenas.com/docs/tn/current/"
    echo "2. Verify: Check that content is correct"
    echo "3. Monitor: Watch for 404 errors in logs"
else
    echo "❌ Error: Symlink verification failed"
    exit 1
fi
```

**archive-version.sh:**
```bash
#!/bin/bash
# Archives an EOL TrueNAS documentation version
# Usage: ./archive-version.sh 25

set -euo pipefail

VERSION="${1:-}"
DOCS_ROOT="/var/www/docs.truenas.com/tn"
ARCHIVE_ROOT="/var/www/docs-archive.truenas.com/tn"
BACKUP_DIR="/var/backups/docs-archives"

if [ -z "$VERSION" ]; then
    echo "❌ Error: Version number required"
    echo "Usage: $0 <version>"
    echo "Example: $0 25"
    exit 1
fi

if [ ! -d "${DOCS_ROOT}/${VERSION}" ]; then
    echo "❌ Error: Version directory ${DOCS_ROOT}/${VERSION} does not exist"
    exit 1
fi

echo "📦 Archiving TrueNAS ${VERSION} documentation..."

# Create backup
mkdir -p "$BACKUP_DIR"
BACKUP_FILE="${BACKUP_DIR}/tn-${VERSION}-$(date +%Y%m%d-%H%M%S).tar.gz"
echo "Creating backup: ${BACKUP_FILE}"
tar -czf "$BACKUP_FILE" -C "$DOCS_ROOT" "$VERSION"
echo "✅ Backup created: ${BACKUP_FILE}"

# Move to archive server
echo "Moving to archive server..."
mkdir -p "$ARCHIVE_ROOT"
rsync -av --delete "${DOCS_ROOT}/${VERSION}/" "${ARCHIVE_ROOT}/${VERSION}/"

# Verify archive
if [ -d "${ARCHIVE_ROOT}/${VERSION}/index.html" ] || [ -f "${ARCHIVE_ROOT}/${VERSION}/index.html" ]; then
    echo "✅ Verified archive at ${ARCHIVE_ROOT}/${VERSION}"
else
    echo "⚠️  Warning: Could not verify archive"
fi

# Remove from active docs (optional - uncomment when ready)
# echo "Removing from active docs..."
# rm -rf "${DOCS_ROOT}/${VERSION}"

echo ""
echo "✅ Archive complete!"
echo ""
echo "Next steps:"
echo "1. Disable Jenkins build for tn-${VERSION}"
echo "2. Update nginx config to redirect /tn/${VERSION}/ to archive"
echo "3. Update version dropdowns in all active branches"
echo "4. Test archived URL: https://docs-archive.truenas.com/tn/${VERSION}/"
```

**update-all-version-configs.sh:**
```bash
#!/bin/bash
# Updates version dropdown configuration across all active branches
# Usage: ./update-all-version-configs.sh

set -euo pipefail

REPO_DIR="/path/to/docs"
BRANCHES=("main" "tn-26" "tn-27")

# Version configuration to apply
# Edit this section when releasing new versions
read -r -d '' VERSION_CONFIG <<'EOF' || true
  # Version dropdown configuration
  [[params.versions]]
    version = "Nightly (28-dev)"
    url = "/tn/nightly/"
    lifecycle = "next"

  [[params.versions]]
    version = "27 (Current)"
    url = "/tn/27/"
    lifecycle = "current"

  [[params.versions]]
    version = "26 (Previous)"
    url = "/tn/26/"
    lifecycle = "previous"
EOF

echo "🔄 Updating version configs across branches..."

cd "$REPO_DIR"

for branch in "${BRANCHES[@]}"; do
    echo ""
    echo "📝 Processing branch: ${branch}"

    git checkout "$branch"
    git pull origin "$branch"

    # Update hugo.toml
    # (This is a simplified example - actual implementation would use sed/awk
    #  or a more sophisticated config parser)

    echo "  → Updated hugo.toml"

    git add hugo.toml
    git commit -m "Update version dropdown configuration"
    git push origin "$branch"

    echo "  ✅ ${branch} updated"
done

git checkout main
echo ""
echo "✅ All branches updated!"
echo "Next: Verify version dropdowns on all docs sites"
```

---

## Document Metadata

**Author:** TrueNAS Documentation Team
**Created:** 2026-01-21
**Last Updated:** 2026-01-21
**Status:** Proposal (Pending Approval)
**Target Audience:** Docs Team, IT/DevOps, Product Management

**Related Documents:**
- `migration-tracker.md` - Documentation consolidation tracking
- `CLAUDE.md` - Development guidance for Claude Code
- Jenkins pipeline configurations in `/jenkins/` directory

**Feedback:**
Submit feedback, questions, or suggestions via:
- GitHub Issues in `truenas/docs` repository
- Docs team Slack channel
- Email to docs-team@truenas.com

---

**End of Proposal**
