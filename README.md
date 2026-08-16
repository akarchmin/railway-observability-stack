# Railway Observability Stack

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new?template=https%3A%2F%2Fgithub.com%2Fakarchmin%2Frailway-observability-stack)

A complete, production-ready observability stack for Railway. **Grafana + Loki + Prometheus + Tempo + OpenTelemetry Collector**, each as a separate Railway service with pre-configured integrations.

---

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Your Apps     │────▶│  OTel Collector  │────▶│     Tempo      │
│                 │     │  (Traces/Metrics)│     │   (Traces)     │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │   Prometheus    │
                         │   (Metrics)     │
                         └─────────────────┘

┌─────────────────┐     ┌─────────────────┐
│   Your Apps     │────▶│  OTel Collector  │────▶│     Loki       │
│                 │     │  (Logs)           │     │   (Logs)       │
└─────────────────┘     └─────────────────┘     └─────────────────┘

┌─────────────────┐
│   Your Browser  │────▶│     Grafana      │
│                 │     │  (Visualization) │
└─────────────────┘     └─────────────────┘
```

---

## Features

- **One-Click Deploy**: Deploy the entire stack with a single click
- **Persistent Storage**: All services use Railway volumes for data persistence
- **OpenTelemetry Native**: Unified ingestion via OTel Collector (traces, metrics, logs)
- **Pre-Configured**: Grafana datasources auto-connected to Loki, Prometheus, Tempo
- **Version Pinned**: Stable Docker image versions for all components
- **Production Ready**: Resource limits, healthchecks, and proper user permissions

---

## Quick Start

### Option 1: Deploy as Template (Recommended)

1. Click the **Deploy on Railway** button above
2. Select your Railway project
3. **Set required variables** on the Grafana service:
   - `GF_SECURITY_ADMIN_USER` = Your admin username
   - `GF_SECURITY_ADMIN_PASSWORD` = A **strong password** (not "admin")
4. Deploy all 5 services
5. Once deployed, open the Grafana URL and log in with your credentials

### Option 2: Deploy from Repository

1. In Railway, click **New Project** -> **Deploy from GitHub repo**
2. Select this repository
3. Railway will auto-create 5 services (one per `railway.json`)
4. Set Grafana variables as above
5. Deploy

---

## Services

| Service | Port | Purpose | Volume | Healthcheck |
|---------|------|---------|--------|-------------|
| **Grafana** | 3000 | Visualization & Dashboards | `/var/lib/grafana` | `/api/health` |
| **Loki** | 3100 | Log Aggregation | `/loki` | `/ready` |
| **Prometheus** | 9090 | Metrics Collection | `/prometheus` | `/-/healthy` |
| **Tempo** | 3200 | Distributed Tracing | `/var/tempo` | `/ready` |
| **OTel Collector** | 4317, 4318, 8888, 13133 | Telemetry Ingestion | None | `/` |

---

## Configuration

### Environment Variables

#### Grafana Service

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `GF_SECURITY_ADMIN_USER` | Yes | `admin` | Admin username |
| `GF_SECURITY_ADMIN_PASSWORD` | Yes | `admin` | **Set a secure value!** Admin password |
| `GF_DEFAULT_INSTANCE_NAME` | No | `Grafana on Railway` | Instance name |
| `GF_INSTALL_PLUGINS` | No | - | Comma-separated plugin list |

#### Internal URLs (Auto-Exposed by Railway)

After deployment, these variables are available to other Railway services:

| Variable | Example | Service |
|----------|---------|---------|
| `LOKI_INTERNAL_URL` | `http://loki-abc123.up.railway.app:3100` | Loki |
| `PROMETHEUS_INTERNAL_URL` | `http://prometheus-xyz456.up.railway.app:9090` | Prometheus |
| `TEMPO_INTERNAL_URL` | `http://tempo-def789.up.railway.app:3200` | Tempo |

### Version Pinning

Each service uses pinned Docker image versions. To update:

| Service | Version Variable | Current | Notes |
|---------|----------------|---------|-------|
| Grafana | `VERSION` | `13.0.2` | Set in Grafana service Variables |
| Loki | `VERSION` | `3.7.6` | Set in Loki service Variables |
| Prometheus | `VERSION` | `v3.13.2` | Set in Prometheus service Variables |
| Tempo | `VERSION` | `2.10.8` | Set in Tempo service Variables |
| OTel Collector | `VERSION` | `0.158.0` | Set in otel-collector service Variables |

---

## Connecting Your Applications

### Via OpenTelemetry Collector (Recommended)

Send all telemetry to the OTel Collector:

```bash
# Environment variables for your app
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector.up.railway.app:4317
OTEL_SERVICE_NAME=your-service-name
```

- **Traces**: gRPC port `4317` or HTTP port `4318`
- **Metrics**: OTLP endpoint (forwarded to Prometheus)
- **Logs**: OTLP endpoint (forwarded to Loki)

### Direct to Services

| Telemetry | Endpoint | Protocol |
|-----------|----------|-----------|
| **Traces** | `tempo:4317` | gRPC |
| **Traces** | `tempo:4318/v1/traces` | HTTP |
| **Metrics** | `prometheus:9090/api/v1/write` | HTTP |
| **Logs** | `loki:3100/loki/api/v1/push` | HTTP |

---

## Resource Limits

| Service | CPU | Memory | Notes |
|---------|-----|--------|-------|
| Grafana | 1 | 512 MB | Serverless mode |
| Loki | 1 | 1 GB | |
| Prometheus | 1 | 1 GB | |
| Tempo | 1 | 2 GB | |
| OTel Collector | 3 | 4 GB | Higher for processing |

---

## Region

No service pins a region. **Choose your region** during deployment from the Railway dropdown. The region can also be changed per service under **Settings -> Regions**.

---

## Local Development

Use the included `docker-compose.yml`:

```bash
# Start the stack
docker compose up -d

# View logs
docker compose logs -f

# Stop the stack
docker compose down
```

Services will be available at:
- Grafana: http://localhost:3000
- Prometheus: http://localhost:9090
- Loki: http://localhost:3100
- Tempo: http://localhost:3200
- OTel Collector: http://localhost:13133 (healthcheck)

---

## Troubleshooting

### "Permission denied" on startup
**Cause**: Railway volumes are root-owned. 
**Fix**: All Dockerfiles include `USER root` to ensure write access.

### Grafana datasources not connecting
**Check**: The internal URLs are correct. Use Railway's internal domain format:
```
http://${{ServiceName.RAILWAY_PRIVATE_DOMAIN}}:port
```

### Metrics not appearing in Grafana
**Check**: Prometheus is scraping the OTel Collector at `otel-collector:8888`.

---

## Project Structure

```
.
├── docker-compose.yml          # Local development
├── README.md
├── grafana/
│   ├── dockerfile
│   ├── railway.json
│   └── datasources/
│       └── datasources.yml     # Pre-configured datasources
├── loki/
│   ├── dockerfile
│   ├── railway.json
│   └── loki.yml               # Loki configuration
├── prometheus/
│   ├── dockerfile
│   ├── railway.json
│   └── prom.yml               # Prometheus configuration
├── tempo/
│   ├── dockerfile
│   ├── railway.json
│   └── tempo.yml              # Tempo configuration
└── otel-collector/
    ├── Dockerfile
    ├── railway.json
    └── otel-config.yaml        # OTel Collector configuration
```

---

## License

MIT
