# admin-api-chart

Helm chart for **admin-api-server** — the bouc.io platform admin API (LLM
providers & assignments, organizations, global/org instructions, system config).

Part of the [bouc.io AI platform](../../../documentation/getting-started/README.md#ai-assistant-platform).

## What it deploys

A stateless Deployment plus a ClusterIP Service, backed by a bundled **PostgreSQL** subchart.

| Object | Name |
|---|---|
| Deployment | `<release>-admin-api-chart` |
| Service | `<release>-admin-api-chart` |
| ServiceAccount | `<release>-admin-api-chart` (when `serviceAccount.create`) |
| HorizontalPodAutoscaler | `<release>-admin-api-chart` (when `autoscaling.enabled`) |
| Ingress | `<release>-admin-api-chart` (when `ingress.enabled`) |

The container name inside the pod is `{{ .Chart.Name }}`, i.e. `admin-api-chart`, so use
`-c admin-api-chart` with `kubectl exec`.

## Values

Values live in three files. There is no plain `values.yaml`.

| File | Purpose |
|---|---|
| `base.values.yaml` | Common defaults shared across environments |
| `lcl.values.yaml` | Local (docker-desktop / Pi) overlay |
| `snbx.values.yaml` | Sandbox cluster overlay |

> In the cluster, FluxCD supplies values from generated ConfigMaps via `valuesFrom:`, not from these
> files directly. They are the source the ConfigMaps are generated from.

### Key values

| Key | Description |
|---|---|
| `replicaCount` | Number of pod replicas |
| `image.registry` / `image.repository` / `image.tag` | Container image, joined by the `admin-api-chart.image` helper (tag bumped by FluxCD image automation) |
| `service` | Service type and port |
| `serviceAccount` | ServiceAccount name/annotations |
| `resources` | CPU/memory requests and limits |
| `autoscaling` | HPA settings (min/max replicas, target utilization) |
| `environment` | App env vars (see `admin-api-server/.env.example` for the full reference) |
| `postgresql` | Bundled PostgreSQL subchart toggle + config |

## Probes

`livenessProbe` and `readinessProbe` are fixed in the Deployment template, not values-driven:
`/health/live` and `/health/ready` on the HTTP port. Splitting live from ready lets a pod whose
dependencies are briefly unavailable drop out of rotation instead of restarting.

## Local usage

```bash
helm dependency update                       # fetch subchart deps (postgresql)
helm lint . -f base.values.yaml -f lcl.values.yaml
helm template test . -f base.values.yaml -f lcl.values.yaml
helm install admin-api . -f base.values.yaml -f lcl.values.yaml
```

The values files layer: `base` first, then exactly one environment file.

> The chart must be published to the chart registry by CI before FluxCD can reconcile it. Pushing
> chart source to git is not enough.

## License

[Elastic License 2.0](./LICENSE) — source-available; not OSI open source.
