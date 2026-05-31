# Telegraf-Metriken an Splunk senden

Das Telegraf HTTP-Output-Plugin ermöglicht direktes Streaming von Telegraf-Metriken
in Splunk via HTTP Event Collector (HEC). Zusammen mit dem spezialisierten Splunk
Metrics Serializer wird eine effiziente Datenaufnahme in Splunk-Metriken-Indizes
sichergestellt.

Es gibt **drei Wege** – je nach Splunk-Setup:

---

## Weg 1: Direkt via HEC (empfohlen)

### Schritt 1: HEC-Token in Splunk erstellen

```
Splunk Web → Settings → Data Inputs → HTTP Event Collector
→ New Token
→ Name: telegraf
→ Source type: telegraf (oder _json)
→ Index: telegraf  (Metriken-Index, nicht Events-Index!)
→ Token kopieren (z. B. c386d4c8-8b50-4178-be76-508dca2f19e2)
```

> **Wichtig:** Den Index als **Metriken-Index** anlegen:
> `Settings → Indexes → New Index → Index Data Type: Metrics`

### Schritt 2: Splunk inputs.conf (auf dem Splunk-Server)

```ini
# /opt/splunk/etc/system/local/inputs.conf
[http://Telegraf]
disabled = 0
index = telegraf
indexes = telegraf
token = c386d4c8-8b50-4178-be76-508dca2f19e2
```

### Schritt 3: Telegraf-Konfiguration

```toml
# /etc/telegraf/telegraf.d/splunk_output.conf

[[outputs.http]]
  ## Splunk HEC URL
  url = "https://splunk.meine-firma.de:8088/services/collector"

  ## Timeout
  timeout = "5s"

  ## HTTP-Methode
  method = "POST"

  ## TLS (Produktionsumgebung)
  # tls_ca   = "/etc/telegraf/ca.pem"
  # tls_cert = "/etc/telegraf/cert.pem"
  # tls_key  = "/etc/telegraf/key.pem"

  ## TLS-Verifikation überspringen (nur Test/Dev!)
  insecure_skip_verify = false

  ## HEC-Authentifizierung via Header
  [outputs.http.headers]
    Content-Type  = "application/json"
    Authorization = "Splunk c386d4c8-8b50-4178-be76-508dca2f19e2"

  ## Splunk Metriken-Serializer
  data_format = "splunkmetric"

  ## HEC-Routing aktivieren (Pflicht für Splunk Metriken-Index)
  [outputs.http.splunkmetric]
    splunkmetric_hec_routing = true
```

Das Flag `splunkmetric_hec_routing = true` modifiziert das JSON so, dass wichtige
Felder wie `time` und `host` in einem Wrapper um das eigentliche Event herum
platziert werden – was für den Splunk HEC erforderlich ist.

---

## Weg 2: Via Datei + Universal Forwarder

Telegraf schreibt Metriken in eine Datei, die vom Splunk Universal Forwarder
überwacht wird – wie eine normale Log-Datei.

### Telegraf-Konfiguration

```toml
[[outputs.file]]
  files = ["/var/log/telegraf/metrics.out"]
  data_format = "splunkmetric"
  [outputs.file.splunkmetric]
    splunkmetric_hec_routing = true
```

### Splunk Universal Forwarder – inputs.conf

```ini
[monitor:///var/log/telegraf/metrics.out]
index = telegraf
sourcetype = telegraf
```

### props.conf (auf Indexer oder Universal Forwarder)

```ini
[telegraf]
SHOULD_LINEMERGE = false
KV_MODE = json
```

---

## Weg 3: Via Kafka (Hochlast-Umgebungen)

Für skalierbare Hochlast-Deployments: Telegraf → Kafka → Kafka Connect Splunk
Sink Connector → Splunk HEC. Dieser Weg ist resilienter und für alle Splunk
on-premise- oder Splunk-Cloud-Plattformen geeignet.

### Telegraf → Kafka

```toml
[[outputs.kafka]]
  brokers = ["kafka1:9092", "kafka2:9092"]
  topic = "telegraf-metrics"
  data_format = "splunkmetric"
  [outputs.kafka.splunkmetric]
    splunkmetric_hec_routing = true
```

### Kafka Connect – Splunk Sink Connector

```properties
# connect-splunk-sink.properties
name=splunk-sink
connector.class=com.splunk.kafka.connect.SplunkSinkConnector
tasks.max=2
topics=telegraf-metrics
splunk.hec.uri=https://splunk:8088
splunk.hec.token=c386d4c8-8b50-4178-be76-508dca2f19e2
splunk.hec.ssl.validate.certs=true
```

---

## Datenformat im Splunk Metriken-Index

Der Splunk Metrics Serializer gibt Daten im Splunk HEC JSON-Format aus.
Der Serializer bündelt mehrere Metriken in einem HTTP-POST, sodass nicht
pro Metrik ein separater Request gesendet wird.

Beispiel-Output (CPU-Metrik):

```json
{
  "time": 1529708430,
  "event": "metric",
  "host": "mein-server",
  "index": "telegraf",
  "fields": {
    "_value": 0.6,
    "cpu": "cpu0",
    "metric_name": "cpu.usage_user"
  }
}
```

Mehrere Metriken im Batch:

```json
{"time":1529708430,"event":"metric","host":"mein-server","fields":{"_value":0.6,"metric_name":"cpu.usage_user","cpu":"cpu0"}}
{"time":1529708430,"event":"metric","host":"mein-server","fields":{"_value":2048,"metric_name":"mem.used","unit":"bytes"}}
```

---

## Gesamtarchitektur

```
Netzwerkgeräte / Linux-Server
        │
    Telegraf
    (inputs: netflow, snmp, gnmi, system, ...)
        │
        ├──→ Weg 1: outputs.http ──→ Splunk HEC (Port 8088)
        │                              │
        │                         Splunk Metriken-Index
        │
        ├──→ Weg 2: outputs.file ──→ /var/log/telegraf/metrics.out
        │                              │
        │                         Splunk Universal Forwarder
        │                              │
        │                         Splunk Indexer
        │
        └──→ Weg 3: outputs.kafka ──→ Kafka Broker
                                        │
                                  Kafka Connect
                                  (Splunk Sink)
                                        │
                                   Splunk HEC
```

---

## Häufige Fehler und Lösungen

| Problem | Ursache | Lösung |
|---|---|---|
| `bytes received but no bytes indexed` | Falscher Index-Typ | Metriken-Index statt Events-Index anlegen |
| `401 Unauthorized` | Falscher HEC-Token | Token im Header prüfen: `Splunk <token>` |
| `400 Bad Request` | Falsches Datenformat | `splunkmetric_hec_routing = true` setzen |
| TLS-Fehler | Selbst-signiertes Zertifikat | `insecure_skip_verify = true` (nur Dev!) |
| Keine Daten im Index | HEC global nicht aktiviert | `Settings → Data Inputs → HTTP Event Collector → Global Settings → Enabled` |
| Daten kommen an, aber kein Index-Eintrag | Index im Token nicht gesetzt | HEC-Token mit korrektem Ziel-Index verknüpfen |
| Mehrere Tokens, sporadisch fehlende Daten | HEC-Kapazität überschritten | Anzahl HEC-Threads erhöhen oder Kafka vorschalten |

---

## Verifikation

### HEC direkt testen (curl)

```bash
curl -k https://splunk:8088/services/collector \
  -H "Authorization: Splunk c386d4c8-8b50-4178-be76-508dca2f19e2" \
  -H "Content-Type: application/json" \
  -d '{"event":"metric","fields":{"metric_name":"test.value","_value":1}}'

# Erwartete Antwort:
# {"text":"Success","code":0}
```

### HEC-Status prüfen

```bash
curl -k https://splunk:8088/services/collector/health
# Antwort: {"text":"HEC is healthy","code":17}
```

### Telegraf-Log auf Fehler prüfen

```bash
journalctl -u telegraf -f | grep -iE "splunk|http|error"
```

### Daten in Splunk suchen

```spl
| mcatalog values(metric_name) WHERE index=telegraf
| head 20
```

```spl
| mstats avg(_value) WHERE index=telegraf metric_name="cpu.usage_user"
  BY host span=1m
```

---

## Splunk Technology Add-on (TA) für Telegraf

Für vorgefertigte Dashboards und Field Extractions steht ein Community-TA bereit:

```
Splunk Apps → Browse More Apps → Suche: "TA-influxdata-telegraf"
→ Installieren auf Search Heads
```

Das TA liefert:
- Vorgefertigte Dashboards für CPU, Memory, Disk, Network
- Field Extractions für Telegraf-Metriken
- Beispiel-Saved-Searches und Alerts

---

## Quellen

- Telegraf Splunk Metrics Serializer Dokumentation:
  https://docs.influxdata.com/telegraf/v1/data_formats/output/splunkmetric/
- Splunk Blog – Getting Started with Telegraf and Splunk:
  https://www.splunk.com/en_us/blog/it/the-daily-telegraf-getting-started-with-telegraf-and-splunk.html
- Splunk Blog – Splunk Metrics via Telegraf:
  https://www.splunk.com/en_us/blog/it/splunk-metrics-via-telegraf.html
- InfluxData – HTTP and Splunk Integration:
  https://www.influxdata.com/integrations/http-splunk/
- GitHub – TA-influxdata-telegraf:
  https://github.com/guilhemmarchand/TA-influxdata-telegraf
- Telegraf GitHub – splunkmetric Serializer:
  https://github.com/influxdata/telegraf/tree/master/plugins/serializers/splunkmetric
