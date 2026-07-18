# Helm Commands Reference

This file is organized by usage type. The command examples keep a consistent argument order:

```bash
helm <command> <release-name/chart-name> --namespace <namespace> [chart/version/value/status flags] [output/debug flags]
```

Use `helm <command> --help` for the full official help of any command.

## 1. Help, Version, And Environment

### Help

```bash
helm help
helm --help
helm <command> --help
helm repo --help
helm install --help
helm upgrade --help
```

### Version

```bash
helm version
helm version --short
helm version --template '{{.Version}}'
```

### Environment

```bash
helm env
helm env HELM_CACHE_HOME
helm env HELM_CONFIG_HOME
helm env HELM_DATA_HOME
helm env HELM_NAMESPACE
```

Important Helm environment variables:

```bash
HELM_CACHE_HOME
HELM_CONFIG_HOME
HELM_DATA_HOME
HELM_DEBUG
HELM_DRIVER
HELM_MAX_HISTORY
HELM_NAMESPACE
HELM_NO_PLUGINS
HELM_PLUGINS
HELM_REGISTRY_CONFIG
HELM_REPOSITORY_CACHE
HELM_REPOSITORY_CONFIG
KUBECONFIG
HELM_KUBECONTEXT
HELM_KUBEAPISERVER
HELM_KUBETOKEN
HELM_KUBEASUSER
HELM_KUBEASGROUPS
HELM_KUBECAFILE
HELM_KUBEINSECURE_SKIP_TLS_VERIFY
HELM_KUBETLS_SERVER_NAME
HELM_BURST_LIMIT
HELM_QPS
```

### Global Flags Used With Most Commands

```bash
helm <command> --namespace <namespace>
helm <command> -n <namespace>
helm <command> --kube-context <context-name>
helm <command> --kubeconfig <path-to-kubeconfig>
helm <command> --debug
helm <command> --registry-config <path>
helm <command> --repository-cache <path>
helm <command> --repository-config <path>
helm <command> --kube-apiserver <api-server-url>
helm <command> --kube-token <token>
helm <command> --kube-ca-file <ca-file>
helm <command> --kube-as-user <username>
helm <command> --kube-as-group <group>
helm <command> --kube-insecure-skip-tls-verify
helm <command> --kube-tls-server-name <server-name>
helm <command> --burst-limit <number>
helm <command> --qps <number>
```

## 2. Repository Commands

Use repository commands to add, update, list, remove, and index Helm chart repositories.

### Add Repository

```bash
helm repo add <repo-name> <repo-url>
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Important arguments:

```bash
helm repo add <repo-name> <repo-url> --username <username> --password <password>
helm repo add <repo-name> <repo-url> --username <username> --password-stdin
helm repo add <repo-name> <repo-url> --pass-credentials
helm repo add <repo-name> <repo-url> --force-update
helm repo add <repo-name> <repo-url> --ca-file <ca-file>
helm repo add <repo-name> <repo-url> --cert-file <cert-file> --key-file <key-file>
helm repo add <repo-name> <repo-url> --insecure-skip-tls-verify
```

### Update Repositories

```bash
helm repo update
helm repo update <repo-name>
helm repo update bitnami
```

### List Repositories

```bash
helm repo list
helm repo list --output table
helm repo list --output yaml
helm repo list --output json
```

### Remove Repository

```bash
helm repo remove <repo-name>
helm repo remove bitnami
helm repo remove <repo-name-1> <repo-name-2>
```

### Create Repository Index

```bash
helm repo index <directory>
helm repo index ./charts
helm repo index ./charts --url https://example.com/charts
helm repo index ./charts --merge ./index.yaml
helm repo index ./charts --json
```

## 3. Search Commands

Use search commands to find charts in local repositories or Artifact Hub.

### Search Local Repositories

```bash
helm search repo <keyword>
helm search repo nginx
helm search repo bitnami/nginx
helm search repo nginx --versions
helm search repo nginx --devel
helm search repo nginx --version <version-or-constraint>
helm search repo nginx --regexp
helm search repo nginx --max-col-width <number>
helm search repo nginx --output table
helm search repo nginx --output yaml
helm search repo nginx --output json
```

### Search Artifact Hub

```bash
helm search hub <keyword>
helm search hub nginx
helm search hub wordpress
helm search hub nginx --max-col-width <number>
helm search hub nginx --output table
helm search hub nginx --output yaml
helm search hub nginx --output json
```

## 4. Chart Information Commands

Use these commands before install or upgrade to inspect chart metadata, default values, templates, and packages.

### Show Chart Information

```bash
helm show chart <chart>
helm show chart bitnami/nginx
helm show chart bitnami/nginx --version <chart-version>
```

### Show Default Values

```bash
helm show values <chart>
helm show values bitnami/nginx
helm show values bitnami/nginx --version <chart-version>
```

### Show README

```bash
helm show readme <chart>
helm show readme bitnami/nginx
```

### Show CRDs

```bash
helm show crds <chart>
helm show crds bitnami/nginx
```

### Show All Chart Details

```bash
helm show all <chart>
helm show all bitnami/nginx
helm show all bitnami/nginx --version <chart-version>
```

Common chart source arguments:

```bash
helm show values <repo/chart> --version <chart-version>
helm show values <chart.tgz>
helm show values <local-chart-directory>
helm show values oci://<registry>/<namespace>/<chart> --version <chart-version>
helm show values <chart> --repo <repo-url>
helm show values <chart> --username <username> --password <password>
helm show values <chart> --ca-file <ca-file> --cert-file <cert-file> --key-file <key-file>
helm show values <chart> --insecure-skip-tls-verify
```

## 5. Pull And Verify Charts

### Pull Chart

```bash
helm pull <repo/chart>
helm pull bitnami/nginx
helm pull bitnami/nginx --version <chart-version>
helm pull bitnami/nginx --destination ./downloads
helm pull bitnami/nginx --untar
helm pull bitnami/nginx --untar --untardir ./charts
helm pull bitnami/nginx --prov
helm pull bitnami/nginx --verify --keyring <keyring-file>
helm pull oci://<registry>/<namespace>/<chart> --version <chart-version>
```

### Verify Chart Package

```bash
helm verify <chart-package.tgz>
helm verify nginx-1.0.0.tgz
helm verify nginx-1.0.0.tgz --keyring <keyring-file>
```

## 6. Install Releases

Use install commands to create a new release from a chart.

### Basic Install

```bash
helm install <release-name> <chart>
helm install my-nginx bitnami/nginx
helm install my-nginx bitnami/nginx --namespace web
helm install my-nginx bitnami/nginx --namespace web --create-namespace
```

### Install A Specific Chart Version

```bash
helm install <release-name> <repo/chart> --namespace <namespace> --version <chart-version>
helm install my-nginx bitnami/nginx --namespace web --version 15.14.0
```

### Install With Values

```bash
helm install <release-name> <chart> --namespace <namespace> --values values.yaml
helm install <release-name> <chart> --namespace <namespace> -f values.yaml
helm install <release-name> <chart> --namespace <namespace> -f base-values.yaml -f prod-values.yaml
helm install <release-name> <chart> --namespace <namespace> --set key=value
helm install <release-name> <chart> --namespace <namespace> --set image.tag=1.2.3
helm install <release-name> <chart> --namespace <namespace> --set-string app.version=1.2.3
helm install <release-name> <chart> --namespace <namespace> --set-json 'podAnnotations={"team":"platform"}'
helm install <release-name> <chart> --namespace <namespace> --set-file secret.data=./secret.txt
helm install <release-name> <chart> --namespace <namespace> --values values.yaml --set image.tag=1.2.3
```

### Install Safely

```bash
helm install <release-name> <chart> --namespace <namespace> --wait
helm install <release-name> <chart> --namespace <namespace> --wait --timeout 10m
helm install <release-name> <chart> --namespace <namespace> --atomic
helm install <release-name> <chart> --namespace <namespace> --atomic --timeout 10m
helm install <release-name> <chart> --namespace <namespace> --dependency-update
```

### Install Dry Run And Debug

```bash
helm install <release-name> <chart> --namespace <namespace> --dry-run
helm install <release-name> <chart> --namespace <namespace> --dry-run --debug
helm install <release-name> <chart> --namespace <namespace> --dry-run=client
helm install <release-name> <chart> --namespace <namespace> --dry-run=server
```

### Install From Different Sources

```bash
helm install <release-name> <repo/chart> --namespace <namespace>
helm install <release-name> ./local-chart --namespace <namespace>
helm install <release-name> ./chart-package.tgz --namespace <namespace>
helm install <release-name> https://example.com/charts/chart-1.0.0.tgz --namespace <namespace>
helm install <release-name> oci://<registry>/<namespace>/<chart> --namespace <namespace> --version <chart-version>
helm install <release-name> <chart-name> --repo <repo-url> --namespace <namespace>
```

### Install Name Options

```bash
helm install <chart> --generate-name
helm install <chart> --generate-name --name-template '<template>'
helm install <release-name> <chart> --replace
helm install <release-name> <chart> --description 'install description'
helm install <release-name> <chart> --labels team=platform,env=dev
```

### Other Important Install Flags

```bash
helm install <release-name> <chart> --skip-crds
helm install <release-name> <chart> --include-crds
helm install <release-name> <chart> --disable-openapi-validation
helm install <release-name> <chart> --take-ownership
helm install <release-name> <chart> --post-renderer <renderer-path>
helm install <release-name> <chart> --post-renderer-args <arg>
helm install <release-name> <chart> --render-subchart-notes
helm install <release-name> <chart> --hide-notes
helm install <release-name> <chart> --hide-secret
helm install <release-name> <chart> --insecure-skip-tls-verify
helm install <release-name> <chart> --plain-http
helm install <release-name> <chart> --username <username> --password <password>
helm install <release-name> <chart> --pass-credentials
```

## 7. Upgrade Releases

Use upgrade commands to change an existing release. `--install` creates it if it does not exist.

### Basic Upgrade

```bash
helm upgrade <release-name> <chart> --namespace <namespace>
helm upgrade my-nginx bitnami/nginx --namespace web
helm upgrade my-nginx bitnami/nginx --namespace web --version 15.14.0
```

### Upgrade Or Install

```bash
helm upgrade --install <release-name> <chart> --namespace <namespace>
helm upgrade --install my-nginx bitnami/nginx --namespace web --create-namespace
helm upgrade --install my-nginx bitnami/nginx --namespace web --version 15.14.0
```

### Upgrade With Values

```bash
helm upgrade <release-name> <chart> --namespace <namespace> --values values.yaml
helm upgrade <release-name> <chart> --namespace <namespace> -f values.yaml
helm upgrade <release-name> <chart> --namespace <namespace> -f base-values.yaml -f prod-values.yaml
helm upgrade <release-name> <chart> --namespace <namespace> --set key=value
helm upgrade <release-name> <chart> --namespace <namespace> --set image.tag=1.2.3
helm upgrade <release-name> <chart> --namespace <namespace> --set-string app.version=1.2.3
helm upgrade <release-name> <chart> --namespace <namespace> --set-json 'podAnnotations={"team":"platform"}'
helm upgrade <release-name> <chart> --namespace <namespace> --set-file secret.data=./secret.txt
```

### Upgrade Value Reuse Or Reset

```bash
helm upgrade <release-name> <chart> --namespace <namespace> --reuse-values
helm upgrade <release-name> <chart> --namespace <namespace> --reset-values
helm upgrade <release-name> <chart> --namespace <namespace> --reset-then-reuse-values
```

### Upgrade Safely

```bash
helm upgrade <release-name> <chart> --namespace <namespace> --wait
helm upgrade <release-name> <chart> --namespace <namespace> --wait --timeout 10m
helm upgrade <release-name> <chart> --namespace <namespace> --atomic
helm upgrade <release-name> <chart> --namespace <namespace> --atomic --cleanup-on-fail --timeout 10m
helm upgrade <release-name> <chart> --namespace <namespace> --dependency-update
```

### Upgrade Dry Run And Debug

```bash
helm upgrade <release-name> <chart> --namespace <namespace> --dry-run
helm upgrade <release-name> <chart> --namespace <namespace> --dry-run --debug
helm upgrade <release-name> <chart> --namespace <namespace> --dry-run=client
helm upgrade <release-name> <chart> --namespace <namespace> --dry-run=server
```

### Other Important Upgrade Flags

```bash
helm upgrade <release-name> <chart> --namespace <namespace> --force
helm upgrade <release-name> <chart> --namespace <namespace> --history-max <number>
helm upgrade <release-name> <chart> --namespace <namespace> --description 'upgrade description'
helm upgrade <release-name> <chart> --namespace <namespace> --labels team=platform,env=prod
helm upgrade <release-name> <chart> --namespace <namespace> --skip-crds
helm upgrade <release-name> <chart> --namespace <namespace> --disable-openapi-validation
helm upgrade <release-name> <chart> --namespace <namespace> --take-ownership
helm upgrade <release-name> <chart> --namespace <namespace> --post-renderer <renderer-path>
helm upgrade <release-name> <chart> --namespace <namespace> --post-renderer-args <arg>
helm upgrade <release-name> <chart> --namespace <namespace> --render-subchart-notes
helm upgrade <release-name> <chart> --namespace <namespace> --hide-notes
helm upgrade <release-name> <chart> --namespace <namespace> --hide-secret
```

## 8. List Releases

Use list commands to view releases by namespace, all namespaces, status, labels, or output format.

### Basic List

```bash
helm list
helm list --namespace <namespace>
helm list -n web
helm list --all-namespaces
helm ls
helm ls -A
```

### List By Release Status

```bash
helm list --deployed
helm list --failed
helm list --pending
helm list --uninstalled
helm list --uninstalling
helm list --superseded
helm list --all
helm list --namespace <namespace> --deployed
helm list --namespace <namespace> --failed
helm list --namespace <namespace> --uninstalled
```

### List With Filtering And Sorting

```bash
helm list --filter '<regular-expression>'
helm list --filter '^my-'
helm list --selector key=value
helm list --namespace <namespace> --selector app=nginx
helm list --date
helm list --reverse
helm list --max <number>
helm list --offset <number>
helm list --short
helm list --time-format '2006-01-02 15:04:05'
```

### List Output Formats

```bash
helm list --output table
helm list --output yaml
helm list --output json
```

## 9. Release Status, History, And Details

### Status

```bash
helm status <release-name> --namespace <namespace>
helm status my-nginx --namespace web
helm status my-nginx --namespace web --revision <revision-number>
helm status my-nginx --namespace web --show-resources
helm status my-nginx --namespace web --output table
helm status my-nginx --namespace web --output yaml
helm status my-nginx --namespace web --output json
```

### History

```bash
helm history <release-name> --namespace <namespace>
helm history my-nginx --namespace web
helm history my-nginx --namespace web --max <number>
helm history my-nginx --namespace web --output table
helm history my-nginx --namespace web --output yaml
helm history my-nginx --namespace web --output json
```

### Get All Release Details

```bash
helm get all <release-name> --namespace <namespace>
helm get all my-nginx --namespace web
helm get all my-nginx --namespace web --revision <revision-number>
```

### Get Manifest

```bash
helm get manifest <release-name> --namespace <namespace>
helm get manifest my-nginx --namespace web
helm get manifest my-nginx --namespace web --revision <revision-number>
```

### Get Values

```bash
helm get values <release-name> --namespace <namespace>
helm get values my-nginx --namespace web
helm get values my-nginx --namespace web --all
helm get values my-nginx --namespace web --revision <revision-number>
helm get values my-nginx --namespace web --output yaml
helm get values my-nginx --namespace web --output json
```

### Get Hooks, Notes, And Metadata

```bash
helm get hooks <release-name> --namespace <namespace>
helm get notes <release-name> --namespace <namespace>
helm get metadata <release-name> --namespace <namespace>
helm get hooks my-nginx --namespace web --revision <revision-number>
helm get notes my-nginx --namespace web --revision <revision-number>
helm get metadata my-nginx --namespace web --revision <revision-number>
```

## 10. Rollback Releases

Use rollback commands to move a release back to a previous revision.

```bash
helm rollback <release-name> <revision-number> --namespace <namespace>
helm rollback my-nginx 1 --namespace web
helm rollback my-nginx 1 --namespace web --wait
helm rollback my-nginx 1 --namespace web --wait --timeout 10m
helm rollback my-nginx 1 --namespace web --atomic
helm rollback my-nginx 1 --namespace web --cleanup-on-fail
helm rollback my-nginx 1 --namespace web --force
helm rollback my-nginx 1 --namespace web --no-hooks
helm rollback my-nginx 1 --namespace web --recreate-pods
helm rollback my-nginx 1 --namespace web --history-max <number>
```

## 11. Test Releases

Use test commands to run chart test hooks.

```bash
helm test <release-name> --namespace <namespace>
helm test my-nginx --namespace web
helm test my-nginx --namespace web --logs
helm test my-nginx --namespace web --filter name=<test-name>
helm test my-nginx --namespace web --timeout 5m
```

## 12. Uninstall Releases

Use uninstall commands to remove releases.

```bash
helm uninstall <release-name> --namespace <namespace>
helm uninstall my-nginx --namespace web
helm uninstall my-nginx --namespace web --wait
helm uninstall my-nginx --namespace web --wait --timeout 10m
helm uninstall my-nginx --namespace web --keep-history
helm uninstall my-nginx --namespace web --ignore-not-found
helm uninstall my-nginx --namespace web --no-hooks
helm uninstall my-nginx --namespace web --cascade background
helm uninstall my-nginx --namespace web --cascade foreground
helm uninstall my-nginx --namespace web --cascade orphan
helm delete my-nginx --namespace web
```

## 13. Render Templates Locally

Use template commands to render Kubernetes YAML without installing the release.

### Basic Template

```bash
helm template <release-name> <chart>
helm template my-nginx bitnami/nginx
helm template my-nginx bitnami/nginx --namespace web
helm template my-nginx bitnami/nginx --namespace web --version <chart-version>
```

### Template With Values

```bash
helm template <release-name> <chart> --namespace <namespace> --values values.yaml
helm template <release-name> <chart> --namespace <namespace> -f values.yaml
helm template <release-name> <chart> --namespace <namespace> --set key=value
helm template <release-name> <chart> --namespace <namespace> --set-string key=value
helm template <release-name> <chart> --namespace <namespace> --set-json 'key={"nested":"value"}'
helm template <release-name> <chart> --namespace <namespace> --set-file key=./file.txt
```

### Template Options

```bash
helm template <release-name> <chart> --namespace <namespace> --debug
helm template <release-name> <chart> --namespace <namespace> --dry-run=client
helm template <release-name> <chart> --namespace <namespace> --include-crds
helm template <release-name> <chart> --namespace <namespace> --skip-crds
helm template <release-name> <chart> --namespace <namespace> --show-only templates/deployment.yaml
helm template <release-name> <chart> --namespace <namespace> --output-dir ./rendered
helm template <release-name> <chart> --namespace <namespace> --validate
helm template <release-name> <chart> --namespace <namespace> --api-versions monitoring.coreos.com/v1
helm template <release-name> <chart> --namespace <namespace> --kube-version 1.30.0
helm template <release-name> <chart> --namespace <namespace> --is-upgrade
helm template <release-name> <chart> --namespace <namespace> --no-hooks
helm template <release-name> <chart> --namespace <namespace> --release-name
helm template <release-name> <chart> --namespace <namespace> --post-renderer <renderer-path>
```

## 14. Create, Lint, Dependency, And Package Charts

Use these commands when developing your own Helm charts.

### Create Chart

```bash
helm create <chart-name>
helm create my-chart
helm create my-chart --starter <starter-name>
```

### Lint Chart

```bash
helm lint <chart-directory>
helm lint ./my-chart
helm lint ./my-chart --values values.yaml
helm lint ./my-chart -f values.yaml
helm lint ./my-chart --set key=value
helm lint ./my-chart --strict
helm lint ./my-chart --with-subcharts
helm lint ./my-chart --quiet
```

### Dependency Commands

```bash
helm dependency list <chart-directory>
helm dependency list ./my-chart
helm dependency update <chart-directory>
helm dependency update ./my-chart
helm dependency build <chart-directory>
helm dependency build ./my-chart
helm dep list ./my-chart
helm dep update ./my-chart
helm dep build ./my-chart
```

Important dependency flags:

```bash
helm dependency update ./my-chart --skip-refresh
helm dependency update ./my-chart --verify
helm dependency update ./my-chart --keyring <keyring-file>
helm dependency build ./my-chart --skip-refresh
helm dependency build ./my-chart --verify
helm dependency build ./my-chart --keyring <keyring-file>
```

### Package Chart

```bash
helm package <chart-directory>
helm package ./my-chart
helm package ./my-chart --destination ./packages
helm package ./my-chart --version <chart-version>
helm package ./my-chart --app-version <app-version>
helm package ./my-chart --dependency-update
helm package ./my-chart --sign --key <key-name> --keyring <keyring-file>
helm package ./my-chart --passphrase-file <passphrase-file>
```

## 15. OCI Registry Commands

Use OCI commands for registries such as Docker Hub, GHCR, ACR, ECR, GCR, and Harbor.

### Registry Login And Logout

```bash
helm registry login <registry-host>
helm registry login ghcr.io
helm registry login <registry-host> --username <username> --password <password>
helm registry login <registry-host> --username <username> --password-stdin
helm registry login <registry-host> --ca-file <ca-file>
helm registry login <registry-host> --cert-file <cert-file> --key-file <key-file>
helm registry login <registry-host> --insecure
helm registry logout <registry-host>
helm registry logout ghcr.io
```

### Push Chart To OCI Registry

```bash
helm push <chart-package.tgz> oci://<registry>/<namespace>
helm push my-chart-1.0.0.tgz oci://ghcr.io/my-org/charts
helm push my-chart-1.0.0.tgz oci://<registry>/<namespace> --ca-file <ca-file>
helm push my-chart-1.0.0.tgz oci://<registry>/<namespace> --cert-file <cert-file> --key-file <key-file>
helm push my-chart-1.0.0.tgz oci://<registry>/<namespace> --insecure-skip-tls-verify
helm push my-chart-1.0.0.tgz oci://<registry>/<namespace> --plain-http
```

### Pull, Show, Install, Upgrade From OCI

```bash
helm pull oci://<registry>/<namespace>/<chart> --version <chart-version>
helm show values oci://<registry>/<namespace>/<chart> --version <chart-version>
helm install <release-name> oci://<registry>/<namespace>/<chart> --namespace <namespace> --version <chart-version>
helm upgrade <release-name> oci://<registry>/<namespace>/<chart> --namespace <namespace> --version <chart-version>
```

## 16. Plugin Commands

Use plugins to extend Helm.

```bash
helm plugin list
helm plugin install <plugin-url>
helm plugin install https://github.com/databus23/helm-diff
helm plugin install <plugin-url> --version <plugin-version>
helm plugin update <plugin-name>
helm plugin update <plugin-name-1> <plugin-name-2>
helm plugin uninstall <plugin-name>
helm plugin uninstall <plugin-name-1> <plugin-name-2>
```

Common plugin examples:

```bash
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade <release-name> <chart> --namespace <namespace> --values values.yaml
helm diff revision <release-name> <revision-number> --namespace <namespace>
helm diff rollback <release-name> <revision-number> --namespace <namespace>
```

## 17. Shell Completion

Use completion commands to enable shell autocomplete.

```bash
helm completion bash
helm completion zsh
helm completion fish
helm completion powershell
```

Linux bash examples:

```bash
helm completion bash > /etc/bash_completion.d/helm
source <(helm completion bash)
```

Zsh example:

```bash
source <(helm completion zsh)
```

Fish example:

```bash
helm completion fish | source
```

PowerShell example:

```powershell
helm completion powershell | Out-String | Invoke-Expression
```

## 18. Common Real-World Workflows

### Add Repo, Search, Install

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami/nginx --versions
helm install my-nginx bitnami/nginx --namespace web --create-namespace --version <chart-version> --values values.yaml --wait
```

### Install Or Upgrade Idempotently

```bash
helm upgrade --install my-nginx bitnami/nginx --namespace web --create-namespace --values values.yaml --atomic --timeout 10m
```

### Check What Is Running

```bash
helm list --all-namespaces
helm list --namespace web --deployed
helm status my-nginx --namespace web --show-resources
helm get values my-nginx --namespace web --all
helm get manifest my-nginx --namespace web
```

### Debug A Failed Release

```bash
helm list --namespace web --failed
helm status my-nginx --namespace web --show-resources
helm history my-nginx --namespace web
helm get all my-nginx --namespace web
helm get hooks my-nginx --namespace web
helm test my-nginx --namespace web --logs
```

### Roll Back A Bad Upgrade

```bash
helm history my-nginx --namespace web
helm rollback my-nginx <revision-number> --namespace web --wait --timeout 10m
helm status my-nginx --namespace web
```

### Render Before Applying

```bash
helm template my-nginx bitnami/nginx --namespace web --values values.yaml --debug > rendered.yaml
helm install my-nginx bitnami/nginx --namespace web --values values.yaml --dry-run --debug
helm upgrade my-nginx bitnami/nginx --namespace web --values values.yaml --dry-run --debug
```

### Clean Up A Release

```bash
helm uninstall my-nginx --namespace web --wait
helm list --namespace web --uninstalled
helm uninstall my-nginx --namespace web --keep-history
```

## 19. Output Format Flags

Commands that support output formatting commonly accept `table`, `json`, or `yaml`.

```bash
helm list --output table
helm list --output json
helm list --output yaml
helm status <release-name> --namespace <namespace> --output json
helm history <release-name> --namespace <namespace> --output yaml
helm get values <release-name> --namespace <namespace> --output json
helm repo list --output yaml
helm search repo <keyword> --output json
```

## 20. Value Flag Order And Examples

Recommended order when values are involved:

```bash
helm install <release-name> <chart> --namespace <namespace> --version <chart-version> --values values.yaml --set key=value --wait
helm upgrade <release-name> <chart> --namespace <namespace> --version <chart-version> --values values.yaml --set key=value --atomic
```

Important value flags:

```bash
--values <file>
-f <file>
--set key=value
--set-string key=value
--set-literal key=value
--set-json key=json
--set-file key=path
--reuse-values
--reset-values
--reset-then-reuse-values
```

Examples:

```bash
helm install app ./chart --namespace app --values values.yaml
helm install app ./chart --namespace app -f values.yaml -f values-prod.yaml
helm install app ./chart --namespace app --set replicaCount=3
helm install app ./chart --namespace app --set image.repository=nginx,image.tag=1.25.0
helm install app ./chart --namespace app --set-string service.port=8080
helm install app ./chart --namespace app --set-json 'extraEnv=[{"name":"ENV","value":"prod"}]'
helm install app ./chart --namespace app --set-file config=./config.yaml
```

## 21. Status Flag Examples

These are the most common release status filters.

```bash
helm list --deployed
helm list --failed
helm list --pending
helm list --uninstalled
helm list --uninstalling
helm list --superseded
helm list --all
helm list --all-namespaces --deployed
helm list --namespace web --failed
helm list --namespace web --uninstalled
```

## 22. All Helm Top-Level Commands

The Helm top-level commands available in Helm v3 include:

```bash
helm completion
helm create
helm dependency
helm env
helm get
helm help
helm history
helm install
helm lint
helm list
helm package
helm plugin
helm pull
helm push
helm registry
helm repo
helm rollback
helm search
helm show
helm status
helm template
helm test
helm uninstall
helm upgrade
helm verify
helm version
```

## 23. All Common Subcommands

```bash
helm completion bash
helm completion fish
helm completion powershell
helm completion zsh

helm dependency build
helm dependency list
helm dependency update

helm get all
helm get hooks
helm get manifest
helm get metadata
helm get notes
helm get values

helm plugin install
helm plugin list
helm plugin uninstall
helm plugin update

helm registry login
helm registry logout

helm repo add
helm repo index
helm repo list
helm repo remove
helm repo update

helm search hub
helm search repo

helm show all
helm show chart
helm show crds
helm show readme
helm show values
```
