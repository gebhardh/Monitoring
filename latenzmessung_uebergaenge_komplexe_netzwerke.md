# Übergänge mit hoher Latenz in komplexen Netzwerken ermitteln

Das Grundproblem: In einem komplexen Netzwerk mit Dutzenden Hops, Tunneln,
VXLANs und Firewalls ist es schwierig, den **einen Übergang** zu identifizieren,
der Latenz verursacht. Es gibt dafür mehrere Methoden mit unterschiedlicher
Tiefe und Genauigkeit.

---

## Methode 1: Traceroute / MTR – klassischer Einstieg

MTR (My TraceRoute) kombiniert Traceroute und Ping zu einem kontinuierlichen
Echtzeit-Monitor, der Paketverlust und Latenzstatistiken für jeden Hop simultan
anzeigt – damit deutlich leistungsfähiger für intermittierende Probleme als
klassisches Traceroute.

### Vier typische MTR-Muster

| Muster | Bedeutung |
|---|---|
| Verlust an Zwischen-Hop, kein Verlust am Ziel | Router deprioritisiert ICMP – kein echtes Problem |
| Verlust ab Hop X an allen nachfolgenden Hops | Echter Paketverlust ab diesem Punkt |
| Hohe Standardabweichung an einem Hop | Congestion oder Jitter auf diesem Link |
| Hohe Worst-Case-Latenz bei normalem Durchschnitt | Bufferbloat oder gelegentliche Burst-Verluste |

### Beispiel MTR-Ausgabe

```bash
# Kontinuierliches MTR (100 Pakete, TCP-Modus für Firewall-Bypass)
mtr -T -P 443 -c 100 --report <Ziel-IP>

# Ausgabe:
# HOST          Loss%  Snt  Last   Avg  Best  Wrst StDev
# Hop 1: GW     0.0%  100   1.1   1.2   1.0   2.0   0.2
# Hop 3: ISP    2.0%  100  12.5  13.8  11.8  82.1   6.4  ← Kandidat
# Hop 5: Ziel   0.0%  100  12.8  13.0  12.1  14.2   0.5
```

### Einschränkungen

Herkömmliches Traceroute liefert keine präzise Latenz zwischen den Hops. Router
priorisieren den Hardware-Forwarding-Pfad, antworten aber auf TTL-Expired-Nachrichten
über Software – was zu künstlich erhöhten Latenzen an einzelnen Hops führt, die
keine reale Verbindungslatenz widerspiegeln.

---

## Methode 2: In-Band Network Telemetry (INT) – modernste Methode

In-Band Network Telemetry (INT) erlaubt es, spezielle Header in Datenpakete
einzubetten, die jeden Netzwerkknoten auf dem Forwarding-Pfad anweisen, spezifische
Daten zu erfassen – darunter Ankunfts-/Abfahrts-Zeitstempel. Die Hop-Latenz
(einschließlich Verarbeitungsdelay und Ausbreitungsdelay) kann durch Differenz
der Zeitstempel zweier benachbarter Geräte bei synchronisierter Netzwerkuhr
präzise ermittelt werden.

### Vom Transit-Gerät erfasste Daten

Jedes Transit-Gerät fügt dem INT-Header folgende Daten hinzu:
- Switch-ID
- Ankunftszeit (Ingress Timestamp)
- Queue-Delay
- Gematchte Weiterleitungsregeln

Diese Daten werden am Egress-Node gesammelt und an einen Telemetrie-Collector
exportiert, ohne den eigentlichen Datenverkehr zu beeinflussen.

### Beispiel: Hop-Latenz-Lokalisierung via INT

```
Paket → Switch A (T1=10µs) → Switch B (T2=45µs) → Switch C (T3=48µs)
                    ↑
              Hop-Latenz A→B = 35µs  ← Auffällig!
              Hop-Latenz B→C =  3µs  ← Normal
```

### Voraussetzungen

- P4-programmierbare ASICs (Intel Tofino) oder Nexus Cloud-Scale ASICs (EX/FX/GX)
- PTP-Zeitsynchronisation (Mikrosekunden-Genauigkeit)
- INT-fähige Telemetrie-Collector-Software

---

## Methode 3: Segment Routing + INT (SR-INT)

SR-INT kombiniert Segment Routing mit In-Band Network Telemetry. Der SDN-Controller
berechnet den Forwarding-Pfad eines Flows, teilt ihn in Segmente auf und weist
kritische Switches als INT-Messpunkte zu. Pro Hop werden Device-ID, Output-Port,
Hop-Latenz und Bandbreite erfasst, wodurch Netzwerkausnahmen wie Congestion oder
Switch-Fehlkonfigurationen präzise lokalisiert werden können.

### Drei Rollen in einem INT-Deployment

```
INT Source    → Bettet INT-Header in Pakete ein
INT Transit   → Fügt eigene Telemetriedaten hinzu (SwitchID, Timestamp, Queue)
INT Sink      → Extrahiert INT-Daten, exportiert an Collector, entfernt INT-Header
```

---

## Methode 4: Cisco ThousandEyes – agenten-basiert

ThousandEyes platziert Software-Agenten an Endpunkten und misst kontinuierlich
Pfade und Latenzen – besonders stark bei WAN-, Internet- und Cloud-Übergängen.

| Feature | Beschreibung |
|---|---|
| **Path Visualization** | Grafische Darstellung jedes Hops mit Latenz und Paketverlust |
| **Synthetic Tests** | Simulierte HTTP/TCP/DNS-Tests von Agents zu Zielen |
| **Real User Monitoring** | Echte Nutzererfahrung als Latenz-Quelle |
| **BGP-Routing-Überwachung** | Erkennt Route-Changes, die Latenzen verursachen |
| **Agenten-Typen** | Enterprise Agent (On-Premises), Cloud Agent (global), Endpoint Agent (Laptop) |

Besonders geeignet für: WAN-Übergänge, Internet-Pfade, SaaS-Dienste,
Multi-Cloud-Verbindungen.

---

## Methode 5: TWAMP / IP SLA – aktive Latenz-Messung

TWAMP (Two-Way Active Measurement Protocol, RFC 5357) ist ein Standard für
aktive Latenzmessung zwischen zwei Netzwerkgeräten.

### Linux (owamp)

```bash
# One-Way Delay Measurement (1000 Testpakete)
owping -c 1000 <Ziel-IP>
```

### Cisco IOS/NX-OS (IP SLA)

```
ip sla 1
 udp-jitter <Ziel-IP> 5000 num-packets 1000
 frequency 30
ip sla schedule 1 life forever start-time now

# Ergebnisse abrufen
show ip sla statistics 1
```

Messwerte: One-Way-Delay, Two-Way-Delay, Jitter, Paketverlust.
Genauigkeit abhängig von PTP/NTP-Qualität. Einsatz: Zwischen zwei bekannten
Endpunkten, z. B. PE-Routern oder Data-Center-Gateways.

---

## Methode 6: BFD (Bidirectional Forwarding Detection)

BFD ist primär für schnelle Ausfallerkennnung ausgelegt, liefert aber indirekt
Latenz-Indikatoren durch Timeout-Verhalten.

```
# Cisco NX-OS – BFD-Intervall 50ms
bfd interval 50 min_rx 50 multiplier 3
```

Erkennt Linkausfälle in < 150ms und zeigt durch häufige BFD-Timeouts
Latenzdegradation auf bestimmten Links an.

---

## Methode 7: Streaming Telemetry + Zeitreihenanalyse

Bei großen Fabrics die systematischste Dauermethode:

### Architektur

```
gNMI Streaming (alle 10–30s)
  Interface-Counters + Queue-Statistiken
        │
  Telegraf (inputs.gnmi)
        │
  Prometheus / InfluxDB (Zeitreihendatenbank)
        │
  Grafana (Dashboards + Anomalie-Alerts)
        │
  Alert: Interface X hat hohe Queue-Drops → Latenz-Kandidat
```

### Telegraf-Konfiguration

```toml
[[inputs.gnmi]]
  addresses = ["switch1:57777"]

  [[inputs.gnmi.subscription]]
    name = "interface_counters"
    path = "/interfaces/interface/state/counters"
    subscription_mode = "sample"
    sample_interval = "10s"

  [[inputs.gnmi.subscription]]
    name = "queue_stats"
    path = "/qos/interfaces/interface/output/queues/queue/state"
    subscription_mode = "sample"
    sample_interval = "10s"
```

Korrelation: Hohe Egress-Drop-Rate auf Interface X → wahrscheinlicher
Latenz-Verursacher auf diesem Segment.

---

## Methode 8: Passive Analyse mit Zeek + Arkime

Für forensische Analyse vergangener Latenzereignisse ohne aktive Messpakete.

### Zeek – TCP-RTT aus Paketsequenz

```zeek
# Zeek conn.log enthält automatisch:
# - orig_bytes, resp_bytes, conn_state
# - history (SYN/FIN/RST-Sequenz)
# - duration (Verbindungsdauer)

# Für detaillierte RTT-Analyse:
@load misc/stats
# → stats.log enthält TCP RTT-Werte
```

### Arkime – Full Packet Capture mit Zeitstempeln

```bash
# Arkime-Suche nach langen TCP-Sessions
# Elasticsearch-Query: duration > 500ms AND protocol == tcp
# → PCAP herunterladen und in Wireshark analysieren
# → TCP Delta-Zeiten zeigen exakte Hop-Latenzen
```

---

## Entscheidungsmatrix: Welche Methode für welches Problem?

| Szenario | Beste Methode | Genauigkeit | Aufwand |
|---|---|---|---|
| Schnelle Erstdiagnose | MTR | Mittel | Gering |
| Internet / WAN-Übergänge | ThousandEyes | Hoch | Mittel |
| Intra-Fabric Hop-Latenz (ACI/NX-OS) | INT / NDI Flow Telemetry | Sehr hoch | Hoch |
| Latenz zwischen zwei Rechenzentren | TWAMP / IP SLA | Hoch | Mittel |
| Tunnel-Latenz (VXLAN, IPSec, GRE) | INT oder ThousandEyes | Hoch | Mittel-hoch |
| Historische Forensik | Zeek + Arkime | Mittel | Mittel |
| Dauerhaftes breites Monitoring | Streaming Telemetry + Grafana | Mittel-hoch | Mittel |
| P4-Fabric (Barefoot/Intel Tofino) | INT nativ | Sehr hoch | Hoch |
| BGP-Routing-bedingte Latenzen | ThousandEyes BGP Monitor | Hoch | Gering |
| Queue-bedingte Latenzen | Streaming Telemetry + QoS-Metriken | Hoch | Mittel |

---

## Typischer Workflow zur Latenz-Lokalisierung

```
Schritt 1: MTR / Traceroute
   → Verdächtigen Hop-Bereich identifizieren (z. B. Hop 4–6)
   → Muster erkennen: Wo springt Avg oder Wrst an?
         │
Schritt 2: Streaming Telemetry prüfen
   → Interface-Utilization und Queue-Drops am verdächtigen Segment
   → Liegt Auslastung > 70–80 %?
         │
Schritt 3: INT / Atomic Counters / TWAMP
   → Exakte Hop-Latenz zwischen konkreten Geräten messen
   → Microsekunden-genaue Lokalisierung des Engpasses
         │
Schritt 4: Root Cause identifizieren
   → Congestion?          → Mehr Bandbreite / QoS anpassen
   → Misconfiguration?    → Routing / MTU / Duplex prüfen
   → Hardware-Defekt?     → Interface-Fehler, CRC-Errors prüfen
   → Routing-Problem?     → BGP-Path-Change, suboptimales Routing
         │
Schritt 5: Dauerhaftes Monitoring einrichten
   → Alert-Threshold in Grafana / NDI / ThousandEyes setzen
   → Baseline definieren und Abweichungen automatisch erkennen
```

---

## Häufige Latenz-Ursachen und Erkennungsmerkmale

| Ursache | Erkennungsmerkmal | Methode |
|---|---|---|
| **Link-Überlastung** | Hohe Interface-Utilization (> 80 %) | Streaming Telemetry |
| **Queue-Overflow** | Hohe Egress-Drop-Rate | ACI Atomic Counter / INT |
| **Bufferbloat** | Hohe Wrst-Latenz bei normalem Avg | MTR (hohes StDev) |
| **Suboptimales Routing** | Plötzlich mehr Hops als zuvor | ThousandEyes Path View |
| **MTU-Mismatch** | Fragmentierung, hohe Retransmit-Rate | Zeek / Wireshark |
| **Duplex-Mismatch** | Hohe CRC-Fehler auf einem Interface | SNMP / gNMI |
| **Hardware-Defekt** | Physikalische Fehler, CRC, Runts | Interface-Error-Counters |
| **Firewall-Inspektion** | Latenz genau an Firewall-Hop | MTR + Topology-Wissen |
| **DNS-Latenz** | Langsame DNS-Auflösung als Schein-Netzwerklatenz | ThousandEyes DNS-Test |
| **BGP-Pfadwechsel** | Plötzliche Latenzzunahme nach Route-Change | ThousandEyes BGP Monitor |

---

## Quellen

- MTR – Continuous Traceroute Monitoring (2026):
  https://oneuptime.com/blog/post/2026-03-20-mtr-continuous-traceroute-monitoring/view
- Intel INT White Paper – In-Band Network Telemetry detects Network Performance Issues:
  https://builders.intel.com/docs/networkbuilders/in-band-network-telemetry-detects-network-performance-issues.pdf
- SR-INT: On Orchestration of Segment Routing and In-Band Network Telemetry:
  https://www.researchgate.net/publication/369031352_On_Orchestration_of_Segment_Routing_and_In-band_Network_Telemetry
- Site24x7 – Troubleshooting Latency Spikes with MTR and ISP Analysis:
  https://www.site24x7.com/solutions/troubleshooting-latency-spikes-using-traceroute-mtr-isp-analysis.html
- Cisco ThousandEyes – Network Intelligence:
  https://www.thousandeyes.com
- Cisco IP SLA Configuration Guide NX-OS:
  https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/ip_sla/
- Zeek Network Security Monitor:
  https://zeek.org
- Arkime Full Packet Capture:
  https://arkime.com
