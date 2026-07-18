# Helm values.yaml — Complete Reference

> The single configuration file that controls everything your chart deploys. Read this to understand how to write, override, validate, and debug values.

---

## Table of Contents

1. [What is `values.yaml`?](#1-what-is-valuesyaml)
2. [The Values Hierarchy — Where Does a Value Come From?](#2-the-values-hierarchy--where-does-a-value-come-from)
3. [Data Types You Can Use](#3-data-types-you-can-use)
4. [How Templates Consume Values](#4-how-templates-consume-values)
5. [Override Methods (Ways to Change Values at Install Time)](#5-override-methods-ways-to-change-values-at-install-time)
6. [The `--set` Flag — Syntax Deep Dive](#6-the---set-flag--syntax-deep-dive)
7. [The `-f` / `--values` Flag — External Value Files](#7-the--f----values-flag--external-value-files)
8. [Validation — Making Sure Values Are Correct](#8-validation--making-sure-values-are-correct)
9. [Global Values Across Sub-Charts](#9-global-values-across-sub-charts)
10. [Built-in Values (Available Without Defining Them)](#10-built-in-values-available-without-defining-them)
11. [Practical Example — Before and After values.yaml](#11-practical-example--before-and-after-valuesyaml)
12. [Common Patterns](#12-common-patterns)
13. [Common Mistakes & Gotchas](#13-common-mistakes--gotchas)
14. [Quick-Reference Cheat Sheet](#14-quick-reference-cheat-sheet)

---

## 1. What is `values.yaml`?

`values.yaml` is a YAML configuration file at the **root of every Helm chart**. It is the chart's default settings file.

When a user runs `helm install`, Helm merges:
1. The chart's own `values.yaml` (defaults)
2. Any user-provided overrides (via `--set`, `-f`, etc.)
3. Sub-chart values (if the chart has dependencies)

The merged result is what your templates render against.

### Why does this matter?

Without `values.yaml`, every user would have to provide every single parameter on the command line. `values.yaml` ships sensible defaults so the chart works out-of-the-box, while still allowing customization.

### Where it lives in the chart

```
my-nginx-chart/
├── Chart.yaml
├── values.yaml          # <-- Default values (this file)
├── values.schema.json   # <-- JSON Schema for validation (optional)
├── charts/
├── templates/
└── ...
```

> `values.yaml` is the only required file besides `Chart.yaml`. A chart with just `Chart.yaml` and `values.yaml` (no templates yet) is structurally valid.

---

## 2. The Values Hierarchy — Where Does a Value Come From?

Helm merges values in a specific order. **Later sources override earlier ones.**

```
Priority (lowest to highest):
  1.  Chart.yaml's default values (if any — rarely used directly)
  2.  values.yaml in the chart                       ← your defaults
  3.  values.yaml from an imported parent chart       ← sub-chart inherits
  4.  values from a referenced values file (-f flag)  ← environment-specific
  5.  --set flags on the command line                 ← highest priority
  6.  --set-file flags                                 ← overrides --set for same key
```

### Example: how merging works

```yaml
# values.yaml (defaults)
replicaCount: 2
image:
  repository: nginx
  tag: "1.25"

# values-prod.yaml (provided via -f)
replicaCount: 5
image:
  tag: "1.25.3"

# helm install myapp . -f values-prod.yaml
# Result:
#   replicaCount: 5      (from values-prod.yaml)
#   image.repository: nginx  (from values.yaml — not overridden)
#   image.tag: "1.25.3"  (from values-prod.yaml)
```

> **Rule:** When two sources set the same key, the higher-priority source wins. For nested maps, the entire map at that level is replaced — NOT merged key-by-key.

```yaml
# values.yaml
image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent

# override via --set
--set image.tag=1.25.3

# Result:
#   image:
#     repository: nginx       ← preserved (not overridden)
#     tag: "1.25.3"           ← overridden
#     pullPolicy: IfNotPresent ← preserved (not overridden)
```

---

## 3. Data Types You Can Use

`values.yaml` is plain YAML. Any valid YAML type works.

### 3.1 Scalars

```yaml
# String
replicaCount: 3

# Integer
port: 80

# Boolean
enabled: true

# Null (explicitly unset)
imageTag: null
```

### 3.2 Strings that look like numbers — the quoting trap

```yaml
# WITHOUT quotes — YAML treats this as an integer
replicaCount: 3       # type: int

# WITH quotes — YAML treats this as a string
version: "1.25.3"     # type: string  (quotes needed because it contains dots)
```

> **Why this matters:** SemVer strings like `"1.25.3"` must be quoted. Without quotes, YAML interprets them as floating-point numbers, which loses precision and breaks version comparisons in templates.

### 3.3 Lists (arrays)

```yaml
# List of strings
tags:
  - web
  - production
  - frontend

# List of objects
ports:
  - name: http
    containerPort: 80
    protocol: TCP
  - name: metrics
    containerPort: 9090
    protocol: TCP
```

### 3.4 Dictionaries (maps)

```yaml
# Nested dictionary
image:
  repository: nginx
  tag: "1.25.3"
  pullPolicy: IfNotPresent

# Access in templates: {{ .Values.image.repository }}
```

### 3.5 Lists of dictionaries

```yaml
# Common pattern for Kubernetes resources
env:
  - name: DATABASE_HOST
    value: postgresql
  - name: DATABASE_PORT
    value: "5432"
  - name: LOG_LEVEL
    value: info
```

### 3.6 Indented block strings (multi-line)

```yaml
# A multi-line string using YAML literal block scalar
podAnnotations: |
  prometheus.io/scrape: "true"
  prometheus.io/port: "9090"
  prometheus.io/path: "/metrics"

# Or using folded style (newlines become spaces)
description: >
  This is a long description
  that spans multiple lines
  but renders as one paragraph.
```

---

## 4. How Templates Consume Values

Templates use Go templating syntax to read values. The root object is `.Values`.

### 4.1 The `.Values` object

```yaml
# values.yaml
replicaCount: 3
service:
  type: NodePort
  port: 80
image:
  repository: nginx
  tag: "1.25.3"
```

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-web
spec:
  replicas: {{ .Values.replicaCount }}        # → 3
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    spec:
      containers:
        - name: web
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          # → nginx:1.25.3
          ports:
            - containerPort: {{ .Values.service.port }}
            # → 80
```

### 4.2 Dot notation for nested values

```yaml
# These are equivalent:
{{ .Values.service.port }}
{{ index .Values "service" "port" }}
{{ index .Values.service "port" }}
```

### 4.3 The `index` function for keys with special characters

If a key contains characters that are not valid Go identifiers (like dots, dashes, or spaces), use `index`:

```yaml
# values.yaml
app:
  my-key: hello
  "127.0.0.1": localhost   # key is not a valid identifier
```

```yaml
# templates
{{ index .Values.app "my-key" }}       # → hello
{{ index .Values.app "127.0.0.1" }}    # → localhost
```

### 4.4 Safe navigation with `default`

```yaml
# If a value is not defined, the template fails with an error:
{{ .Values.optionalField }}    # Error if optionalField doesn't exist

# Use `default` to provide a fallback:
{{ .Values.optionalField | default "fallback-value" }}
{{ .Values.replicaCount | default 1 }}
{{ .Values.image.tag | default "latest" }}
```

```yaml
# values.yaml
replicaCount: 3

# templates
{{ .Values.replicaCount | default 1 }}   # → 3 (value exists, used as-is)
```

---

## 5. Override Methods (Ways to Change Values at Install Time)

Helm provides several ways to override `values.yaml` defaults. Pick the one that fits your use case.

### 5.1 `-f` / `--values` — override with a file (most common)

```bash
helm install myapp . -f values-prod.yaml
helm install myapp . -f values-dev.yaml -f values-prod.yaml
```

- Passes an **entire YAML file** of overrides.
- Multiple `-f` flags merge in order (last one wins for conflicts).
- Best for environment-specific configs (dev, staging, prod).

### 5.2 `--set` — override individual keys on the command line

```bash
helm install myapp . --set replicaCount=5
helm install myapp . --set image.tag=1.25.3
helm install myapp . --set service.type=ClusterIP
```

- Best for quick one-off overrides.
- See [Section 6](#6-the---set-flag--syntax-deep-dive) for the full syntax.

### 5.3 `--set-string` — force a value to be treated as a string

```bash
# Without --set-string, "3.14" might be parsed as a float
helm install myapp . --set-string image.tag=1.25.3
helm install myapp . --set-string replicaCount=3
```

- Forces the value to be a string, even if it looks like a number.
- Useful when a template expects a string (e.g., an image tag with dots).

### 5.4 `--set-file` — read value from a file on disk

```bash
# Read the entire file contents as a string value
helm install myapp . --set-file configMap.data=./my-config.conf

# The content of ./my-config.conf becomes the value of configMap.data
```

- Useful for injecting config files, certificates, or scripts without embedding them in YAML.

### 5.5 `--set-json` — set a value from a JSON string (Helm 3.9+)

```bash
helm install myapp . --set-json 'env=[{"name":"DB_HOST","value":"pg"},{"name":"DB_PORT","value":"5432"}]'
```

- Allows passing complex structures (lists, maps) via `--set`.

### Priority recap

```
values.yaml  <  -f file1.yaml  <  -f file2.yaml  <  --set  <  --set-file  <  --set-json
(lowest)                                                              (highest)
```

---

## 6. The `--set` Flag — Syntax Deep Dive

`--set` is powerful but has tricky syntax rules.

### 6.1 Basic key=value

```bash
--set replicaCount=5
--set service.type=ClusterIP
```

### 6.2 Nested keys with dots

```bash
# values.yaml
# image:
#   tag: "1.25"

--set image.tag=1.25.3          # Sets .Values.image.tag
--set image.pullPolicy=Never    # Sets .Values.image.pullPolicy
```

### 6.3 List elements with brackets

```bash
# values.yaml
# ports:
#   - name: http
#     containerPort: 80
#   - name: https
#     containerPort: 443

--set ports[0].containerPort=8080    # Override first element
--set ports[1].name=ssl              # Override second element
--set ports[2].name=metrics          # Add a third element
--set ports[2].containerPort=9090
```

### 6.4 Escaping special characters

| Character | Escape method | Example |
|-----------|---------------|---------|
| `.` in a key name | `\\.` | `--set "my\.key=value"` |
| `,` in a list | `\\,` | `--set "list=a\\,b\\,c"` |
| `[` or `]` | `\\[`, `\\]` | `--set "key=\\[0\\]"` |

```bash
# values.yaml has a key literally named "my.key":
# my:
#   my.key: default-value

--set "my\\.my\\.key=overridden"
```

### 6.5 Passing arrays

```bash
--set tags="{web,production,frontend}"
```

The `{...}` syntax creates a list.

### 6.6 Passing maps inline

```bash
--set "env={name:DB_HOST,value:postgres},{name:DB_PORT,value:5432}"
```

This creates:
```yaml
env:
  - name: DB_HOST
    value: postgres
  - name: DB_PORT
    value: 5432
```

### 6.7 `--set` gotchas

| Gotcha | Problem | Solution |
|--------|---------|----------|
| Boolean `true`/`false` | `--set enabled=true` may be interpreted as a string by the shell | Usually fine, but use `--set-string` if the template expects a string |
| Numeric values | `--set port=80` passes an integer | Use `--set-string port="80"` if the template does string comparison |
| Long keys | `--set some.very.deep.nested.key=value` is easy to mess up | Use a `-f` file instead for complex overrides |
| Shell interpretation | `--set "image.tag=v1.0"` needs quotes if the shell would otherwise interpret characters | Always quote `--set` values that contain special chars |

---

## 7. The `-f` / `--values` Flag — External Value Files

### 7.1 Single file override

```bash
helm install myapp . -f values-production.yaml
```

### 7.2 Multiple files — layered overrides

```bash
helm install myapp . -f values.yaml -f values-production.yaml
```

The files are applied in order. Later files override earlier ones. This enables a pattern like:

```bash
# values.yaml          — defaults (committed to Git)
# values-dev.yaml      — dev environment overrides (committed)
# values-prod.yaml     — prod overrides (committed)

# For dev:
helm install myapp . -f values.yaml -f values-dev.yaml

# For prod:
helm install myapp . -f values.yaml -f values-prod.yaml
```

### 7.3 Using environment variables with -f

```bash
# Generate a values file on the fly:
export DB_PASSWORD=$(openssl rand -base64 16)
envsubst < values-with-secrets.yaml.template | helm install myapp . -f values.yaml -f /dev/stdin
```

### 7.4 Recommended directory structure for environments

```
my-chart/
├── values.yaml            # Defaults (committed)
├── values-dev.yaml        # Dev overrides (committed)
├── values-staging.yaml    # Staging overrides (committed)
├── values-prod.yaml       # Production overrides (committed or kept secret)
├── values-secrets.yaml    # Secrets (NOT committed — in .gitignore)
└── templates/
```

```bash
# Install for each environment:
helm install myapp . -f values.yaml -f values-dev.yaml
helm install myapp . -f values.yaml -f values-staging.yaml
helm install myapp . -f values.yaml -f values-prod.yaml -f values-secrets.yaml
```

---

## 8. Validation — Making Sure Values Are Correct

### 8.1 JSON Schema validation (`values.schema.json`)

Place a `values.schema.json` file next to `values.yaml`. Helm uses it to validate user-provided values before rendering templates.

```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "replicaCount": {
      "type": "integer",
      "minimum": 1,
      "maximum": 100,
      "description": "Number of replicas to run"
    },
    "service": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "enum": ["ClusterIP", "NodePort", "LoadBalancer"],
          "description": "Kubernetes service type"
        },
        "port": {
          "type": "integer",
          "minimum": 1,
          "maximum": 65535
        }
      },
      "required": ["type", "port"]
    },
    "image": {
      "type": "object",
      "properties": {
        "repository": {
          "type": "string",
          "pattern": "^[a-zA-Z0-9./-]+$"
        },
        "tag": {
          "type": "string",
          "description": "Image tag, must not be empty"
        },
        "pullPolicy": {
          "type": "string",
          "enum": ["Always", "IfNotPresent", "Never"]
        }
      },
      "required": ["repository", "tag"]
    }
  },
  "required": ["replicaCount", "service", "image"]
}
```

```bash
# Validation runs automatically during:
helm install
helm upgrade
helm lint        # With --strict, schema errors are fatal

# Test a values file against the schema:
helm lint ./mychart -f values-prod.yaml --strict
```

**What JSON Schema can enforce:**
- Types (string, integer, boolean, array, object)
- Ranges (`minimum`, `maximum`)
- Enumeration (`enum`)
- Regex patterns (`pattern`)
- Required fields (`required`)
- Nested object structure

**What it CANNOT enforce:**
- Cross-field relationships (e.g., "if service.type is NodePort, nodePort must also be set")
- Conditional logic (use `required` in templates for that)

### 8.2 Template-level validation with `required`

```yaml
# templates/deployment.yaml
spec:
  containers:
    - name: web
      image: {{ required "image.repository is required" .Values.image.repository }}:{{ .Values.image.tag }}
      {{- if not .Values.image.tag }}
        {{- required "image.tag must not be empty" .Values.image.tag }}
      {{- end }}
```

The `required` function:
- Takes two arguments: a message and a value
- If the value is empty (`""`, `0`, `null`, `false`), it halts rendering and shows the message
- If the value is non-empty, it returns the value unchanged

### 8.3 Template-level validation with `fail`

```yaml
{{- if gt (.Values.replicaCount | int) 100 }}
  {{- fail "replicaCount cannot exceed 100" }}
{{- end }}
```

The `fail` function:
- Takes a message
- Immediately halts template rendering with that error
- Useful for custom validation logic that JSON Schema can't express

### 8.4 Linting

```bash
helm lint ./mychart                     # Check Chart.yaml and templates
helm lint ./mychart -f values.yaml      # Lint with specific values
helm lint ./mychart --strict            # Treat warnings as errors
```

`helm lint` catches:
- Missing required values (via `required` in templates)
- YAML syntax errors in values
- Template syntax errors
- Schema validation failures

---

## 9. Global Values Across Sub-Charts

When a chart has sub-chart dependencies, you often need to share values across all of them.

### 9.1 The `global` key

```yaml
# Parent chart values.yaml
global:
  domain: example.com
  environment: production
  imagePullSecrets:
    - name: regcred
```

In a sub-chart's template:

```yaml
# subchart/templates/service.yaml
spec:
  ports:
    - port: 80
      # Uses the parent's global.domain
      hostname: {{ .Values.global.domain }}
```

> `global` values propagate to ALL sub-charts. Every sub-chart can access `.Values.global.*` without explicit configuration.

### 9.2 Sub-chart isolation

By default, a sub-chart only sees its OWN values (under its chart name) plus `global`:

```yaml
# Parent values.yaml
postgresql:
  enabled: true
  auth:
    username: myapp

global:
  domain: example.com
```

```yaml
# In the postgresql sub-chart template:
# Accesses:
{{ .Values.postgresql.auth.username }}    # its own values
{{ .Values.global.domain }}               # global values

# Does NOT see:
{{ .Values.redis.enabled }}               # sibling sub-chart — not visible
```

### 9.3 Overriding sub-chart values from parent

```yaml
# Parent values.yaml — override sub-chart defaults
postgresql:
  enabled: true
  auth:
    username: myapp
    password: secretpass
    database: myappdb
  primary:
    persistence:
      enabled: true
      size: 10Gi
```

```bash
# Or override at install time:
helm install myapp . \
  --set postgresql.auth.password=mysecret \
  --set postgresql.primary.persistence.size=20Gi
```

---

## 10. Built-in Values (Available Without Defining Them)

These are special values Helm provides automatically. You do NOT define them in `values.yaml`.

| Object | Example Path | Description |
|--------|-------------|-------------|
| `.Release.Name` | `myapp` | The release name (given at `helm install <name>`) |
| `.Release.Namespace` | `default` | The target namespace |
| `.Release.IsUpgrade` | `true` / `false` | `true` if this is an upgrade |
| `.Release.IsInstall` | `true` / `false` | `true` if this is a fresh install |
| `.Release.Revision` | `1`, `2`, `3` | The revision number (increments on each upgrade) |
| `.Release.Service` | `Helm` | Always `"Helm"` (legacy from Tiller days) |
| `.Chart` | (nested object) | Information from the chart's `Chart.yaml` |
| `.Chart.Name` | `my-webapp` | The `name` field from Chart.yaml |
| `.Chart.Version` | `1.2.0` | The `version` field from Chart.yaml |
| `.Chart.AppVersion` | `3.4.1` | The `appVersion` field from Chart.yaml |
| `.Chart.APIVersion` | `v2` | The `apiVersion` field from Chart.yaml |
| `.Values` | (your values) | All values from the merged values files and --set flags |
| `.Files` | (file access) | Access to non-template files in the chart |
| `.Capabilities` | (Kubernetes info) | Kubernetes server version and API versions available |
| `.Template` | (template info) | The current template's name and base path |

### Using built-in values in templates

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-web              # "myapp-web"
  namespace: {{ .Release.Namespace }}        # "default"
  labels:
    app: {{ .Chart.Name }}                   # "my-webapp"
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"  # "my-webapp-1.2.0"
    version: "{{ .Chart.AppVersion }}"       # "3.4.1"
spec:
  replicas: {{ .Values.replicaCount }}
```

### Accessing `.Capabilities`

```yaml
# templates/_helpers.tpl
{{- define "mychart.apiVersion" -}}
{{- if .Capabilities.APIVersions.Has "networking.k8s.io/v1" }}
apiVersion: networking.k8s.io/v1
{{- else }}
apiVersion: networking.k8s.io/v1beta1
{{- end }}
{{- end -}}
```

This lets you conditionally use newer Kubernetes API versions when available, maintaining backward compatibility.

---

## 11. Practical Example — Before and After values.yaml

Let's use the Kubernetes manifests from your study repo to show how `values.yaml` parameterizes them.

### The raw Kubernetes manifests (hardcoded)

```yaml
# deployment.yaml (hardcoded — no flexibility)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp1-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp1
  template:
    metadata:
      name: myapp1-pod
      labels:
        app: myapp1
    spec:
      containers:
        - name: myapp1-container
          image: ghcr.io/stacksimplify/kubenginxhelm:0.1.0
          ports:
            - containerPort: 80
```

```yaml
# service.yaml (hardcoded — no flexibility)
apiVersion: v1
kind: Service
metadata:
  name: myapp1-nodeport-service
spec:
  type: NodePort
  selector:
    app: myapp1
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 31300
```

### The parameterized version with values.yaml

```yaml
# values.yaml
# ============================================================
# Default values for my-nginx chart
# Every value the chart needs has a sensible default here.
# ============================================================

# --- Replica configuration ---
replicaCount: 2

# --- Application configuration ---
image:
  repository: ghcr.io/stacksimplify/kubenginxhelm
  tag: "0.1.0"
  pullPolicy: IfNotPresent

# --- Service configuration ---
service:
  type: NodePort
  port: 80           # Service port
  targetPort: 80     # Container port
  nodePort: 31300    # Node port (30000-32767 range)

# --- Pod labels ---
podLabels:
  app: myapp1

# --- Pod annotations ---
podAnnotations: {}

# --- Resource limits (optional) ---
resources: {}
  # limits:
  #   cpu: 200m
  #   memory: 256Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi
```

```yaml
# templates/deployment.yaml (parameterized)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.podLabels.app }}
  template:
    metadata:
      name: {{ .Release.Name }}-pod
      labels:
        app: {{ .Values.podLabels.app }}
    spec:
      containers:
        - name: {{ .Release.Name }}-container
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.targetPort }}
          {{- with .Values.resources }}
          resources:
            {{- toYaml . | nindent 12 }}
          {{- end }}
```

```yaml
# templates/service.yaml (parameterized)
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Values.podLabels.app }}
  ports:
    - name: http
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      nodePort: {{ .Values.service.nodePort }}
```

### Deploying with different configurations

```bash
# Install with defaults (replicaCount=2, NodePort, etc.)
helm install myapp .

# Install with 5 replicas
helm install myapp . --set replicaCount=5

# Install as a ClusterIP service with a different image tag
helm install myapp . \
  --set service.type=ClusterIP \
  --set image.tag=0.2.0 \
  --set image.pullPolicy=Always

# Install with a full environment file
helm install myapp . -f values-production.yaml
# (where values-production.yaml overrides replicaCount, service.type, etc.)
```

---

## 12. Common Patterns

### 12.1 Boolean flags with conditionals

```yaml
# values.yaml
ingress:
  enabled: false
  hosts:
    - host: myapp.example.com
      path: /
```

```yaml
# templates/ingress.yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
spec:
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host }}
      http:
        paths:
          - path: {{ .path }}
            pathType: Prefix
            backend:
              service:
                name: {{ $.Release.Name }}-service
                port:
                  number: {{ $.Values.service.port }}
    {{- end }}
{{- end }}
```

The Ingress resource only renders when `ingress.enabled` is `true`.

### 12.2 Conditional container ports

```yaml
# values.yaml
containerPorts:
  http: 80
  metrics: 9090
```

```yaml
# templates/deployment.yaml
          ports:
            {{- range $name, $port := .Values.containerPorts }}
            - name: {{ $name }}
              containerPort: {{ $port }}
              protocol: TCP
            {{- end }}
```

This renders a port block for every entry in the `containerPorts` map.

### 12.3 Image pull secrets

```yaml
# values.yaml
imagePullSecrets: []
  # - name: regcred
```

```yaml
# templates/deployment.yaml
spec:
  {{- if .Values.imagePullSecrets }}
  imagePullSecrets:
    {{- range .Values.imagePullSecrets }}
    - name: {{ .name }}
    {{- end }}
  {{- end }}
```

### 12.4 Node selector and tolerations

```yaml
# values.yaml
nodeSelector: {}
  # disktype: ssd

tolerations: []
  # - key: "node-type"
  #   operator: "Equal"
  #   value: "compute"
  #   effect: "NoSchedule"
```

```yaml
# templates/deployment.yaml
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### 12.5 Using `toYaml` and `nindent` for clean output

```yaml
# values.yaml
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "9090"

# templates/deployment.yaml
      annotations:
        {{- toYaml .Values.podAnnotations | nindent 8 }}
```

Renders as:

```yaml
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
```

`toYaml` converts a map/list to YAML. `nindent N` adds a newline then N spaces of indentation. This combination is the idiomatic way to dump complex values into a manifest.

### 12.6 Using `b64enc` for secrets

```yaml
# values.yaml
auth:
  username: myapp
  password: mysecretpassword

# templates/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-db-credentials
type: Opaque
data:
  username: {{ .Values.auth.username | b64enc }}
  password: {{ .Values.auth.password | b64enc }}
```

`b64enc` base64-encodes the value. Kubernetes Secrets require base64-encoded values in the `data` section.

### 12.7 Conditional service type

```yaml
# values.yaml
service:
  type: NodePort
  port: 80
  targetPort: 80
  nodePort: 31300  # Only used when type is NodePort
  loadBalancerIP: ""  # Only used when type is LoadBalancer
```

```yaml
# templates/service.yaml
      type: {{ .Values.service.type }}
      {{- if eq .Values.service.type "NodePort" }}
      nodePort: {{ .Values.service.nodePort }}
      {{- end }}
      {{- if eq .Values.service.type "LoadBalancer" }}
      loadBalancerIP: {{ .Values.service.loadBalancerIP }}
      {{- end }}
```

---

## 13. Common Mistakes & Gotchas

### Mistake 1: Quoting SemVer strings

```yaml
# WRONG — YAML interprets 1.25.3 as a float
tag: 1.25.3

# CORRECT — quoted so it stays a string
tag: "1.25.3"

# CORRECT — single quotes also work
tag: '1.25.3'
```

### Mistake 2: Expecting deep merge with `--set` and `-f`

```yaml
# values.yaml
image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent

# values-override.yaml
image:
  tag: "1.25.3"
```

```bash
# Result — pullPolicy is NOT lost:
# image:
#   repository: nginx      ← preserved
#   tag: "1.25.3"          ← overridden
#   pullPolicy: IfNotPresent ← preserved
```

```yaml
# But with --set:
--set image.tag=1.25.3

# Same result — only tag changes, other keys in image are preserved
```

> Helm does a **deep merge** for map values. Only the leaf key you override is replaced. Sibling keys at the same level are kept.

### Mistake 3: Using `--set` for complex structures

```bash
# Hard to read and error-prone:
--set "env={name:DB_HOST,value:postgres},{name:DB_PORT,value:5432}"

# Better: use a values file:
# env-values.yaml
# env:
#   - name: DB_HOST
#     value: postgres
#   - name: DB_PORT
#     value: "5432"

helm install myapp . -f env-values.yaml
```

### Mistake 4: Forgetting that list indices are 0-based

```bash
# values.yaml
# ports:
#   - name: http
#     containerPort: 80
#   - name: https
#     containerPort: 443

# ports[0] is "http", ports[1] is "https"
--set ports[0].containerPort=8080     # Modifies the FIRST element
--set ports[1].containerPort=8443     # Modifies the SECOND element
```

### Mistake 5: Not quoting values that the shell would interpret

```bash
# WRONG — shell interprets the asterisk:
helm install myapp . --set args="--log-level=*"

# CORRECT — quote it:
helm install myapp . --set "args=--log-level=*"
```

### Mistake 6: Confusing `tpl` with regular values

```yaml
# values.yaml
message: "Hello {{ .Release.Name }}"

# template.yaml
message: {{ .Values.message | quote }}
# Renders: message: "Hello {{ .Release.Name }}"  (literal string, not evaluated)

# CORRECT — use tpl to evaluate Go templates inside values:
message: {{ tpl .Values.message . }}
# Renders: message: "Hello myapp"  (template evaluated)
```

### Mistake 7: Not providing defaults with `default`

```yaml
# If a user doesn't set image.tag, this breaks:
image: {{ .Values.image.repository }}:{{ .Values.image.tag }}

# CORRECT — provide a fallback:
image: {{ .Values.image.repository }}:{{ .Values.image.tag | default "latest" }}
```

### Mistake 8: Overriding an entire map unintentionally

```bash
# values.yaml has:
# image:
#   repository: nginx
#   tag: "1.25"
#   pullPolicy: IfNotPresent

# This --set REPLACES the entire "image" map:
--set image={repository:nginx,tag:1.26}

# Result:
# image:
#   repository: nginx
#   tag: 1.26
#   pullPolicy: MISSING!  ← gone because the whole map was replaced
```

### Mistake 9: Forgetting that `true`/`false` in YAML are booleans

```yaml
# values.yaml
debug: false     # This is a boolean

# If your template checks it with eq, this is fine:
{{ if eq .Values.debug true }}

# But if you override with --set-string, it becomes a string:
--set-string debug=false
# Now .Values.debug is the STRING "false", not boolean false
# eq .Values.debug true → false (string "false" != boolean true)
```

### Mistake 10: `null` values are skipped in deep merge

```yaml
# values.yaml
image:
  tag: "1.25"
  pullPolicy: IfNotPresent

# override file
image:
  tag: null    # This REMOVES the tag key entirely!

# Result:
# image:
#   pullPolicy: IfNotPresent
#   (tag is gone!)
```

---

## 14. Quick-Reference Cheat Sheet

```
values.yaml — at a glance
=========================

# Scalars
replicaCount: 3
imageTag: "1.25.3"       # Quote strings that look like numbers
enabled: true
debug: false

# Dictionaries
image:
  repository: nginx
  tag: "1.25.3"
  pullPolicy: IfNotPresent

# Lists
ports:
  - name: http
    containerPort: 80

# Multi-line strings
podAnnotations: |
  key1: value1
  key2: value2

Access in templates
===================
{{ .Values.replicaCount }}                    # scalar
{{ .Values.image.repository }}                # nested key
{{ .Values.image.tag | default "latest" }}    # with fallback
{{ index .Values "weird.key.name" }}          # keys with dots/dashes
{{ required "replicaCount is required" .Values.replicaCount }}  # enforce required
{{ fail "replicaCount exceeds max" }}         # custom validation
{{ .Values.image.tag | b64enc }}              # base64 encode
{{ .Values.podAnnotations | toYaml | nindent 8 }}  # dump map as YAML

Override methods
================
helm install myapp . -f values-prod.yaml                    # file override
helm install myapp . -f a.yaml -f b.yaml                    # layered files
helm install myapp . --set key=value                        # single key
helm install myapp . --set image.tag=1.25                   # nested key
helm install myapp . --set ports[0].containerPort=8080      # list element
helm install myapp . --set-string tag=1.25.3                # force string
helm install myapp . --set-file config.data=./file.conf     # file content
helm install myapp . --set-json 'env=[{...}]'               # JSON (3.9+)

Priority: values.yaml < -f files < --set < --set-file < --set-json

Validation
==========
helm lint .                                    # Check for errors
helm lint . --strict                           # Warnings as errors
helm lint . -f values-prod.yaml                # Lint with values
# values.schema.json in chart root → auto-validated on install/upgrade

Built-in objects (no need to define in values.yaml)
====================================================
.Release.Name, .Release.Namespace, .Release.Revision
.Release.IsInstall, .Release.IsUpgrade
.Chart.Name, .Chart.Version, .Chart.AppVersion, .Chart.APIVersion
.Files.Get, .Files.AsConfig, .Files.AsSecrets
.Capabilities.APIVersions.Has
```
