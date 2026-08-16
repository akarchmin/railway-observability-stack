# Railway Observability Stack

Grafana + Loki + Prometheus + Tempo + OpenTelemetry Collector, each deployable as a Railway service.

## Region

No service pins a region in its `railway.json`. Config-as-code always overrides dashboard settings, so
a hardcoded `multiRegionConfig` would silently ignore the region picked in the Railway deploy dropdown.
Choose the region when deploying the template, or per service under **Settings -> Regions**.

## Grafana credentials

Set these variables on the Grafana service (Railway **Variables** tab, or as template variables):

| Variable | Description |
| --- | --- |
| `GF_SECURITY_ADMIN_USER` | Admin username. Defaults to `admin` if unset. |
| `GF_SECURITY_ADMIN_PASSWORD` | Admin password. Defaults to `admin` if unset — set a real value. |

The password is only applied to the existing admin account on first boot; change it in the Grafana UI afterwards.

Datasource URLs come from `LOKI_INTERNAL_URL`, `PROMETHEUS_INTERNAL_URL`, and `TEMPO_INTERNAL_URL`,
which should reference the internal domains of the other services, for example
`http://${{Loki.RAILWAY_PRIVATE_DOMAIN}}:3100`.

## Volumes

Prometheus (`/prometheus`), Loki (`/loki`), and Tempo (`/var/tempo`) each need a Railway volume mounted at
those paths. Railway mounts volumes root-owned, so the Prometheus and Tempo images run as `root`
(`USER root` in their Dockerfiles); otherwise they fail at startup with `permission denied`.

## Version pinning

Tempo is pinned to the 2.x line. Tempo 3.x restructured its config (the top-level `compactor` block was
replaced by `backend_scheduler`/`backend_worker`), so `tempo.yml` must be rewritten before moving to 3.x.
