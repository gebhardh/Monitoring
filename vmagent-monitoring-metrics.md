# Die 10 wichtigsten vmagent-Monitoring-Metriken

vmagent ist eine kritische Komponente in der VictoriaMetrics-Pipeline – wenn er ausfällt oder klemmt, hast du Datenlücken oder verlorene Metriken. Diese Übersicht zeigt die zehn wichtigsten Metriken mit PromQL-Queries und Alert-Schwellen.

Die Metriken selbst stellt vmagent unter `/metrics` zur Verfügung – am besten lässt du Telegraf oder vmagent selbst diese scrapen.

---

## 1. Remote-Write-Fehler

**Was**: Fehler beim Schreiben an das Storage-Backend. Das wichtigste Frühwarnsignal überhaupt.

**Metrik**: `vmagent_remotewrite_requests_failed_total`

```promql
sum by (url) (
  rate(vmagent_remotewrite_requests_failed_total[5m])
)
```

**Alert**:

```promql
sum by (url) (rate(vmagent_remotewrite_requests_failed_total[5m])) > 0.1
```

Mehr als 0.1 Fehler/s über 5 Minuten = anhaltendes Schreibproblem. Häufige Ursachen: Storage-Backend down, Auth-Fehler, Netzwerk.

---

## 2. Pending Data in der Persistent Queue

**Was**: Daten, die in der lokalen Queue auf dem Disk warten, weil sie nicht weggeschrieben werden konnten. Wächst die Queue, hast du ein Problem.

**Metrik**: `vmagent_remotewrite_pending_data_bytes`

```promql
vmagent_remotewrite_pending_data_bytes
```

**Alert (wachsende Queue)**:

```promql
deriv(vmagent_remotewrite_pending_data_bytes[10m]) > 0
  and
vmagent_remotewrite_pending_data_bytes > 100 * 1024 * 1024
```

Queue wächst und überschreitet 100 MB → akuter Handlungsbedarf, bevor `-remoteWrite.maxDiskUsagePerURL` erreicht wird.

---

## 3. Verworfene Samples wegen voller Queue

**Was**: Samples, die endgültig verloren gegangen sind, weil die Persistent Queue voll war.

**Metrik**: `vmagent_remotewrite_samples_dropped_total`

```promql
sum by (url) (
  rate(vmagent_remotewrite_samples_dropped_total[5m])
)
```

**Alert**:

```promql
sum by (url) (rate(vmagent_remotewrite_samples_dropped_total[5m])) > 0
```

Jeder Wert > 0 bedeutet **Datenverlust**. Sofort untersuchen.

---

## 4. Scrape-Fehler

**Was**: Wie viele Targets antworten nicht erfolgreich beim Scrapen?

**Metrik**: `vmagent_scrapes_failed_total` und `vm_promscrape_scrapes_failed_total`

```promql
sum by (job) (
  rate(vm_promscrape_scrapes_failed_total[5m])
)
/
sum by (job) (
  rate(vm_promscrape_scrapes_total[5m])
)
```

Liefert die Fehlerquote pro Job (0 = perfekt, 1 = alles tot).

**Alert**:

```promql
(
  sum by (job) (rate(vm_promscrape_scrapes_failed_total[5m]))
  /
  sum by (job) (rate(vm_promscrape_scrapes_total[5m]))
) > 0.1
```

Über 10 % Fehlerquote pro Job → Targets sind nicht erreichbar oder antworten falsch.

---

## 5. Anzahl aktiver Scrape-Targets

**Was**: Wie viele Targets sind aktuell aktiv? Plötzliche Einbrüche zeigen Service-Discovery-Probleme.

**Metrik**: `vm_promscrape_targets`

```promql
sum by (status) (vm_promscrape_targets)
```

`status="up"` vs. `status="down"`.

**Alert (Targets sind down)**:

```promql
sum by (job) (vm_promscrape_targets{status="down"}) > 0
```

**Alert (plötzlicher Einbruch)**:

```promql
sum(vm_promscrape_targets{status="up"})
  <
sum(vm_promscrape_targets{status="up"} offset 1h) * 0.9
```

Mehr als 10 % der Targets gegenüber vor einer Stunde verschwunden → Service Discovery oder Netzwerkproblem.

---

## 6. Scrape-Dauer

**Was**: Wie lange dauert es, ein Target zu scrapen? Lange Scrapes deuten auf überlastete Targets oder zu große Metrik-Volumina hin.

**Metrik**: `vm_promscrape_scrape_duration_seconds`

```promql
histogram_quantile(0.95,
  sum by (job, le) (
    rate(vm_promscrape_scrape_duration_seconds_bucket[5m])
  )
)
```

P95-Scrape-Dauer pro Job.

**Alert**:

```promql
histogram_quantile(0.95,
  sum by (job, le) (rate(vm_promscrape_scrape_duration_seconds_bucket[5m]))
) > 10
```

P95 > 10 s ist meist problematisch, je nach `scrape_interval`. Faustregel: Scrape-Dauer sollte deutlich unter dem Scrape-Intervall liegen.

---

## 7. Abgelehnte Samples wegen Limit (z. B. `sample_limit`)

**Was**: Wenn `sample_limit` in `scrape_configs` gesetzt ist, werden Targets mit zu vielen Metriken komplett abgelehnt.

**Metrik**: `vm_promscrape_scrapes_skipped_by_sample_limit_total`

```promql
sum by (job) (
  rate(vm_promscrape_scrapes_skipped_by_sample_limit_total[5m])
)
```

**Alert**:

```promql
sum by (job) (rate(vm_promscrape_scrapes_skipped_by_sample_limit_total[5m])) > 0
```

Bedeutet: Ein Target liefert mehr Samples als erlaubt → Cardinality-Explosion auf dem Target, oder `sample_limit` zu niedrig gesetzt.

---

## 8. Memory Usage von vmagent selbst

**Was**: vmagent hat einen aktiven In-Memory-Buffer und Caches. Bei Cardinality-Explosionen oder vielen Targets kann der RAM-Bedarf stark steigen.

**Metrik**: `process_resident_memory_bytes`

```promql
process_resident_memory_bytes{job="vmagent"}
```

**Alert (Prozent vom Limit)**:

```promql
process_resident_memory_bytes{job="vmagent"}
/
vm_available_memory_bytes
> 0.85
```

Über 85 % der verfügbaren Memory → OOM-Risiko. Bei Container-Setups stattdessen gegen das Container-Limit alerten.

---

## 9. Connection-Verhalten zu Remote-Storage

**Was**: Wie oft werden Verbindungen zum Backend wiederhergestellt? Häufige Reconnects = instabile Verbindung.

**Metrik**: `vmagent_remotewrite_retries_count_total`

```promql
sum by (url) (
  rate(vmagent_remotewrite_retries_count_total[5m])
)
```

**Alert**:

```promql
sum by (url) (rate(vmagent_remotewrite_retries_count_total[5m])) > 1
```

Mehr als 1 Retry/s deutet auf Netzwerk- oder Backend-Probleme hin – noch bevor `requests_failed_total` ansteigt.

---

## 10. Block-Send-Duration (Latenz zum Backend)

**Was**: Wie lange dauert das Versenden eines Datenblocks an das Storage? Steigende Latenz = Backend wird langsam.

**Metrik**: `vmagent_remotewrite_send_duration_seconds_total`

```promql
rate(vmagent_remotewrite_send_duration_seconds_total[5m])
```

Der Wert zeigt die kumulierte Sendezeit pro Sekunde. Wenn er sich der Anzahl der parallelen Sender (`-remoteWrite.queues`) annähert, sind die Sender ausgelastet.

**Alert (Sender-Auslastung)**:

```promql
rate(vmagent_remotewrite_send_duration_seconds_total[5m])
/
vmagent_remotewrite_queues
> 0.9
```

Über 90 % Sender-Auslastung → Backend ist der Flaschenhals. Lösung: mehr Queues, schnelleres Backend oder weniger Daten.

---

## Bonus: Compound-Health-Score

Wenn du einen einzelnen "vmagent ist gesund"-Score willst:

```promql
(
  # Keine Schreibfehler
  (sum(rate(vmagent_remotewrite_requests_failed_total[5m])) < 0.1)
  +
  # Keine verlorenen Samples
  (sum(rate(vmagent_remotewrite_samples_dropped_total[5m])) == 0)
  +
  # Queue stabil
  (deriv(sum(vmagent_remotewrite_pending_data_bytes)[10m:]) <= 0)
  +
  # Scrape-Fehlerquote unter 5 %
  (
    (sum(rate(vm_promscrape_scrapes_failed_total[5m]))
     / sum(rate(vm_promscrape_scrapes_total[5m]))) < 0.05
  )
)
```

Ergibt einen Wert zwischen 0 (alles kaputt) und 4 (alles gut). Praktisch für ein Stat-Panel mit Ampel-Färbung.

---

## Priorisierung nach Wichtigkeit

Wenn du nicht alle gleichzeitig einbauen willst, empfehle ich diese Reihenfolge:

1. **Pending Data Bytes** (#2) – Frühwarnindikator
2. **Samples Dropped** (#3) – akuter Datenverlust
3. **Remote-Write Failed** (#1) – Schreibproblem
4. **Targets Down** (#5) – Service-Discovery-Probleme
5. **Scrape Failures** (#4) – Target-Probleme

Die restlichen fünf sind dann Feintuning und Performance-Sicht.

---

## Übersichtstabelle

| # | Metrik                                               | Zweck                             | Alert-Schwelle              |
|---|------------------------------------------------------|-----------------------------------|------------------------------|
| 1 | `vmagent_remotewrite_requests_failed_total`          | Schreibfehler zum Backend         | > 0.1/s                      |
| 2 | `vmagent_remotewrite_pending_data_bytes`             | Wachsende Disk-Queue              | > 100 MB & wachsend          |
| 3 | `vmagent_remotewrite_samples_dropped_total`          | Datenverlust                      | > 0                          |
| 4 | `vm_promscrape_scrapes_failed_total`                 | Scrape-Fehlerquote                | > 10 %                       |
| 5 | `vm_promscrape_targets`                              | Anzahl aktiver Targets            | Einbruch > 10 %              |
| 6 | `vm_promscrape_scrape_duration_seconds`              | Scrape-Latenz                     | P95 > 10 s                   |
| 7 | `vm_promscrape_scrapes_skipped_by_sample_limit_total`| Sample-Limit überschritten        | > 0                          |
| 8 | `process_resident_memory_bytes`                      | RAM-Nutzung vmagent               | > 85 % verfügbar             |
| 9 | `vmagent_remotewrite_retries_count_total`            | Retries zum Backend               | > 1/s                        |
| 10| `vmagent_remotewrite_send_duration_seconds_total`    | Sender-Auslastung                 | > 90 %                       |

---

## Dashboards und Vorlagen

VictoriaMetrics stellt ein offizielles **Grafana-Dashboard für vmagent** zur Verfügung: Grafana Dashboard ID **12683**. Das deckt die meisten dieser Metriken bereits ab – guter Startpunkt, dann nach eigenen Bedürfnissen erweitern.
