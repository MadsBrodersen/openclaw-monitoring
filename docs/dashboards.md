# OpenClaw Grafana Dashboards

Dette dokument beskriver Grafana‑dashboards for OpenClaw i produktion, med fokus på
hvad hvert panel viser, hvilke queries der bruges, og hvordan de tolkes korrekt.

Dokumentet er et supplement til:
- `docs/observability.md` (arkitektur og ingestion)
- `docs/runbook.md` (driftsscenarier)


---

## 1. Overordnet dashboard‑filosofi

Dashboardene er designet efter følgende principper:

- ✅ **Kun label‑baserede LogQL‑queries**
- ✅ Ingen parsing eller regex i dashboards
- ✅ Struktur bestemmes ved ingestion (Promtail)
- ✅ Klart skel mellem:
  - Teams‑aktivitet
  - Gateway lifecycle
  - Fejl og advarsler
  - Intern støj
  - Ground truth (rå logs)

Primært produktionsdashboard:
**OpenClaw – Gateway (Production)** (v7)


---

## 2. Label‑model (kort recap)

Dashboard‑queries er baseret på disse labels:

| Label     | Eksempel   | Betydning                         |
|----------|------------|-----------------------------------|
| container | openclaw-gateway | Docker container               |
| channel   | msteams    | Kommunikationskanal               |
| subsystem | gateway    | OpenClaw subsystem                |
| module    | cron       | Intern service                    |

Labels sættes i Promtail ved ingestion.


---

## 3. Panel‑gennemgang

### 3.1 Microsoft Teams – Activity

**Formål**  
Viser rå Teams‑events i real‑time.

**Query**
```logql
{container="openclaw-gateway", channel="msteams"}
```
**Hvad du ser**
* dispatching to agent
* dispatch complete
* received message
**Normal adfærd**
* Panelet er tomt, hvis der ikke er sendt Teams‑beskeder
* Det er korrekt og forventet
### 3.2 Microsoft Teams – Message volume
**Formål**
Viser volumen af Teams‑beskeder over tid.
**Query**
```txt
count_over_time(
  {container="openclaw-gateway", channel="msteams"}
[$__interval])
```
**Noter**
* $__interval styres af dashboardets timeframe
* Grafen viser spikes ved aktivitet
* En flad graf betyder blot ingen Teams‑trafik
### 3.3 Gateway lifecycle
**Formål**
Viser gateway‑relaterede lifecycle‑events.
**Query**
```txt
{container="openclaw-gateway", subsystem="gateway"}
```
**Eksempler på events**
* Gateway startup
* HTTP server bind
* Shutdown / restart
**Normal adfærd**
* “No data” i lange perioder er helt normalt
* Gateway logger kun ved faktiske lifecycle‑ændringer
### 3.4 Warnings & Errors
**Formål**
Samler advarsler og fejl på tværs af OpenClaw‑komponenter.
**Query**
```txt
{container=~"openclaw-(gateway|agent|blackbox)"} |~ "WARN|ERROR"
```
**Bemærk**
* Indeholder både kritiske og ikke‑kritiske fejl
* Brug kontekst fra andre panels før eskalation
### 3.5 Internal subsystems (cron / plugins / health)
**Formål**
Viser aktivitet fra interne services.
**Queries**
```txt
{container="openclaw-gateway", module="cron"}
```
```txt
{container="openclaw-gateway", subsystem=~"plugins|health-monitor"}
```
**Normal adfærd**
* Ofte tomt
* Kun aktiv, når interne jobs kører
### 3.6 Raw OpenClaw logs (ground truth)
**Formål**
Viser alle OpenClaw‑logs uden filtrering.
**Query**
```txt
{container=~"openclaw-.*"}
```
**Brug**
* Fejlsøgning
* Sammenkædning af events
* Verifikation af ingestion
Dette panel er den autoritative sandhed.
### 4. Hvordan dashboards bør bruges
**Hurtigt overblik**
* Kig først på Teams Activity og Message volume
* Hvis tomt → sandsynligvis ingen trafik
**Fejlsøgning**
* Teams‑panel
* Warnings & Errors
* Gateway lifecycle
* Raw logs
**Undgå**
* At tolke “No data” som fejl uden kontekst
* At bruge regex eller parsing i dashboards
### 5. Ændringer og udvidelser
Når nye logtyper tilføjes:
* Opdater Promtail (labels)
* Brug labels i dashboards
* Undgå LogQL‑parsing
Dashboards bør altid være “dumme” og hurtige.
### 6. Konklusion
Grafana‑dashboards for OpenClaw er:
* Label‑styrede
* Produk­tionsklare
* Letlæselige
* Modstandsdygtige over for logformat‑ændringer
Dashboard v7 betragtes som stabil reference.
