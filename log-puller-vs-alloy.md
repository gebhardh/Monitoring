# Kann Alloy den log-puller ersetzen?

**Kurze Antwort: Teilweise – die zentralen Funktionen ja, aber das Pull-Modell ist konzeptionell nicht das, wofür Alloy gemacht ist.**

## Was Alloy gut kann

Alloy ist der konsolidierte Nachfolger von Promtail, Grafana Agent und mehreren anderen Tools. Es hat eine eigene Konfigurationssprache (River/Alloy) und ein Komponentenmodell, ähnlich Vector oder Fluent Bit. Für **klassisches Push-basiertes Log-Shipping** ist es eine sehr gute Wahl.

Konkret deckt Alloy ab:

- `loki.source.journal` liest **lokales** systemd-Journal direkt
- `loki.source.file` für Datei-Tailing
- `loki.relabel` für Label-Transformation
- `loki.write` als Sink (auch mit mTLS)
- TLS/mTLS sowohl in Quellen als auch in Senken
- Konfigurierbare Filter (`matches = "_SYSTEMD_UNIT=nginx.service PRIORITY=3"`)
- Hot-Reload der Konfiguration

Für deinen Anwendungsfall – Logs sammeln und zu VictoriaLogs/Loki schicken – ist Alloy konzeptionell genau das richtige Tool.

## Wo es entscheidend hakt: das Pull-Modell

Hier wird die Antwort differenziert. Der log-puller läuft **zentral** und holt sich Logs von vielen Hosts ab. Alloy ist primär als **dezentraler Agent pro Host** gedacht.

**Standard-Alloy-Architektur:**

```
Quell-Host 1:  Alloy ─push─►
Quell-Host 2:  Alloy ─push─► Loki/VictoriaLogs
Quell-Host N:  Alloy ─push─►
```

**Deine log-puller-Architektur:**

```
Quell-Host 1: journal-gatewayd ◄─pull─┐
Quell-Host 2: journal-gatewayd ◄─pull─┤ log-puller (zentral) ─►Sink
Quell-Host N: journal-gatewayd ◄─pull─┘
```

Es gibt in Alloy **keine direkte Entsprechung** für "HTTP-Pull gegen journal-gatewayd mit Cursor-State". Die relevanten Komponenten:

- `loki.source.journal` liest **nur lokal** über libsystemd (kein HTTP)
- `prometheus.scrape` macht zwar HTTP-Pull, aber für Prometheus-Metriken, nicht für Journal-Daten
- Eine generische `http.scrape`-Komponente, die journal-gatewayd's Format versteht und Cursor verwaltet, existiert nicht

## Die zwei realistischen Migrationswege

### Weg A: Architektur ändern – Alloy auf jedem Quell-Host (empfohlen, wenn Topologie erlaubt)

Das ist die "konventionelle" Lösung. Statt zentralem Pull von außen läuft Alloy auf jedem Host und liest das lokale Journal:

```hcl
// config.alloy auf jedem Quell-Host
loki.source.journal "read" {
  forward_to = [loki.write.sink.receiver]
  matches    = "_SYSTEMD_UNIT=nginx.service"
  labels     = { role = "webserver", env = "prod" }
}

loki.relabel "extract_fields" {
  forward_to = [loki.write.sink.receiver]
  rule {
    source_labels = ["__journal__systemd_unit"]
    target_label  = "unit"
  }
  rule {
    source_labels = ["__journal_priority_keyword"]
    target_label  = "level"
  }
}

loki.write "sink" {
  endpoint {
    url = "https://victorialogs.internal:9428/insert/loki/api/v1/push"
    tls_config {
      ca_file   = "/etc/alloy/certs/ca.crt"
      cert_file = "/etc/alloy/certs/client.crt"
      key_file  = "/etc/alloy/certs/client.key"
    }
  }
}
```

**Dabei deckt Alloy diese log-puller-Features ab:**

| log-puller-Feature | Alloy-Entsprechung |
|---|---|
| Endpunkt-Konfiguration | `loki.source.journal` Komponente pro Host |
| Filter (`ENDPOINT_X_FILTER`) | `matches` Argument |
| Labels (`ENDPOINT_X_LABELS`) | `labels` Argument + `loki.relabel` |
| Sink (jsonline/loki) | `loki.write` (Loki-Format) |
| mTLS zum Sink | `tls_config` in `loki.write` |
| Cursor-Persistenz | Eingebaut, in `storage.path` |
| systemd-Service | Wird per Debian-Paket installiert |

**Was du dabei aufgibst:**

- **Zentrale Konfiguration**: Jeder Host braucht eigene Alloy-Konfig. Dafür gibt es Tools wie Ansible/Salt, aber ein Overhead bleibt.
- **Zentrale Wartung**: Alloy-Updates auf N Hosts statt nur einem.
- **Push-statt-Pull-Architektur**: kann gegen Firewall-Politik laufen, in der ausgehende Verbindungen vom Quell-Host beschränkt sind.

### Weg B: Architektur beibehalten – Alloy ist nicht das richtige Tool

Wenn du beim Pull-Modell bleiben **musst** (Firewall-Topologie, zentrale Verwaltung, einheitliche Cert-Pflege etc.), passt Alloy einfach nicht. Workarounds:

1. **Alloy mit `prometheus.scrape`-Hack**: Theoretisch könnte man journal-gatewayd als HTTP-Endpoint behandeln, aber es liefert keine Prometheus-Metriken, sondern JSON-Lines mit Cursors. Funktioniert nicht ohne weiteres.
2. **Alloy zentral + SSH-Mount**: SSHFS oder NFS, sodass `/var/log/journal` vom Quell-Host auf dem Alloy-Host gemountet ist. Sehr fragil, nicht empfehlenswert.
3. **Bei log-puller bleiben** für das Pull-Modell und Alloy/Promtail für andere Anwendungen.

## Feature-für-Feature-Vergleich

| Feature | log-puller | Alloy (Weg A) |
|---|---|---|
| Pull-Modell (zentral holt von vielen Hosts) | ✅ | ❌ |
| Push-Modell (Agent auf jedem Host) | ❌ | ✅ |
| systemd-Journal-Quelle | ✅ via journal-gatewayd HTTP | ✅ via libsystemd direkt |
| Cursor/State-Management | ✅ pro Endpunkt | ✅ eingebaut |
| Per-Endpoint-Filter | ✅ | ✅ (`matches`) |
| Per-Endpoint-Labels | ✅ | ✅ (`labels` + `relabel`) |
| TLS_MODE=off (HTTP) | ✅ explizit konfigurierbar | ✅ einfach http:// URL |
| TLS_MODE=insecure | ✅ explizit konfigurierbar | ✅ `insecure_skip_verify` |
| mTLS | ✅ | ✅ (`tls_config`) |
| Sink-Typ jsonline (VictoriaLogs) | ✅ | ✅ (über loki-kompatiblen Endpoint) |
| Sink-Typ Loki | ✅ | ✅ (nativ) |
| `_msg`-Feld für VictoriaLogs | ✅ automatisch | ⚠️ via Relabel selbst bauen |
| Konfiguration `/etc/default/` | ✅ Shell-Syntax | ❌ Eigenes Format (.alloy/River) |
| Debian-Paket (offiziell) | ✅ eigenes | ✅ offizielles Grafana-Paket |
| Hot-Reload | ❌ Restart nötig | ✅ |
| Web-UI für Debugging | ❌ | ✅ Port 12345 |
| Prometheus-Metriken zum Selbst-Monitoring | ❌ noch nicht | ✅ eingebaut |
| Ressourcen-Footprint | Sehr gering (~30 MB RAM) | Mittel (~80-150 MB RAM) |
| Komplexität | Klein, lesbarer Python-Code | Groß, viel Funktionalität |
| Wartung durch Grafana Labs | – | ✅ aktive Entwicklung |

## Empfehlung

Es hängt davon ab, **warum** du ursprünglich das Pull-Modell wolltest:

**Wenn der Hauptgrund Pull war (Firewall, zentrale Pflege):**

- Bei log-puller bleiben. Alloy bringt hier keinen Vorteil und passt nicht.
- Eventuell den log-puller um die fehlenden Quality-of-Life-Features ergänzen (Prometheus-Metriken zum Selbst-Monitoring, Hot-Reload via SIGHUP).

**Wenn Pull eher historisch gewachsen war (du wolltest die Konsistenz zu Metriken-Pull):**

- Wechsel zu Alloy als Push-Agent erwägen. Vorteile: ausgereifter, viele Features, professionelle Wartung, Web-UI fürs Debugging.
- Migration ist nicht trivial wegen Konfiguration pro Host – mit Ansible o.ä. aber gut machbar.

**Hybrid-Ansatz:**

- log-puller für die paar Hosts, die nur per Pull erreichbar sind (Legacy, eingeschränkte Topologie)
- Alloy für alles andere (Standardfall)

Das ist kein Entweder-oder – beide Tools können nebeneinander dieselbe Sink-Instanz beliefern. log-puller schreibt jeden Eintrag mit `_endpoint=...`, Alloy mit `host=...` – in VictoriaLogs/Loki kannst du anhand der Labels filtern.

## Eine konkrete Frage zurück

Was war für dich der ausschlaggebende Grund für das Pull-Modell? Wenn das primär die **Firewall-Topologie** war (Push-Verbindungen vom Quell-Host nicht erlaubt), ist log-puller weiterhin die richtige Wahl. Wenn es eher die **Einheitlichkeit zum Prometheus-Pull-Modell** war, lohnt sich ein zweiter Blick auf Alloy – das ist heute der etabliertere Weg im Loki/Grafana-Ökosystem.
