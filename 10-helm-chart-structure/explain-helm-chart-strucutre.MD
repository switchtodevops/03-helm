# Helm Chart Structure

A Helm chart is a folder that contains Kubernetes YAML templates, default values, and chart metadata. Helm reads this folder, replaces template variables with values, and creates final Kubernetes manifests.

Example chart:

```text
basechart/
├── Chart.yaml
├── values.yaml
├── .helmignore
└── templates/
    ├── _helpers.tpl
    ├── deployment.yaml
    ├── service.yaml
    ├── serviceaccount.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    ├── httproute.yaml
    ├── NOTES.txt
    └── tests/
        └── test-connection.yaml
```

## 1. `Chart.yaml`

`Chart.yaml` is the identity card of the chart. It tells Helm the chart name, chart version, app version, and chart type.

```yaml
apiVersion: v2
name: basechart
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
```

Important fields:

- `apiVersion`: Helm chart API version. Helm 3 uses `v2`.
- `name`: Chart name.
- `description`: Short explanation of the chart.
- `type`: Usually `application`. Use `library` only for reusable helper charts.
- `version`: Chart package version. Change this when chart files change.
- `appVersion`: Application version, for example image/app version.

## 2. `values.yaml`

`values.yaml` contains default configuration values for the chart. Templates read these values using `.Values`.

Example:

```yaml
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: ""

service:
  type: ClusterIP
  port: 80
```

Template usage:

```yaml
replicas: {{ .Values.replicaCount }}
image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
containerPort: {{ .Values.service.port }}
```

You can override values during install or upgrade:

```bash
helm install my-app ./basechart --set replicaCount=3
helm upgrade my-app ./basechart --values prod-values.yaml
```

## 3. `.helmignore`

`.helmignore` tells Helm which files should not be included when packaging the chart.

Common examples:

```text
.git/
.vscode/
*.tmp
*.bak
*.swp
```

Use this for local files, editor files, backup files, and Git folders.

## 4. `templates/`

The `templates/` directory contains Kubernetes manifest templates. Helm renders these files into final YAML.

Common template files:

- `deployment.yaml`: Creates Pods through a Kubernetes Deployment.
- `service.yaml`: Exposes Pods inside the cluster.
- `serviceaccount.yaml`: Creates the service account used by Pods.
- `ingress.yaml`: Exposes the app through an Ingress controller.
- `hpa.yaml`: Creates autoscaling rules.
- `httproute.yaml`: Exposes the app using Gateway API.
- `NOTES.txt`: Prints helpful instructions after install.
- `_helpers.tpl`: Stores reusable template snippets.
- `tests/`: Stores Helm test manifests.

## 5. `templates/deployment.yaml`

`deployment.yaml` usually defines the application workload.

It commonly uses values like:

```yaml
replicas: {{ .Values.replicaCount }}
serviceAccountName: {{ include "basechart.serviceAccountName" . }}
image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
```

Simple meaning:

- `.Values.replicaCount` comes from `values.yaml`.
- `.Values.image.repository` comes from `values.yaml`.
- `.Chart.AppVersion` comes from `Chart.yaml`.
- `include "basechart.serviceAccountName" .` calls a helper from `_helpers.tpl`.

## 6. `templates/service.yaml`

`service.yaml` creates a Kubernetes Service. A Service gives stable networking for Pods.

Example idea:

```yaml
type: {{ .Values.service.type }}
port: {{ .Values.service.port }}
```

If `values.yaml` has `service.port: 80`, Helm renders the Service with port `80`.

## 7. `templates/ingress.yaml`

`ingress.yaml` is used when you want to access the application from outside the cluster using a domain name.

Typical scenario:

- Internal service: `my-app.default.svc.cluster.local`
- Public URL wanted: `https://app.example.com`
- Use Ingress to route external traffic to the Service.

Usually it is controlled by:

```yaml
ingress:
  enabled: false
```

Set it to `true` only when Ingress is needed.

## 8. `templates/hpa.yaml`

`hpa.yaml` creates a HorizontalPodAutoscaler. It increases or decreases Pod count based on CPU, memory, or custom metrics.

Simple scenario:

- Normal traffic: 1 Pod
- High traffic: scale to 3 or more Pods

Usually controlled by values like:

```yaml
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
```

## 9. `templates/serviceaccount.yaml`

`serviceaccount.yaml` creates a Kubernetes ServiceAccount. Pods use it when they need an identity inside Kubernetes.

Simple scenario:

- App only serves web traffic: default permissions may be enough.
- App reads Kubernetes API: it needs a ServiceAccount and RBAC permissions.

## 10. `templates/httproute.yaml`

`httproute.yaml` is used with Kubernetes Gateway API. It is similar in purpose to Ingress, but newer and more flexible.

Use it when your cluster has Gateway API installed and you want routing through a Gateway.

## 11. `templates/NOTES.txt`

`NOTES.txt` is not a Kubernetes resource. Helm prints it after install or upgrade.

Use it to show helpful next steps, for example:

```text
Run this command to get the application URL:
kubectl get svc --namespace {{ .Release.Namespace }}
```

## 12. `templates/_helpers.tpl`

`_helpers.tpl` stores reusable template code. Files starting with `_` are not rendered as Kubernetes manifests directly.

Why it is useful:

- Avoid repeating labels in every YAML file.
- Generate consistent resource names.
- Keep templates cleaner.

Example from this chart:

```yaml
{{- define "basechart.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
```

This helper creates a full resource name.

Scenario:

- Chart name is `basechart`
- Release name is `myapp`
- Helper output becomes `myapp-basechart`

Usage inside another template:

```yaml
metadata:
  name: {{ include "basechart.fullname" . }}
```

Rendered result:

```yaml
metadata:
  name: myapp-basechart
```

Another common helper is labels:

```yaml
{{- define "basechart.labels" -}}
helm.sh/chart: {{ include "basechart.chart" . }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}
```

Usage:

```yaml
labels:
  {{- include "basechart.labels" . | nindent 4 }}
```

Simple meaning: write labels once in `_helpers.tpl`, reuse them in Deployment, Service, and other templates.

## 13. `templates/tests/`

The `templates/tests/` folder contains Helm test resources. These are usually Pods that check if the release works.

Run tests with:

```bash
helm test <release-name>
helm test myapp --namespace default
```

Example scenario:

- Install app using Helm.
- Run `helm test`.
- Test Pod tries to connect to the Service.
- If connection works, test passes.

## 14. Optional Chart Files And Folders

Some charts may also include these:

```text
charts/          # downloaded chart dependencies
Chart.lock       # locked dependency versions
crds/            # CustomResourceDefinition files installed before templates
README.md        # chart documentation
LICENSE          # license information
values.schema.json # validates values.yaml input
```

Simple meaning:

- `charts/`: Stores dependency charts.
- `Chart.lock`: Records exact dependency versions.
- `crds/`: Stores CRDs that must exist before custom resources are installed.
- `README.md`: Explains how to use the chart.
- `values.schema.json`: Validates user-provided values.

## 15. How Helm Renders A Chart

Basic flow:

```text
Chart.yaml + values.yaml + templates/ = rendered Kubernetes YAML
```

Example commands:

```bash
helm template myapp ./basechart
helm install myapp ./basechart
helm upgrade myapp ./basechart --set replicaCount=3
```

In simple terms:

- `Chart.yaml` describes the chart.
- `values.yaml` provides default settings.
- `templates/` contains Kubernetes YAML with variables.
- `_helpers.tpl` contains reusable template functions.
- Helm combines everything and sends the final YAML to Kubernetes.
