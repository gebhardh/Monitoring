# Cisco SNA – Skalierung für ~400.000 IPs

---

## 1. Einordnung: Was bedeutet „400.000 IPs" für SNA?

SNA unterscheidet zwei wichtige Dimensionen:

- **Unique Inside Hosts** – Anzahl der intern überwachten IP-Adressen (hier: ~400.000)
- **Flows per Second (FPS)** – Anzahl der NetFlow-Datensätze pro Sekunde
  (abhängig von Netzwerkaktivität, nicht direkt von der Host-Anzahl)

400.000 IPs sind eine **mittelgroße Enterprise-Umgebung**. Mit aktiviertem Analytics
(ML-basierte Erkennung) unterstützt ein Single Node Data Store (DN6300, M6-Hardware)
bis zu 500.000 FPS und 2,0 Millionen Unique Hosts. Ein Three Node Data Store (DN6300,
M6) mit vier Flow Collectors unterstützt bis zu 1 Million FPS bei 1,0 Million Unique
Hosts. 400.000 IPs passen damit gut in einen **Single Node Data Store**, vorausgesetzt
die Flow-Rate bleibt im Rahmen.

---

## 2. Schritt 1: Flow-Rate schätzen

Als Faustformel für typische Umgebungen:

| Netzwerktyp | Flows/s pro 10.000 Hosts | Hochrechnung bei 400.000 Hosts |
|---|---|---|
| Campus (Office) | 2.000–5.000 | ~80.000–200.000 FPS |
| Rechenzentrum | 10.000–50.000 | bis 2.000.000 FPS |
| Gemischt (Campus + DC) | 5.000–15.000 | ~200.000–600.000 FPS |

> **Hinweis:** Flow Sampling (z. B. 1:100 oder 1:1000) reduziert die effektive FPS-Rate
> erheblich, verringert jedoch auch die Erkennungsgenauigkeit. Für sicherheitskritische
> Umgebungen wird kein oder nur minimales Sampling empfohlen.

---

## 3. Schritt 2: Komponenten-Empfehlung

Für ~400.000 IPs mit einer angenommenen Flow-Rate von **100.000–300.000 FPS**
(typische gemischte Enterprise-Umgebung):

| Komponente | Empfehlung | Begründung |
|---|---|---|
| **SNA Manager** | 1× (physisch oder VM) | Zentrale Steuerung, bis zu 25 Flow Collectors |
| **Flow Collector** | 2–4× FC4300 oder FC5210 (M6) | Je FC ~100.000 FPS, Redundanz durch mehrere Instanzen |
| **Data Store** | 1× Single Node DN6300 (M6) | 500K FPS, 2M Hosts, Analytics-fähig |
| **UDP Director** | 1× (optional) | Flow-Bündelung und -Verteilung auf mehrere Collectors |
| **Flow Sensor** | Nach Bedarf | Für Segmente ohne native NetFlow-Unterstützung |
| **Cisco Telemetry Broker** | Optional | Falls AWS/Azure-Workloads integriert werden sollen |

---

## 4. Schritt 3: Skalierungspfad

```
Einstieg (< 200K FPS, ~400K Hosts):
  1× Manager
  2× Flow Collector (FC4300 oder FC5210 M6)
  1× Single Node Data Store (DN6300 M6)

Wachstum (200K–500K FPS):
  1× Manager
  4× Flow Collector
  1× Single Node Data Store (DN6300 M6)

Großes Wachstum / Rechenzentrum (> 500K FPS):
  1× Manager (HA optional: Primary + Secondary)
  4–8× Flow Collector
  3× Data Nodes → Three Node Data Store (DN6300 M6)

Enterprise / Service Provider (> 1M FPS):
  1× Manager (HA)
  bis zu 25× Flow Collector
  bis zu 12× DS6200 = 36 Data Nodes
  1× UDP Director zur Flow-Verteilung
```

---

## 5. Performance-Referenz (offizieller Cisco Design Guide, März 2025)

### Analytics deaktiviert (reine Flow-Erfassung + Verhaltensanalyse)

| Deployment | Flows/s | Unique Hosts | Report-Zeit Avg/Max |
|---|---|---|---|
| Alte Distributed-Architektur (FC5210) | 300.000 | 33 Mio. | < 60 Min. / Stunden |
| Single Node DN6300 (M6) | 500.000 | 33 Mio. | < 7 Min. / < 30 Min. |
| Single Node DN6300 (M6) | 1.000.000 | 33 Mio. | < 15 Min. / < 65 Min. |
| Three Node DN6300 (M6) | 700.000 | 33 Mio. | < 2 Min. / < 20 Min. |
| Three Node DN6300 (M6) | 2.000.000 | 33 Mio. | < 5 Min. / < 45 Min. |
| Three Node DN6300 (M6) | 3.000.000 | 33 Mio. | < 10 Min. / < 100 Min. |

### Analytics aktiviert (ML-Cloud-Integration + On-Premises-Erkennung)

| Deployment | Max. Flows/s | Max. Unique Hosts |
|---|---|---|
| Single Node DS6200 (M5) + 1× FC | 250.000 | 1,3 Mio. |
| **Single Node DN6300 (M6) + 1× FC** | **500.000** | **2,0 Mio.** |
| Three Node DS6200 (M5) + 4× FC | 600.000 | 1,3 Mio. |
| Three Node DN6300 (M6) + 4× FC | 1.000.000 | 1,0 Mio. |
| Three Node DN6300 (M6) + 4× FC | 1.500.000 | 500.000 |

> **Einordnung für 400.000 IPs:** Der Single Node DN6300 (M6) mit Analytics
> unterstützt bis zu 2,0 Millionen Unique Hosts – 400.000 IPs werden damit
> mit erheblichem Wachstumspuffer abgedeckt, solange die FPS-Rate unter
> 500.000 bleibt.

---

## 6. Retention-Planung für 400.000 IPs

Angenommene Flow-Rate: 250.000 FPS (typischer Campus-Mix):

| Gewünschte Retention | Benötigte Data Store Nodes |
|---|---|
| 90 Tage | 1× DS6200 (Single Node) |
| 180 Tage | 1× DS6200 (Single Node) |
| 360 Tage | 2× DS6200 |
| 1–2 Jahre | 3–4× DS6200 |

---

## 7. Lizenzierung

Die Flow Rate License ist für die Erfassung, Verwaltung und Analyse von
Flow-Telemetrie erforderlich. Sie definiert das maximale FPS-Volumen des Deployments.

Für 400.000 IPs benötigen Sie:

| Lizenz | Beschreibung |
|---|---|
| **Flow Rate License** | Nach FPS-Volumen gestaffelt (100K / 250K / 500K FPS) |
| **Threat Feed License** | Pro Flow Collector – für globale Bedrohungsintelligenz |
| **Analytics License** | Für ML-basierte Erkennung (optional, aber empfohlen) |

---

## 8. Architektur-Diagramm für ~400.000 IPs

```
Netzwerkgeräte (NetFlow/IPFIX)
Switches, Router, Firewalls, WLAN
         │
   UDP Director (optional)        ←── Flow-Bündelung
         │
   ┌─────┴─────┐
   │           │
  FC1         FC2                 ←── 2–4× Flow Collector FC4300/FC5210
   │           │
   └─────┬─────┘
         │
   Single Node Data Store         ←── DN6300 (M6): 500K FPS, 2M Hosts
   (DN6300 M6)
         │
   SNA Manager (SMC)              ←── Zentrale UI, Alerting, Reports
         │
   ┌─────┴──────────────┐
   │                    │
 SIEM                Cisco XDR
(Splunk/Elastic)    (Extended Detection)
```

---

## 9. Wichtige Planungshinweise

### Rechenzentrum vs. Campus trennen

SNA unterstützt mehrere Domains innerhalb eines Deployments. Separate Domains
für Rechenzentrum und Campus werden empfohlen, da der DC überproportional viele
Flows erzeugt und eigene Analyse-Schwellenwerte benötigt.

### High Availability

Der Manager kann in einer HA-Konfiguration betrieben werden (Primary + Secondary).
Alerts und Observations werden dabei nicht zwischen Primary und Secondary
synchronisiert – bei Failover starten Jobs neu.

### Flow Sampling

Flow Sampling (1:100 oder 1:1000) reduziert die FPS-Rate und die Hardware-
Anforderungen erheblich. Für sicherheitskritisches Monitoring (Erkennung von
Lateral Movement, Exfiltration) wird jedoch kein oder nur minimales Sampling
empfohlen, da feine Muster sonst nicht erkannt werden.

### Cloud-Integration

Falls AWS- oder Azure-Workloads überwacht werden sollen, ist der **Cisco Telemetry
Broker** als zusätzliche Komponente nötig. Dieser zieht VPC/VNet Flow Logs aus
S3-Buckets bzw. Azure Blob Storage, transformiert sie zu IPFIX und leitet sie
an den Flow Collector weiter.

### Wachstumspuffer einplanen

Bei 400.000 IPs empfiehlt sich ein Puffer von 50–100 % für organisches Wachstum,
neue Cloud-Workloads und Telemetrie-Erweiterungen (Remote Worker NVM, Firewall Logs).

---

## 10. Zusammenfassung: Empfohlenes Deployment für 400.000 IPs

| Szenario | Komponenten | Kosten-Effizienz |
|---|---|---|
| **Einsteiger (< 200K FPS)** | 1× Manager + 2× FC + 1× Single Node DN6300 | Optimal |
| **Standard (200K–400K FPS)** | 1× Manager + 4× FC + 1× Single Node DN6300 | Empfohlen |
| **Mit DC-Traffic (> 400K FPS)** | 1× Manager + 4× FC + 3× Nodes DN6300 | Zukunftssicher |
| **Mit HA + Cloud** | 2× Manager + 4× FC + 3× Nodes + Telemetry Broker | Enterprise-Grade |

---

## Quellen

- Cisco SNA Data Store Design Guide (März 2025):
  https://www.cisco.com/c/dam/en/us/products/collateral/security/stealthwatch/stealthwatch-data-store-guide.pdf
- Cisco SNA Datasheet:
  https://www.cisco.com/c/en/us/products/collateral/security/stealthwatch/datasheet-c78-739398.html
- Cisco Blog – SNA Data Store Upgrades:
  https://blogs.cisco.com/security/secure-network-analytics-is-upgraded-with-new-data-store-deployment
- Cisco Blog – Introducing the SNA Data Store:
  https://blogs.cisco.com/security/introducing-the-cisco-secure-network-analytics-data-store
