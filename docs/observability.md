# OpenClaw Observability

This document describes the production observability setup for OpenClaw, including log ingestion, labeling, and Grafana dashboards.

The goals are:
- Clear visibility into Microsoft Teams traffic
- Clean separation between gateway, agent, and internal noise
- Stable log querying without LogQL hacks or dashboard-side parsing


## Architecture

OpenClaw logs are collected and visualized using the following pipeline:

OpenClaw (Docker containers)
→ Docker JSON logs
→ Promtail (parsing and labeling)
→ Loki (log storage)
→ Grafana (dashboards)


## OpenClaw Log Structure

OpenClaw emits structured JSON logs where the field `"0"` contains a JSON-encoded string describing the event context.

### Microsoft Teams events

```json
{
  "0": "{\"name\":\"msteams\"}",
  "1": "dispatch complete",
  "_meta": { "...": "..." }
}
```
These logs represent Microsoft Teams activity and are emitted by openclaw-gateway.
## Gateway lifecycle events
```json
{
  "0": "{\"subsystem\":\"gateway\"}",
  "1": "starting HTTP server",
  "_meta": { "...": "..." }
}
```
Lifecycle logs are only written when meaningful state changes occur.
## Internal services (cron, plugins, health)
```json
{
  "0": "{\"module\":\"cron\"}",
  "1": "cron: started",
  "_meta": { "...": "..." }
}
```
## Promtail Ingestion and Labels
Promtail parses logs in two stages:
1. Parse the outer JSON emitted by OpenClaw
2. Parse the JSON string contained in field "0"

From this, the following labels are extracted:

|Label|Description|
|--------------|-------------|
|channel|Communication channel (e.g. msteams)|
|subsystem|OpenClaw subsystem (e.g. gateway)|
|module|Internal service (e.g. cron)|

Labels are only set when the corresponding field exists.

Reference configuration:

monitoring/promtail/promtail.yml
## Grafana Dashboards
Primary production dashboard:

**OpenClaw – Gateway (Production) (v7)**

The dashboard is fully label-driven and contains:

* Microsoft Teams activity
* Teams message volume
* Gateway lifecycle
* Warnings and errors
* Internal subsystems
* Raw logs (ground truth)
## Example Queries
Microsoft Teams activity:
```txt
{container="openclaw-gateway", channel="msteams"}
```
Teams message volume:
```txt
count_over_time(
  {container="openclaw-gateway", channel="msteams"}
[$__interval])
```
Gateway lifecycle:
```txt
{container="openclaw-gateway", subsystem="gateway"}
```
Cron jobs:
```txt
{container="openclaw-gateway", module="cron"}
```
## Expected Behavior
* openclaw-agent logs continuously
* openclaw-gateway logs only on actual events
* It is normal for gateway panels to show "No data" when no Teams activity occurs

## Quick Verification
Verify label ingestion:
```txt
{container="openclaw-gateway"} | line_format "{{.container}} {{.channel}} {{.subsystem}} {{.module}}"
```
Verify Teams ingestion:
1. Send a Teams message
2. Expect events within seconds:
```txt
{container="openclaw-gateway", channel="msteams"}
```
## Known Pitfalls
* Promtail JSON parsing uses JMESPath
  * Numeric keys must be quoted (e.g. "0")
* Labels cannot be added retroactively
* Dashboards must not parse raw log text
## Conclusion
This observability setup follows these principles:
* Parsing and structure handled once in Promtail
* Dashboards rely solely on labels
* JSON-in-JSON logs are handled correctly
* The system is robust against log format changes
The setup is considered production-ready.
