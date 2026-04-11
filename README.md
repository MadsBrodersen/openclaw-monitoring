# OpenClaw Monitoring Suite

Denne repository indeholder hele overvågningspakken til:
- OpenClaw Gateway
- Nginx reverse proxy
- Docker host metrics
- Log centralization (Loki + Promtail)
- Dashboards (Grafana)
- Alerting (Prometheus + Grafana)

## Indhold

- Grafana provisionering (datasources, dashboards, alerts)
- Prometheus konfiguration og alert rules
- Loki + Promtail log setup
- Monitoring docker-compose fil
- Custom OpenClaw PRO dashboards

## Installation

### 1. Upload ZIP til serveren

```bash
scp openclaw-monitoring.zip root@<server>:/tmp/
```
