# OpenClaw Monitoring

Dette repository indeholder observability‑setup for **OpenClaw**, herunder:

- Log ingestion med **Promtail**
- Log storage og søgning i **Loki**
- Visualisering i **Grafana**
- Fokus på **Microsoft Teams‑kanalen** og gateway‑drift

Målet er et stabilt, label‑baseret observability‑setup uden parsing og regex i dashboards.


---

## 📊 Hvad overvåges?

- **Microsoft Teams‑aktivitet**
  - Modtagne beskeder
  - Dispatch til agent
  - Message volume over tid
- **OpenClaw Gateway lifecycle**
  - Startup / shutdown
  - HTTP server events
- **Warnings & errors**
- **Interne services**
  - Cron jobs
  - Plugins
  - Health monitors
- **Raw logs (ground truth)**

Alt logik for struktur og labels håndteres **én gang i Promtail**.


---

## 📁 Repository‑struktur

```text
monitoring/
├─ promtail/
│  └─ promtail.yml          # Log ingestion & label enrichment
│
├─ grafana/
│  └─ dashboards/
│     └─ openclaw-gateway-prod-v7.json
│
docs/
├─ observability.md         # Arkitektur & designvalg
├─ runbook.md               # Drift & fejlsøgning
└─ dashboards.md            # Dashboard‑forklaring
```
### 📚 Dokumentation
Start her, hvis du er ny i setup’et:
* 🔍 Arkitektur & observability‑design
  → docs/observability.md
* 🛠️ Drift & fejlsøgning (runbook)
  → docs/runbook.md
* 📈 Grafana dashboards & LogQL‑queries
  → docs/dashboards.md

### 🧠 Designprincipper
* ✅ Logs parses i Promtail, ikke i dashboards
* ✅ Kun label‑baserede LogQL‑queries
* ✅ Ingen regex eller escaping i Grafana
* ✅ JSON‑i‑JSON håndteres korrekt via JMESPath
* ✅ Dashboards afspejler arkitektur, ikke logformat
Dette reducerer kompleksitet og øger stabilitet i drift.

### ✅ Status
* Observability‑setup er production‑ready
* Promtail pipeline er verificeret via dry‑run
* Dashboard v7 er canonical reference
