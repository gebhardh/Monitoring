# Cisco Secure Network Analytics (SNA) – Monitoring-Übersicht

Cisco Secure Network Analytics (früher Stealthwatch) bietet unternehmensweite
Netzwerksichtbarkeit zur Erkennung und Reaktion auf Bedrohungen in Echtzeit.
Die Lösung analysiert kontinuierlich Netzwerkaktivitäten, erstellt eine Basislinie
normalen Netzwerkverhaltens und nutzt diese zusammen mit nicht-signaturbasierten,
fortgeschrittenen Analysen – einschließlich Verhaltensmodellierung und
Machine-Learning-Algorithmen sowie globaler Bedrohungsintelligenz – um Anomalien
zu erkennen.

---

## 1. NetFlow / IPFIX Analyse (Kern-Funktion)

Der Flow Collector sammelt und analysiert Enterprise-Telemetrie wie NetFlow, IPFIX
und andere Flow-Daten von Routern, Switches, Firewalls, Endpunkten und anderen
Netzwerkgeräten. Er kann auch Telemetrie von Proxy-Datenquellen sammeln.

| Monitoring-Aspekt | Beschreibung |
|---|---|
| **Traffic-Volumen** | Bytes/Pakete pro Flow, pro Host, pro Subnetz |
| **Kommunikationsbeziehungen** | Wer spricht mit wem (Host-zu-Host, intern/extern) |
| **Top Talker** | Hosts mit höchstem Datenaufkommen |
| **Protokollverteilung** | TCP/UDP/ICMP-Anteile im Netzwerk |
| **Port-Nutzung** | Verwendete Ports und Dienste je Host |
| **Geo-IP** | Kommunikation mit bestimmten Ländern/Regionen |

---

## 2. Encrypted Traffic Analytics (ETA) – Malware in verschlüsseltem Traffic

ETA ermöglicht die Analyse verschlüsselten Traffics, ohne ihn entschlüsseln zu
müssen. Dazu werden Enhanced Telemetry-Daten von aktuellen Cisco-Netzwerkgeräten
genutzt und eine Kombination aus erweiterten Analysen angewandt, um Malware zu
erkennen und Krypto-Compliance sicherzustellen. SNA ist mit Cisco Cognitive
Analytics gekoppelt – einem Cloud-basierten Service mit Machine Learning auf
globaler Ebene.

ETA extrahiert zwei wesentliche Datenelemente:
- **IDP (Initial Data Packet):** Erste Bytes einer Verbindung für Protokollanalyse
- **SPLT (Sequence of Packet Length and Time):** Paketlängen- und Zeitmuster
  innerhalb eines Flows – unabhängig vom Protokoll und damit auch für
  verschlüsselten Traffic anwendbar

| Monitoring-Aspekt | Beschreibung |
|---|---|
| **Malware in TLS-Traffic** | Erkennung ohne Entschlüsselung |
| **TLS-Zertifikat-Analyse** | Verdächtige oder abgelaufene Zertifikate |
| **Crypto-Compliance** | Verwendung schwacher Cipher-Suites |
| **C&C-Kommunikation** | Beaconing-Muster in verschlüsselten Sessions |
| **Datenexfiltration** | Ungewöhnlich hohe Upload-Volumen in Encrypted Flows |

---

## 3. Verhaltensbasierte Anomalie-Erkennung

SNA erkennt kontinuierlich Advanced Threats, die bestehende Sicherheitskontrollen
umgangen haben oder aus dem Inneren des Netzwerks stammen. Es fokussiert auf
kritische Vorfälle statt auf Rauschen, mit kontextuellen, hochpräzisen Alarmen,
priorisiert nach Bedrohungsschweregrad.

| Bedrohungstyp | Erkennungsmethode |
|---|---|
| **Command & Control (C&C)** | Beaconing-Muster, regelmäßige externe Verbindungen |
| **Ransomware** | Massenhaftes Verschlüsseln, SMB-Anomalien |
| **DDoS** | Plötzlicher Traffic-Spike auf bestimmte Hosts |
| **Cryptomining** | Verbindungen zu Mining-Pools, hohe CPU-Aktivität |
| **Insider Threats** | Ungewöhnlicher Datenzugriff/-transfer interner Hosts |
| **Laterale Bewegung** | Ost-West-Traffic zwischen isolierten Hosts |
| **Port-Scanning** | Verbindungsversuche auf viele Ports eines Hosts |
| **Data Exfiltration** | Große Datenmengen an externe, unbekannte IPs |

---

## 4. Netzwerk-Performance Monitoring

Neben Security überwacht SNA auch Performance-Aspekte:

| Monitoring-Aspekt | Beschreibung |
|---|---|
| **Bandbreitennutzung** | Top-Anwendungen und -Hosts nach Volumen |
| **Verbindungsdauer** | Lange Sessions als Anomalie-Indikator |
| **Round-Trip-Time** | Latenz-Indikatoren aus Flow-Daten |
| **Retransmissions** | TCP-Retransmit-Rate als Performance-Signal |
| **Applikationserkennung** | Zuordnung von Flows zu Applikationen (NBAR-basiert) |

---

## 5. Identity-Context (Integration mit Cisco ISE)

Die Stealthwatch Management Console (SMC) aggregiert, organisiert und präsentiert
Analysen von bis zu 25 Flow Collectoren, Cisco ISE und anderen Quellen. Durch
die ISE-Integration werden Flow-Daten mit Nutzeridentitäten angereichert:

| Monitoring-Aspekt | Beschreibung |
|---|---|
| **User-zu-Flow-Zuordnung** | Wer hat welchen Traffic erzeugt |
| **Geräte-Klassifikation** | Typ des Endgeräts (PC, IoT, Mobil) |
| **Compliance-Status** | Ist das Gerät posture-compliant |
| **SGT (Security Group Tags)** | TrustSec-basierte Zugriffsgruppierung |

---

## 6. Cloud Monitoring (Secure Cloud Analytics)

Cisco Stealthwatch Cloud Public Cloud Monitoring bietet Sichtbarkeit und
Bedrohungserkennung in AWS-, GCP- und Microsoft Azure-Cloud-Infrastrukturen
als SaaS-Lösung ohne Software-Agenten – basierend auf nativen Telemetriequellen
wie VPC Flow Logs. Stealthwatch Cloud modelliert den gesamten IP-Traffic innerhalb
von VPCs, zwischen VPCs oder zu externen IP-Adressen und ist zusätzlich mit AWS
CloudTrail, Amazon CloudWatch, AWS Config, Inspector, IAM und Lambda integriert.

---

## 7. Forensik und Compliance

SNA kann Telemetrie skaliert speichern und bietet Netzwerk-Audit-Trails für
forensische Untersuchungen vergangener Ereignisse und für Compliance-Monitoring.

| Monitoring-Aspekt | Beschreibung |
|---|---|
| **Flow-Archivierung** | Langzeitspeicherung für Forensik |
| **Incident Timeline** | Rekonstruktion von Angriffspfaden |
| **Compliance Reports** | PCI-DSS, HIPAA, GDPR-Nachweise |
| **Audit Trails** | Wer hat wann auf was zugegriffen |

---

## 8. Architektur und Datenquellen

```
Netzwerkgeräte (NetFlow/IPFIX/sFlow)
  Switches, Router, Firewalls, WLAN-Controller
           │
    Flow Sensor (optional)     ←── Tap/Mirror-Port
           │
    UDP Director (optional)    ←── Bündelung/Verteilung
           │
    Flow Collector (bis 25x)
           │
    Data Store (optional)      ←── Zentraler Langzeitspeicher
           │
    SNA Manager (SMC)
           │
    ┌──────┴──────────────┐
    │                     │
  SIEM               Cisco XDR
 (Splunk/            (Extended
  Elastic)            Detection)
```

---

## 9. Speicherung von Flow-Daten in Cisco SNA

### 9.1 Zwei Speichermodelle

SNA bietet zwei grundlegende Deployment-Modelle für die Flow-Datenspeicherung:

#### Modell A: Distributed (Flow Collector-lokal)

Im klassischen Modell speichert jeder Flow Collector die von ihm verarbeiteten
Flow-Daten lokal in seiner eigenen Datenbank. Die Telemetrie ist damit auf
mehrere Flow Collectors verteilt.

- Geeignet für: Kleinere und mittlere Deployments
- Nachteil: Begrenzte Speicherkapazität je Collector, keine zentrale Abfrage
- Abfragen müssen je Flow Collector separat gestellt werden

#### Modell B: Central Data Store (empfohlen für große Umgebungen)

Der Data Store bietet eine Lösung für Umgebungen, die hohe Daten-Ingest-Kapazitäten oder lange Retention-Zeiten benötigen, die die Kapazität eines oder mehrerer Flow Collectors übersteigen. Der Data Store-Cluster wird zwischen dem SNA Manager und den Flow Collectors eingefügt. Ein oder mehrere Flow Collectors nehmen Flow-Daten entgegen, deduplizieren sie, führen Analysen durch und senden die Daten dann direkt an den Data Store. Diese Flow-Daten werden gleichmäßig über die Data Nodes im Data Store verteilt.

---

### 9.2 Technische Details des Data Store

Der Data Store ermöglicht es, Ingest- und Speicherfunktionen unabhängig voneinander zu skalieren. Data Stores können kombiniert werden, um einen einzelnen Cluster zu bilden, der in der Lage ist, über 3 Millionen Flows pro Sekunde zu überwachen. Die Speicherskalierbarkeit ermöglicht es, durch Hinzufügen weiterer Datenbank-Cluster zu wachsen. Die Langzeit-Datenretention ermöglicht eine Flow-Aufbewahrung von bis zu 1–2 Jahren, ohne weitere Flow Collectors hinzufügen zu müssen.

| Eigenschaft | Wert |
|---|---|
| **Mindest-Konfiguration** | 1 Data Store (DS6200) = 3 physische Data Nodes |
| **Maximale Konfiguration** | 12× DS6200 = bis zu 36 Data Nodes |
| **Max. Ingest-Rate** | > 3 Millionen Flows/Sekunde (kombiniert) |
| **Retention (Standard)** | 90 / 180 / 360 Tage (je nach Flows/s und Nodes) |
| **Retention (maximal)** | 1–2 Jahre |
| **Skalierung** | Horizontal durch Hinzufügen von Data Nodes |
| **Redundanz** | Daten werden gleichmäßig über alle Nodes verteilt |

---

### 9.3 Retention-Planung: Flows/s vs. benötigte Nodes

Die folgende Tabelle zeigt die Anzahl benötigter DS6200-Appliances je nach Flow-Rate und gewünschter Retention-Zeit für eine typische Campus-Umgebung:

| Flows/s | 90 Tage | 180 Tage | 360 Tage |
|---|---|---|---|
| 250.000 | 1× DS6200 | 1× DS6200 | 2× DS6200 |
| 500.000 | 1× DS6200 | 2× DS6200 | 4× DS6200 |
| 1.000.000 | 2× DS6200 | 4× DS6200 | 8× DS6200 |

> **Hinweis:** Service Provider-Umgebungen mit hohen Anteilen gesampelter Flows und
> vielen unique Hosts benötigen typischerweise mehr Appliances als Campus-Umgebungen
> mit vergleichbarer Flow-Rate.

---

### 9.4 Datenverarbeitung im Flow Collector vor der Speicherung

Bevor Flow-Daten in den Data Store geschrieben werden, verarbeitet der Flow
Collector sie in mehreren Schritten:

```
Rohe NetFlow/IPFIX-Records
        │
   1. Deduplication       ← Mehrfach-exportierte Flows zusammenführen
        │
   2. Normalisierung      ← Verschiedene Flow-Formate vereinheitlichen
        │
   3. Analyse / Scoring   ← Verhaltensbasierte Auswertung, ETA
        │
   4. Alarmierung         ← Security Events an Manager senden
        │
   5. Weiterleitung       ← Verarbeitete Flows → Data Store
```

---

### 9.5 Vergleich: Distributed vs. Central Data Store

| Eigenschaft | Distributed (Flow Collector lokal) | Central Data Store |
|---|---|---|
| Speicherort | Je Flow Collector lokal | Zentraler Data Store Cluster |
| Skalierung | Durch mehr Flow Collectors | Durch mehr Data Nodes |
| Max. Retention | Begrenzt durch FC-Disk | 1–2 Jahre |
| Abfragen | Nur je Collector | Zentral über alle Flows |
| Redundanz | Gering | Hoch (Datenverteilung) |
| Empfohlen für | Kleine Deployments | Große Unternehmen, Forensik |

---

## 10. Was SNA NICHT abdeckt (Abgrenzung)

| Bereich | Abdeckung durch SNA | Ergänzung nötig durch |
|---|---|---|
| Endpoint-Schutz | ❌ Agentlos, kein EDR | Cisco Secure Endpoint |
| Web-Proxy / URL-Filterung | ❌ | Cisco Umbrella |
| E-Mail-Sicherheit | ❌ | Cisco Secure Email |
| ZTNA / Zero Trust | ⚠️ Teilweise (ISE-Integration) | Cisco Duo / ISE |
| Vulnerability Management | ❌ | Cisco Vulnerability Mgmt |
| Firewall-Regelwerk | ❌ | Cisco FMC / Firepower |

---

## Quellen

- Cisco SNA Datasheet:
  https://www.cisco.com/c/en/us/products/collateral/security/stealthwatch/datasheet-c78-739398.html
- Cisco SNA At-a-Glance:
  https://www.cisco.com/c/en/us/products/collateral/security/stealthwatch/secure-network-analytics-aag.html
- Cisco SNA Data Store Design Guide:
  https://www.cisco.com/c/dam/en/us/products/collateral/security/stealthwatch/stealthwatch-data-store-guide.pdf
- Cisco Blog – Introducing the SNA Data Store:
  https://blogs.cisco.com/security/introducing-the-cisco-secure-network-analytics-data-store
- Cisco ETA Design Guide:
  https://www.cisco.com/c/dam/en/us/td/docs/solutions/CVD/Campus/eta-design-guide-2019oct.pdf
- Study CCNP – Cisco SNA Overview:
  https://study-ccnp.com/cisco-secure-network-analytics-stealthwatch-overview/
