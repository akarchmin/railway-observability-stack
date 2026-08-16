# OpenTelemetry Collector

This repository contains the OpenTelemetry Collector configuration for routing telemetry data to separate observability services.

## Configuration

The collector receives OTLP data and routes it to:
- **Traces** → Tempo
- **Metrics** → Prometheus 
- **Logs** → Loki

## Environment Variables

Set these environment variables to configure the endpoints:

```bash
TEMPO_ENDPOINT=http://your-tempo-host:4317
PROMETHEUS_ENDPOINT=http://your-prometheus-host:9090/api/v1/write
LOKI_ENDPOINT=http://your-loki-host:3100/loki/api/v1/push
```

## Deployment

### Railway
Deploy this as a separate service on Railway with the environment variables configured.

### Docker
```bash
docker build -t otel-collector .
docker run -p 4317:4317 -p 4318:4318 \
  -e TEMPO_ENDPOINT=http://tempo:4317 \
  -e PROMETHEUS_ENDPOINT=http://prometheus:9090/api/v1/write \
  -e LOKI_ENDPOINT=http://loki:3100/loki/api/v1/push \
  otel-collector
```

## Ports

- `4317` - OTLP gRPC receiver
- `4318` - OTLP HTTP receiver
- `13133` - Health check endpoint