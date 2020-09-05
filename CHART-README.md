# {{.Chart.Name}}

<!-- Simple description of application the chart is installing, with a link to the upstream app.-->
{{.App.Description}}

{{if .Prerequisites}}
## Prerequisites

{{range .Prerequisites}}{{printf "- %s\n" .}}{{end}}
{{end}}

## Get Repo Info

```console
helm repo add {{Repo.Name}} {{Repo.URL}}
{{range $d := .Chart.Dependencies}}
<-- To-do: get preferred repo shortname from Artifact Hub API (example: bitnami).-->
helm repo add {{$d.ShortName}} {{$d.Repository}}
{{end}}
helm repo update
```

_See [helm repo](https://helm.sh/docs/helm/helm_repo/) for command documentation._

## Install Chart

```console
# Helm 3
$ helm install [RELEASE_NAME] {{Repo.Name}}/{{.Chart.Name}} [flags]

# Helm 2
$ helm install --name [RELEASE_NAME] {{Repo.Name}}/{{.Chart.Name}} [flags]
```

_See [configuration](#configuration) below._

_See [helm install](https://helm.sh/docs/helm/helm_install/) for command documentation._

{{if .Chart.Dependencies}}
## Dependencies

By default this chart installs additional, dependent charts:

{{range $d := .Chart.Dependencies}}
<!-- Example:
- [bitnami/kube-state-metrics](https://artifacthub.io/packages/helm/bitnami/kube-state-metrics)
-->
<!-- To-do: get link to dependency via Artifact Hub API -->
- [{{$d.ShortName}} {{$d.Repository}}]({{$d.HubURL}})
{{end}}

<!-- Example:
To disable the dependency during installation, set `kubeStateMetrics.enabled` to `false`.
-->
<!-- To-do: fix this pseudocode lol -->
<!-- Requires `Join()` func
tmpl.Funcs(template.FuncMap{"StringsJoin": strings.Join}) -->
{{range $d := .Chart.Dependencies}}
{{define "depEnabledKeys"}}
{{default $d.Name $d.alias}}.enabled
{{end}}
{{end}}
To disable the dependency during installation, set {{StringsJoin .depEnabledKeys ", "}} to `false`.

_See [helm dependency](https://helm.sh/docs/helm/helm_dependency/) for command documentation._
{{end}}

## Uninstall Chart

```console
# Helm 3
$ helm uninstall [RELEASE_NAME]

# Helm 2
# helm delete --purge [RELEASE_NAME]
```

This removes all the Kubernetes components associated with the chart and deletes the release.

_See [helm uninstall](https://helm.sh/docs/helm/helm_uninstall/) for command documentation._

## Upgrading Chart

```console
# Helm 3 or 2
$ helm upgrade [RELEASE_NAME] {{Repo.Name}}/{{.Chart.Name}} [flags]
```

_See [helm upgrade](https://helm.sh/docs/helm/helm_upgrade/) for command documentation._

<!-- If major version bumps, under H3 explain breaking changes justifying bump and manual steps required. Examples below: -->
### Major Version Upgrades

Chart release versions follow [semver](../../CONTRIBUTING.md#versioning), where a MAJOR version change (example `1.0.0` -> `2.0.0`) indicates an incompatible breaking change needing manual actions.

### To v2.0.0
Example manual steps.

### To v1.0.0
Example manual steps.

## Configuration

See [Customizing the Chart Before Installing](https://helm.sh/docs/intro/using_helm/#customizing-the-chart-before-installing). To see all configurable options with detailed comments, visit the chart's [values.yaml](./values.yaml), or run these configuration commands:

```console
# Helm 2
$ helm inspect values prometheus-community/prometheus

# Helm 3
$ helm show values prometheus-community/prometheus
```

You may similarly use the above configuration commands on each chart [dependency](#dependencies) to see it's configurations.

<!-- If Specific configuration categories need explaining, explain under H3. Examples below: -->
### Configuration Type A

Example config steps:
1. Create secret with `shell command`
1. [Install](#install-chart) chart with setting `foo.bar` to `[SECRET_NAME]`, and `baz.qux.quux` to `true`

### Configuration Type B

In order to X, you must set `quuz.corge` to `false` and `quuz.grault` to `garply`.
