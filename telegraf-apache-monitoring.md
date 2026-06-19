# Apache Access Log Auswertung mit Telegraf

## Architektur

```
Apache HTTP Server
  → /var/log/apache2/access.log (Extended Combined Format mit %D)
      → Telegraf inputs.tail + grok-Parser
          → processors.starlark  (Statuskategorie + ms-Umrechnung)
          → aggregators.basicstats (Response-Time-Statistiken)
          → aggregators.valuecounter (Statuscode-Zählung)
              → VictoriaMetrics / Prometheus / InfluxDB
```

---

## 1. Apache: LogFormat anpassen

Das Standard-Combined-Format wird um `%D` (Response-Zeit in Mikrosekunden) erweitert.

### Datei: `/etc/apache2/conf-available/telegraf-logging.conf`

```apache
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D" extended_combined
```

In den VirtualHost-Blöcken:
```apache
CustomLog /var/log/apache2/access.log extended_combined
```

Aktivieren und neu laden:
```bash
sudo a2enconf telegraf-logging
sudo systemctl reload apache2
```

### Beispiel-Logzeile

```
192.168.1.10 - - [18/Jun/2026:10:23:45 +0200] "GET /index.html HTTP/1.1" 200 1234 "-" "Mozilla/5.0" 2345
```

Das letzte Feld `2345` ist die Response-Zeit in Mikrosekunden.

---

## 2. Telegraf: Konfiguration

### Datei: `/etc/telegraf/telegraf.d/apache-access-log.conf`

#### inputs.tail – Log-Datei einlesen

```toml
[[inputs.tail]]
  name_override  = "apache_access_log"
  files          = ["/var/log/apache2/access.log"]
  from_beginning = false
  watch_method   = "inotify"
  data_format    = "grok"
  grok_timezone  = "Europe/Berlin"
  grok_patterns  = [
    "%{COMBINED_LOG_FORMAT} %{NUMBER:response_time_us:int}"
  ]
  tag_keys = ["http_method", "http_version"]
```

**Grok-Pattern Erklärung:**

| Pattern-Teil | Captured Field | Typ | Inhalt |
|---|---|---|---|
| `%{COMBINED_LOG_FORMAT}` | (integriertes Pattern) | — | Alle Standard-Felder |
| `%{NUMBER:response_time_us:int}` | `response_time_us` | Integer | Response-Zeit in µs |

**Felder aus `COMBINED_LOG_FORMAT`:**

| Field | Typ | Bedeutung |
|---|---|---|
| `client_ip` | string | Client-IP-Adresse |
| `http_method` | string (Tag) | GET, POST, … |
| `request` | string | Angeforderter Pfad |
| `http_status_code` | int | 200, 404, 500, … |
| `response_bytes` | int | Größe der Antwort |
| `referrer` | string | Referrer-URL |
| `agent` | string | User-Agent |
| `http_version` | float (Tag) | 1.1 / 2.0 |

#### processors.starlark – Anreicherung

Leitet aus `http_status_code` das Tag `status_category` ab:

```toml
[[processors.starlark]]
  namepass = ["apache_access_log"]
  source = '''
def apply(metric):
    code = int(metric.fields.get("http_status_code", 0))
    if   code >= 500: cat = "5xx"
    elif code >= 400: cat = "4xx"
    elif code >= 300: cat = "3xx"
    elif code >= 200: cat = "2xx"
    else:             cat = "other"
    metric.tags["status_category"] = cat
    rt_us = metric.fields.get("response_time_us", None)
    if rt_us is not None:
        metric.fields["response_time_ms"] = float(rt_us) / 1000.0
    return metric
'''
```

#### aggregators.basicstats – Response-Time-Statistiken

Erzeugt je 60-Sekunden-Fenster:

| Feld | Bedeutung |
|---|---|
| `response_time_us_count` | Anzahl Requests im Intervall |
| `response_time_us_mean` | Durchschnittliche Response-Zeit (µs) |
| `response_time_us_max` | Maximale Response-Zeit (µs) |
| `response_time_us_min` | Minimale Response-Zeit (µs) |
| `response_time_ms_mean` | Durchschnittliche Response-Zeit (ms) |

#### aggregators.valuecounter – Statuscode-Zählung

Erzeugt Messung `apache_status_count`:

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `http_status_code_200` | 1523 | Anzahl 200-Antworten |
| `http_status_code_404` | 12 | Anzahl 404-Antworten |
| `http_status_code_500` | 3 | Anzahl 500-Antworten |

---

## 3. Deployment

### Telegraf starten und testen

```bash
# Konfiguration in Telegraf-Drop-in-Verzeichnis kopieren
sudo cp telegraf-apache-access-log.conf /etc/telegraf/telegraf.d/

# Konfiguration validieren
telegraf --config /etc/telegraf/telegraf.conf \
         --config-directory /etc/telegraf/telegraf.d \
         --test 2>&1 | head -30

# Debug: Output direkt in stdout (outputs.file aktivieren, dann):
telegraf --config /etc/telegraf/telegraf.d/apache-access-log.conf \
         --debug

# Dienst neu starten
sudo systemctl restart telegraf
sudo journalctl -u telegraf -f
```

### Berechtigungen prüfen

```bash
# Telegraf-User muss das Log lesen dürfen
sudo usermod -aG adm telegraf
# oder
sudo setfacl -m u:telegraf:r /var/log/apache2/access.log
```

---

## 4. Abfrage-Beispiele

### Prometheus / VictoriaMetrics (PromQL)

```promql
# Requests pro Statuskategorie (Rate über 5 Minuten)
rate(apache_access_log_http_status_code[5m])

# Nur 5xx-Fehler zählen
sum(rate(apache_status_count_http_status_code_500[5m]))

# Alle 4xx+5xx zusammen
sum(rate(apache_access_log{status_category=~"4xx|5xx"}[5m]))

# Durchschnittliche Response-Zeit in ms
avg(apache_access_log_response_time_ms_mean)

# P95 Response-Zeit (wenn Histogramm-Aggregator konfiguriert)
histogram_quantile(0.95, rate(apache_access_log_response_time_ms_bucket[5m]))
```

### InfluxDB (Flux)

```flux
from(bucket: "apache_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "apache_access_log")
  |> filter(fn: (r) => r.status_category == "5xx")
  |> aggregateWindow(every: 1m, fn: count)
```

---

## 5. Troubleshooting

| Problem | Ursache | Lösung |
|---|---|---|
| Keine Metriken | Grok-Pattern passt nicht | `--test` Flag + Debug-Output aktivieren |
| `response_time_us` fehlt | Apache ohne `%D` konfiguriert | `extended_combined` LogFormat aktivieren |
| Permission denied | Telegraf darf Log nicht lesen | `usermod -aG adm telegraf` |
| Falsche Zeitstempel | Zeitzone falsch | `grok_timezone = "Europe/Berlin"` prüfen |
| Starlark-Fehler | Python-Syntax in Starlark | `int()` statt direkter Cast verwenden |

### Grok-Pattern testen

```bash
# Einzelne Log-Zeile testen
echo '192.168.1.10 - - [18/Jun/2026:10:23:45 +0200] "GET /index.html HTTP/1.1" 200 1234 "-" "Mozilla/5.0" 2345' \
  > /tmp/test-apache.log

telegraf --config /etc/telegraf/telegraf.d/apache-access-log.conf \
         --input-filter tail --test
```

Online-Tools zum Pattern-Testen:
- https://grokdebug.herokuapp.com
- https://www.debuggex.com
