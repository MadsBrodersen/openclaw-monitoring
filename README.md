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

### 2. Udpak filerne

```bash
cd /srv/docker
unzip /tmp/openclaw-monitoring.zip -d monitoring/
```

### 3. Start monitoring stack

```bash
docker compose -f docker-compose.monitoring.yml up -d
```

### 4. Login til Grafana
http://<server-ip>:3000<br>
<strong>User:</strong> admin<br>
<strong>Password:</strong> admin</server-ip>
Dashboard mappen OpenClaw Monitoring oprettes automatisk.

## Alerts

Prometheus og Grafana indeholder:

* OpenClaw backend health
* WebSocket health
* Token/pairing errors
* Nginx upstream latency + 5xx alerts
* Host CPU/disk alerts
* Log-baserede alerts (Loki)

## Support
Kontakt: mads@brodersen.cloud
