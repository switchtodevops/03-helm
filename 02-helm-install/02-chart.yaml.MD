# Helm Chart.yaml — Complete Reference

> A single file that defines everything Helm needs to know about your chart. Read this when you need to understand, write, or debug a Chart.yaml.

---

## Table of Contents

1. [What is a Helm Chart?](#1-what-is-a-helm-chart)
2. [What is Chart.yaml?](#2-what-is-chartyaml)
3. [Anatomy of Chart.yaml — Fields Explained](#3-anatomy-of-chartyaml--fields-explained)
4. [Versioning Rules (SemVer 2)](#4-versioning-rules-semver-2)
5. [Dependencies Section](#5-dependencies-section)
6. [Annotating Your Chart](#6-annotating-your-chart)
7. [Complete Reference Example](#7-complete-reference-example)
8. [Common Mistakes & Gotchas](#8-common-mistakes--gotchas)
9. [Quick-Reference Cheat Sheet](#9-quick-reference-cheat-sheet)

---

## 1. What is a Helm Chart?

A **Helm Chart** is a packaged set of Kubernetes manifests. Think of it as a "recipe" that tells Helm:

- Which Kubernetes resources to create (Deployments, Services, ConfigMaps, etc.)
- What values (parameters) the user can change
- How the chart depends on other charts
- How to upgrade or roll back safely

### Directory layout of a chart

```
my-nginx-chart/          # <-- chart root (must contain Chart.yaml)
├── Chart.yaml            # <-- THE file this doc is about
├── values.yaml           # Default configuration values
├── values.schema.json    # JSON Schema for validating values.yaml
├── charts/               # Bundled chart dependencies (sub-charts)
│   └── postgresql/
├── crds/                 # Custom Resource Definitions (Kubernetes 1.14+)
│   └── my-crd.yaml
├── templates/            # Go template files that render into Kubernetes YAML
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl      # Named templates (shared snippets)
│   └── NOTES.txt         # Post-install instructions shown to the user
└── tests/                # Helm test manifests
    └── test-connection.yaml
```

**Only `Chart.yaml` is strictly required.** A chart with just `Chart.yaml` and one template file is a valid (though minimal) chart.

---

## 2. What is Chart.yaml?

`Chart.yaml` is a YAML metadata file that lives at the **root of every Helm chart**. It is the chart's identity card. Helm reads this file to:

- Identify the chart (name + version)
- Determine the target Kubernetes API version
- Parse dependencies on other charts
- Validate the chart (`helm lint` uses it)
- Package the chart (`helm package` uses the version field for the `.tgz` filename)

---

## 3. Anatomy of Chart.yaml — Fields Explained

Every field is shown with its type, whether it is required, and a plain-English explanation.

### 3.1 `apiVersion` (required)

```yaml
apiVersion: v2
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | String                                                                                    |
| **Required**| Yes                                                                                       |
| **Values**  | `v1` (legacy) or `v2` (current standard)                                                 |
| **What it means** | Which schema/format version the chart follows. Always use `v2` for new charts.     |

**`v2` vs `v1` differences:**

| Feature                   | `v1`          | `v2` (since Helm 3.0)         |
|---------------------------|---------------|-------------------------------|
| Dependencies keyword      | `dependencies` array inside `Chart.yaml` | Same, but also supports `requirements.yaml` for backward compat |
| Condition field on deps   | Not supported | Supported (you can enable/disable a dependency with a value) |
| Tags field on deps        | Not supported | Supported (activate multiple deps at once) |
| Alias field on deps       | Not supported | Supported (rename a sub-chart to avoid name collisions) |
| `appVersion`              | N/A           | Supported (tracks the app's version, separate from chart version) |
| `type` field              | N/A           | Supported (`application` or `library`) |
| Tiller reference          | Implicit      | Removed — Helm 3 has no server-side component (Tiller was removed) |

> **Rule of thumb:** If you are writing a new chart, always use `apiVersion: v2`. Helm 3 (released Nov 2019) made `v2` the standard and removed Tiller entirely.

---

### 3.2 `name` (required)

```yaml
name: my-nginx
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | String                                                                                    |
| **Required**| Yes                                                                                       |
| **Rules**   | Must start and end with an alphanumeric character (`[a-zA-Z0-9]`)<br>Can contain `-` (hyphen) in the middle<br>Maximum 253 characters<br>Lowercase preferred (Helm converts to lowercase for release names) |

This is the chart's identity. Combined with the version, it must be unique within a Helm repository. It is also the default sub-directory name when you run `helm create`.

---

### 3.3 `version` (required)

```yaml
version: 1.2.3
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | String                                                                                    |
| **Required**| Yes                                                                                       |
| **Format**  | SemVer 2 (see [Section 4](#4-versioning-rules-semver-2))                                 |
| **Used for**| The chart package version (`my-nginx-1.2.3.tgz`). Controls upgrade/rollback behavior.   |

> This is the **chart's version** (how the chart itself evolves), NOT the application's version. Use `appVersion` for that.

---

### 3.4 `description` (required but no format enforcement)

```yaml
description: A Helm chart for deploying Nginx on Kubernetes
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | String                                                                                    |
| **Required**| Yes (Helm `lint` will warn if missing)                                                   |
| **Length**  | One sentence is fine. Keep it under ~100 characters.                                      |
| **Purpose** | A human-readable summary shown in `helm search`, `helm show chart`, and the repo index.  |

---

### 3.5 `type` (optional)

```yaml
type: application
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | String                                                                                    |
| **Required**| No (defaults to `application`)                                                           |
| **Values**  | `application` or `library`                                                                |

| Value          | Meaning                                                                                                   |
|----------------|-----------------------------------------------------------------------------------------------------------|
| `application`  | A standard chart that installs a deployable application (most charts). Helm treats it as installable.    |
| `library`      | A helper chart that provides templates/utilities but **cannot be installed on its own**. It has no `templates/` directory with standard resources. Used as a dependency only. |

**Library chart example** — a chart that only defines reusable named templates:

```
my-shared-lib/
├── Chart.yaml          # type: library
└── _helpers.tpl        # shared template snippets
```

---

### 3.6 `keywords` (optional)

```yaml
keywords:
  - nginx
  - web
  - http
  - reverse-proxy
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | Array of strings                                                                          |
| **Required**| No                                                                                       |
| **Purpose** | Tags used by `helm search` and Helm repository index pages. Helps users discover your chart. |

---

### 3.7 `home` (optional)

```yaml
home: https://nginx.org
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | String (URL)                                                                              |
| **Required**| No                                                                                       |
| **Purpose** | A URL to the project's homepage. Shown in `helm search` results and the repo index.      |

---

### 3.8 `sources` (optional)

```yaml
sources:
  - https://github.com/nginx/nginx
  - https://github.com/my-org/nginx-helm-chart
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | Array of strings (URLs)                                                                   |
| **Required**| No                                                                                       |
| **Purpose** | Links to the upstream source code. Useful for users who want to inspect or contribute.    |

---

### 3.9 `maintainers` (optional)

```yaml
maintainers:
  - name: Jane Doe
    email: jane@example.com
    url: https://github.com/janedoe
```

| Field      | Type     | Required | Description                                      |
|------------|----------|----------|--------------------------------------------------|
| `name`     | String   | No       | Maintainer's name                                |
| `email`    | String   | No       | Contact email                                    |
| `url`      | String   | No       | Link to personal page / GitHub profile / org page|

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | Array of objects                                                                          |
| **Required**| No                                                                                       |
| **Purpose** | Attribution. Shown in `helm search` and repo index.                                      |

---

### 3.10 `icon` (optional)

```yaml
icon: https://nginx.org/nginx-logo.png
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | String (URL to an image, preferably PNG or SVG, 512x512 px)                               |
| **Required**| No                                                                                       |
| **Purpose** | An icon displayed in the Helm Hub / Artifact Hub alongside the chart listing.             |
| **Tip**     | Use a small file (< 1 MB) and a stable URL. SVG is best.                                 |

---

### 3.11 `kubeVersion` (optional)

```yaml
kubeVersion: ">=1.20.0-0"
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | SemVer constraint string                                                                  |
| **Required**| No                                                                                       |
| **Purpose** | Declares the minimum Kubernetes version required. Helm blocks installation on older clusters. |

**How constraints work (same syntax as `values.yaml` selectors):**

```yaml
kubeVersion: ">=1.25.0"      # Kubernetes 1.25 or later
kubeVersion: ">=1.25.0 <1.30.0"  # Kubernetes 1.25 to 1.29 (inclusive)
kubeVersion: ">=1.20.0-0"    # Common pattern — ignores patch/build metadata
```

> If the running cluster's Kubernetes version does not satisfy the constraint, Helm refuses to install the chart with an error.

---

### 3.12 `appVersion` (optional)

```yaml
appVersion: "1.25.3"
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | String                                                                                    |
| **Required**| No                                                                                       |
| **Purpose** | The version of the **application** being packaged (e.g. Nginx 1.25.3). This is informational only — Helm does NOT use it for upgrade logic. |

**`version` vs `appVersion` — the key distinction:**

| Field         | What it tracks                         | Used for upgrade/rollback? |
|---------------|----------------------------------------|----------------------------|
| `version`     | The chart package itself (your templates, values, docs) | **Yes** — bump this to trigger a Helm upgrade |
| `appVersion`  | The software the chart deploys (e.g. Nginx 1.25.3) | **No** — purely informational |

```yaml
# Example: packaging Nginx 1.25.3 in chart version 2.0.0
version: 2.0.0        # Changed this → helm upgrade deploys new templates
appVersion: "1.25.3"  # Changed this → only informational, no Helm action
```

---

### 3.13 `dependencies` (optional) — See full section below

```yaml
dependencies:
  - name: postgresql
    version: "12.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
    tags:
      - database
    import-values:
      - child: service
        parent: database
```

See [Section 5](#5-dependencies-section) for the complete guide.

---

### 3.14 `deprecated` (optional)

```yaml
deprecated: true
```

| Aspect      | Detail                                                                                     |
|-------------|-------------------------------------------------------------------------------------------|
| **Type**    | Boolean                                                                                   |
| **Required**| No                                                                                       |
| **Purpose** | Marks the chart as deprecated. Helm shows a warning when a user searches or installs it. |

---

## 4. Versioning Rules (SemVer 2)

Both `version` and `appVersion` must follow **Semantic Versioning 2.0.0**.

### The format

```
MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]
```

| Component      | Example      | Meaning                                                                 |
|----------------|--------------|-------------------------------------------------------------------------|
| **MAJOR**      | `2`          | Incompatible API changes (e.g., renamed values keys, removed templates) |
| **MINOR**      | `0`          | New functionality, backwards-compatible                                |
| **PATCH**      | `3`          | Bug fixes, backwards-compatible                                        |
| **Pre-release**| `-beta.1`    | Pre-release identifier (prerelease versions sort LOWER than release)   |
| **Build**      | `+build.123` | Build metadata (ignored in precedence comparisons)                     |

### How Helm uses version for upgrades

When you run `helm upgrade`, Helm compares the **installed chart version** with the **new chart version**:

| Scenario                           | Helm Behavior                      |
|------------------------------------|------------------------------------|
| New version > installed version    | Performs the upgrade               |
| New version <= installed version   | Blocks the upgrade (unless `--force` is used) |
| Pre-release is installed           | Upgrade to a non-prerelease IS allowed |
| Pre-release -> newer pre-release  | Allowed if the new one is greater   |

> **Rule:** `helm upgrade` refuses to downgrade. Bump `version` to a higher SemVer value for every chart change.

### Example version progression

```yaml
# Initial release
version: 1.0.0

# Fixed a bug in the template
version: 1.0.1

# Added a new configurable value
version: 1.1.0

# Breaking change — renamed .Values.service.port to .Values.service.targetPort
version: 2.0.0

# Pre-release for testing
version: 2.0.0-beta.1
```

---

## 5. Dependencies Section

The `dependencies` array declares that your chart needs other charts to function (e.g., your app chart depends on a PostgreSQL chart).

### Full field reference

```yaml
dependencies:
  - name: postgresql           # (required) Name of the dependency chart
    version: "12.x.x"          # (required) SemVer constraint
    repository: "https://charts.bitnami.com/bitnami"  # (required) URL of the Helm repo
    condition: postgresql.enabled   # (optional) Values path to enable/disable this dependency
    tags:                       # (optional) Group dependencies under a tag
      - database
    import-values:              # (optional) Import values from sub-chart into parent
      - child: service
        parent: database
    alias: pg                   # (optional) Rename the sub-chart to avoid conflicts
```

### Field-by-field

| Field            | Type              | Required | Description                                                                 |
|------------------|-------------------|----------|-----------------------------------------------------------------------------|
| `name`           | String            | Yes      | The chart name as declared in the dependency's own `Chart.yaml`             |
| `version`        | String (SemVer)   | Yes      | Version constraint. Supports ranges: `">=12.0.0 <13.0.0"`, `"~12.1.0"`     |
| `repository`     | String (URL)      | Yes      | URL of the Helm repository hosting the chart (must be added via `helm repo add`) |
| `condition`      | String            | No       | A values path. If the resolved value is `true`, the dependency is installed. E.g., `"postgresql.enabled"` checks `values.yaml` → `postgresql.enabled` |
| `tags`           | Array of strings  | No       | Categorize dependencies. You can activate ALL dependencies with a given tag using a single value |
| `import-values`  | Array of objects  | No       | Copy specific values from the sub-chart into the parent chart's values scope  |
| `alias`          | String            | No       | A new name for the sub-chart in the template context. Useful when two deps share a name |

### How `condition` works

In your `values.yaml`:

```yaml
postgresql:
  enabled: true       # <-- Chart.yaml condition: postgresql.enabled
  auth:
    username: myapp
```

If `postgresql.enabled` is `false`, Helm skips installing the PostgreSQL sub-chart entirely.

### How `tags` work

```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    tags: [database]
  - name: redis
    tags: [database, cache]

# values.yaml
database:
  enabled: true    # Activates BOTH postgresql AND redis because both have the "database" tag
cache:
  enabled: false   # redis is also disabled because it has the "cache" tag AND this is false
```

> A dependency is installed only when **all** its tags evaluate to `true`.

### How `alias` works

```yaml
dependencies:
  - name: postgresql
    alias: pg          # Templates in your chart reference it as {{ .Values.pg.* }} instead of {{ .Values.postgresql.* }}
    version: "12.x.x"
    repository: "https://charts.bitnami.com/bitnami"
```

In your templates you then use:

```yaml
{{ .Values.pg.auth.username }}
```

This avoids name collisions when two dependencies happen to share a chart name.

### Managing dependencies

```bash
# Declare deps in Chart.yaml, then resolve + download them:
helm dependency update

# This downloads .tgz files into the charts/ directory:
ls charts/
# postgresql-12.1.2.tgz

# List resolved dependencies:
helm dependency list

# Package the chart with deps bundled:
helm package .
```

> `helm dependency update` reads the `repository` URL from each dependency. Make sure the repo is already added (`helm repo add`) or provide a direct URL.

---

## 6. Annotating Your Chart

Annotations are key-value pairs attached to the chart metadata. They do not affect the chart's runtime behavior but carry machine-readable information.

```yaml
annotations:
  # --- Category / catalog ---
  "artifacthub.io/category": "database"
  "artifacthub.io/license": Apache-2.0
  "artifacthub.io/links": |
    [
      {"name": "Website", "url": "https://myapp.io"},
      {"name": "Source", "url": "https://github.com/my-org/myapp"}
    ]

  # --- OCI image tracking (shows which image version this chart deploys) ---
  "artifacthub.io/images": |
    [
      {"name": "myapp", "image": "myapp.io/web:1.5.0"},
      {"name": "nginx", "image": "nginx:1.25.3"}
    ]

  # --- Chart maturity ---
  "artifacthub.io/changes": |
    [
      {"kind": "added", "description": "Initial chart release"},
      {"kind": "changed", "description": "Bumped appVersion to 1.5.0"}
    ]

  # --- Keep annotations ---
  "helm.sh/chart": "myapp-1.0.0"       # Added automatically by helm package

  # --- Maintainer / support ---
  "artifacthub.io/support": "https://github.com/my-org/myapp/issues"
  "artifacthub.io/prerelease": "false"
```

### Common annotation keys (Artifact Hub)

| Annotation Key                          | Purpose                                              |
|-----------------------------------------|------------------------------------------------------|
| `artifacthub.io/category`               | Category for the hub (database, networking, storage) |
| `artifacthub.io/license`                | SPDX license identifier                              |
| `artifacthub.io/links`                  | JSON array of related links                          |
| `artifacthub.io/images`                 | JSON array of container images this chart uses       |
| `artifacthub.io/changes`                | JSON array of changelog entries                      |
| `artifacthub.io/prerelease`             | `"true"` if this is a pre-release version            |
| `artifacthub.io/containsSecurityUpdate` | `"true"` if this chart fixes a security issue        |
| `artifacthub.io/readme`                 | Override the README file (defaults to README.md)     |
| `helm.sh/resource-policy`               | Keep CRDs on uninstall (see below)                   |

### The `helm.sh/resource-policy` annotation

```yaml
# In Kubernetes manifests (NOT Chart.yaml) — shown here for reference
metadata:
  annotations:
    "helm.sh/resource-policy": keep
```

When applied to a resource in a template, this tells Helm: **do not delete this resource when the release is uninstalled**. This is most commonly used for:

- **PersistentVolumeClaims** — you don't want to lose data when tearing down a release
- **Custom Resource Definitions** — removing a CRD can break other releases that use it

```yaml
# In templates/pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "mychart.fullname" . }}
  annotations:
    "helm.sh/resource-policy": keep
spec:
  ...
```

---

## 7. Complete Reference Example

Here is a fully annotated `Chart.yaml` that demonstrates every field in context:

```yaml
# Chart.yaml — a production-ready Helm chart for a web application

# === IDENTITY ===
# Schema version — always v2 for Helm 3+
apiVersion: v2

# Chart name — lowercase, hyphens allowed
name: my-webapp

# Chart version — SemVer 2. Bump this to trigger helm upgrade.
# This is the PACKAGE version, not the app version.
version: 1.2.0

# Short human-readable description (required)
description: A Helm chart for deploying the MyWebApp web application on Kubernetes

# The APPLICATION version this chart currently packages (informational only)
# This tells users: "this chart deploys MyWebApp version 3.4.1"
appVersion: "3.4.1"

# Chart type — "application" for installable charts, "library" for template-only helper charts
type: application

# === DISCOVERABILITY ===
# Search keywords
keywords:
  - web
  - http
  - nginx
  - nodejs

# Project homepage
home: https://mywebapp.io

# Source code links
sources:
  - https://github.com/my-org/mywebapp
  - https://github.com/my-org/mywebapp-helm-chart

# === MAINTAINERS ===
maintainers:
  - name: Vinod M V
    email: vinod@example.com
    url: https://github.com/vinodmv

# === VISUAL ===
# Icon URL (displayed in Artifact Hub, Helm Hub)
icon: https://mywebapp.io/assets/logo.png

# === COMPATIBILITY ===
# Minimum Kubernetes version required
kubeVersion: ">=1.24.0-0"

# === DEPENDENCIES ===
# Sub-charts that must be installed alongside this chart
dependencies:
  # PostgreSQL database
  - name: postgresql
    version: "12.1.2"
    repository: "https://charts.bitnami.com/bitnami"
    # Only install PostgreSQL when the user sets postgresql.enabled=true in values.yaml
    condition: postgresql.enabled
    # Grouped under the "database" tag — activated by database.enabled=true
    tags:
      - database

  # Redis cache
  - name: redis
    version: "17.11.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
    tags:
      - cache

  # Certificate issuer (always installed, no condition)
  - name: cert-manager
    version: "1.13.0"
    repository: "https://charts.jetstack.io"
    tags:
      - infrastructure

# === ANNOTATIONS ===
# Machine-readable metadata for Helm repositories and Artifact Hub
annotations:
  # Artifact Hub categorization
  "artifacthub.io/category": "platform"
  "artifacthub.io/license": MIT

  # Related links (JSON array)
  "artifacthub.io/links": |
    [
      {"name": "Documentation", "url": "https://docs.mywebapp.io"},
      {"name": "Website", "url": "https://mywebapp.io"},
      {"name": "Source", "url": "https://github.com/my-org/mywebapp"}
    ]

  # Container images this chart deploys (for Artifact Hub display)
  "artifacthub.io/images": |
    [
      {"name": "mywebapp", "image": "mywebapp.io/web:3.4.1"},
      {"name": "nginx", "image": "nginx:1.25.3"}
    ]

  # Changelog
  "artifacthub.io/changes": |
    [
      {"kind": "added", "description": "Initial release of my-webapp chart", "links": [{"name": "PR", "url": "https://github.com/my-org/mywebapp-helm-chart/pull/1"}]},
      {"kind": "changed", "description": "Bump appVersion to 3.4.1"},
      {"kind": "fixed", "description": "Fixed service port template rendering"}
    ]

  # Mark as production-ready
  "artifacthub.io/prerelease": "false"

  # Added automatically by `helm package` — do not hand-write
  # "helm.sh/chart": "my-webapp-1.2.0"

  # Mark this chart as deprecated (informational — users get a warning)
  # deprecated: true
```

---

## 8. Common Mistakes & Gotchas

### Mistake 1: Confusing `version` with `appVersion`

```yaml
# WRONG — thinking bumping appVersion triggers an upgrade
appVersion: "3.5.0"   # Helm ignores this for upgrade checks!

# CORRECT — bump the chart version
version: 1.3.0        # This triggers the upgrade
appVersion: "3.5.0"   # This is informational only
```

### Mistake 2: Forgetting to run `helm dependency update`

```bash
# After editing the dependencies section:
helm dependency update   # MUST run this to download .tgz files into charts/
```

If you skip this step, `helm install` will fail with "found in Chart.yaml, but missing in charts/".

### Mistake 3: Using `apiVersion: v1`

```yaml
# WRONG for Helm 3
apiVersion: v1

# CORRECT
apiVersion: v2
```

### Mistake 4: Version does not increase after a change

```bash
# Helm refuses to upgrade if the chart version didn't increase:
# Error: UPGRADE FAILED: cannot re-use a name that is still in use
# Fix: bump version in Chart.yaml
```

### Mistake 5: `kubeVersion` too loose or too tight

```yaml
# Too loose — chart will install on Kubernetes that doesn't support required APIs
kubeVersion: ">=1.0.0"

# Too tight — blocks installation on perfectly capable clusters
kubeVersion: ">=1.28.0"

# Good practice — match the Kubernetes features your templates use
kubeVersion: ">=1.24.0"   # e.g., if you use ephemeral containers (K8s 1.25+) or structured auth (K8s 1.24+)
```

### Mistake 6: `type: library` chart with `templates/` that create resources

```yaml
# WRONG — library charts should not install resources
type: library

# A library chart should only have:
# - _helpers.tpl
# - (no standard templates that create Deployments, Services, etc.)
```

### Mistake 7: Hardcoding repository URLs without adding the repo first

```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    repository: "https://charts.bitnami.com/bitnami"  # Helm must know about this URL

# Then in the terminal, you MUST add the repo before `helm dependency update`:
helm repo add bitnami https://charts.bitnami.com/bitnami
helm dependency update
```

### Mistake 8: Using pre-release versions in production

```yaml
# Helm treats pre-release versions as lower precedence
version: 1.0.0-beta.1

# Helm will NOT upgrade from 1.0.0 to 1.0.0-beta.1 unless --force is used
# Pre-release charts are for testing, not for production releases
```

---

## 9. Quick-Reference Cheat Sheet

```
Chart.yaml — at a glance
========================

apiVersion: v2              # Always v2 for Helm 3
name: <string>              # Chart name, lowercase, hyphens OK
version: <SemVer>           # Chart package version — bump to upgrade
description: <string>       # One-line description
type: application           # "application" or "library"
appVersion: <string>        # Application version (informational)
kubeVersion: <SemVer>       # Min Kubernetes version constraint
home: <URL>                 # Project homepage
keywords: [<strings>]       # Search tags
sources: [<URLs>]           # Source code links
icon: <URL>                 # Chart icon URL
maintainers:                # Array of {name, email, url}
deprecated: <bool>          # Mark chart as deprecated

dependencies:               # Array of sub-chart requirements
  - name: <string>          #   Dependency chart name
    version: <SemVer>       #   Version constraint
    repository: <URL>        #   Helm repo URL
    condition: <path>       #   Values path to enable/disable
    tags: [<strings>]       #   Tag group
    alias: <string>         #   Rename sub-chart
    import-values:           #   Copy values from sub-chart
      - child: <path>
        parent: <path>

annotations:                # Key-value metadata
  <key>: <value>

Commands
========
helm create mychart         # Scaffold a new chart (includes Chart.yaml)
helm lint ./mychart         # Validate Chart.yaml and templates
helm show chart ./mychart   # Display Chart.yaml metadata
helm dependency update      # Download dependency charts
helm dependency list        # Show resolved dependencies
helm package .              # Package chart into .tgz (uses version from Chart.yaml)
helm lint . --strict        # Treat warnings as errors
```
