# OpenClaw Runbook

Denne runbook beskriver de mest almindelige driftsscenarier for OpenClaw
og hvordan de hurtigt verificeres via Grafana og Loki.

Formålet er at kunne:
- Afklare om noget reelt er i stykker
- Skelne mellem normal og unormal adfærd
- Finde roden til et problem hurtigt


---

## 1. Teams‑beskeder ses ikke i dashboardet

### Symptom
- "Microsoft Teams – Activity" viser ingen logs
- Message volume‑grafen er flad

### Vigtigt først
Det er **normal adfærd**, at panelet er tomt, hvis der ikke er sendt Teams‑beskeder.

Gateway logger **kun ved events**.

### Hurtig verifikation

Send én test‑besked i Teams og kør i Grafana → Explore → Loki:

```logql
{container="openclaw-gateway", channel="msteams"}
```
### Forventet resultat

*Loglinjer som:
  * dispatching to agent
  * dispatch complete
  * received message

Hvis ja → systemet virker korrekt.
## 2. Der kommer logs, men kun fra openclaw-agent
### Symptom
* {container=~".*openclaw.*"} viser logs
* {container="openclaw-gateway"} gør ikke
### Forklaring
* openclaw-agent logger kontinuerligt
* openclaw-gateway logger kun ved trafik
Dette er forventet adfærd.
### Verifikation
```txt
{container=~".*openclaw.*"} | line_format "{{.container}}"
```
Hvis openclaw-gateway ikke ses → der har ikke været gateway‑events.
## 3. Promtail ser ud til ikke at sætte labels
### Symptom
* Teams‑logs ses i raw logs
* Query med channel="msteams" giver ingen hits
### Check 1: virker ingestion?
```txt
{container="openclaw-gateway"} | line_format "{{.container}} {{.channel}} {{.subsystem}} {{.module}}"
```
### Check 2: dry‑run (Promtail)
```bash
echo '{"log":"{\"0\":\"{\\\"name\\\":\\\"msteams\\\"}\",\"1\":\"dispatch complete\"}","time":"2026-04-15T12:00:00+02:00"}' \
| docker exec -i monitoring-promtail-1 \
  promtail \
  -stdin \
  -dry-run \
  -server.disable=true \
  -config.file=/etc/promtail/promtail.yml
```
Forvent:
```txt
{channel="msteams"}
```
Hvis ikke → tjek promtail.yml.
## 4. Gateway lifecycle‑panel er tomt
### Symptom
* "Gateway lifecycle" viser "No data"
### Forklaring
Gateway lifecycle‑logs skrives kun ved startup, shutdown eller rebind.

"No data" betyder: ✔ Gateway'en har ikke skiftet tilstand
Dette betragtes som sund adfærd.
## 5. Se alt rå data (ground truth)
Når noget er uklart, brug altid raw logs‑panelet:
```txt
{container=~"openclaw-.*"}
```
Dette panel er den autoritative sandhed.
## 6. Kendte normale fejlmeddelelser
Nogle fejl ses ofte og betyder ikke nødvendigvis driftfejl:
* error: required option '-m, --message <text>' not specified
* Periodiske probe‑fejl fra blackbox
Disse skal kun reageres på, hvis de korrelerer med funktionelt nedbrud.
## 7. Hvornår er der reelt et problem?
Reagér kun hvis:
* Teams‑beskeder bliver ikke behandlet
* Gateway genstarter gentagne gange
* Message volume pludseligt stopper i længere tid end forventet
Ellers er stilhed normalt.
## 8. Eskalation
Hvis problemer ikke kan forklares via denne runbook:
1. Tjek promtail logs
2. Tjek Loki ingestion
3. Tjek OpenClaw gateway container logs
4. Eskalér til dybere debugging
## OpenClaw – Admin-adgang
### Admin-URL
OpenClaw stiller et browserbaseret administrativt interface (Canvas) til rådighed på:
```url
https://openclaw.brodersen.cloud/admin/
```
Dette er **det eneste understøttede administrative entrypoint**.
---
### Autentificering
- Admin-interfacet er beskyttet med **HTTP Basic Auth**
- Autentificering håndteres udelukkende af **Nginx**
- Brugere defineres i filen:
```bash
/srv/docker/data/nginx/.openclaw.htpasswd
```
- Filen mountes read-only ind i Nginx-containeren

---

### Sikkerhedsmodel

- OpenClaw eksponerer ikke admin-interfacet direkte
- Nginx fungerer som eneste sikkerheds- og auth-boundary
- Interne OpenClaw-stier såsom `/__openclaw__/` og `/control/` er bevidst blokeret
  
“Uautoriseret adgang til /admin/ omdirigeres til en login‑side, som anvender HTTP Basic Auth via Nginx.”
---
### Noter
- Browsere kan cache HTTP Basic Auth-credentials
- Ved uventet adfærd kan et privat browser-vindue anvendes
- Admin-adgang bør kun gives til betroede brugere
