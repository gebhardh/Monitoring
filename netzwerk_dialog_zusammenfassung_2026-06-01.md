# Dialogzusammenfassung – Netzwerk-Monitoring & IT-Management
**Datum:** 01. Juni 2026

---

## Inhaltsverzeichnis

1. [Cisco Nexus – Metriken und Monitoring](#1-cisco-nexus--metriken-und-monitoring)
2. [Telegraf mit gRPC/gNMI](#2-telegraf-mit-grpcgnmi)
3. [Cisco ACI – Metriken, Produkte und Latenzmessung](#3-cisco-aci--metriken-produkte-und-latenzmessung)
4. [Discard-Rate vs. Drop-Rate](#4-discard-rate-vs-drop-rate)
5. [Was ist eine Line-Card?](#5-was-ist-eine-line-card)
6. [AWS und STACKIT – Netzwerktopologie-Daten](#6-aws-und-stackit--netzwerktopologie-daten)
7. [ZTNA – Metriken und Logs](#7-ztna--metriken-und-logs)
8. [Cisco SNA – Monitoring und Datenspeicherung](#8-cisco-sna--monitoring-und-datenspeicherung)
9. [Cisco SNA – Skalierung für 400.000 IPs](#9-cisco-sna--skalierung-für-400000-ips)
10. [Alternativen zu Cisco SNA](#10-alternativen-zu-cisco-sna)
11. [Latenzmessung in komplexen Netzwerken](#11-latenzmessung-in-komplexen-netzwerken)
12. [NetFlow auf Linux mit Telegraf](#12-netflow-auf-linux-mit-telegraf)
13. [NetFlow-Dashboards in Grafana/Prometheus](#13-netflow-dashboards-in-grafanaprometheus)
14. [Telegraf-Metriken an Splunk senden](#14-telegraf-metriken-an-splunk-senden)
15. [Qbilon – Produktmerkmale und CSDM-Vergleich](#15-qbilon--produktmerkmale-und-csdm-vergleich)
16. [NNMi – Datenerfassung und ServiceNow CSDM](#16-nnmi--datenerfassung-und-servicenow-csdm)

---

## 1. Cisco Nexus – Metriken und Monitoring

### 20 wichtigste Metriken

**System-Ressourcen:**
- CPU-Auslastung (`cpmCPUTotal5minRev`, OID `1.3.6.1.4.1.9.9.109.1.1.1.1.8`)
- Memory-Auslastung (`cpmCPUMemoryUsed`)
- Buffer-Auslastung, Uptime, Temperatur

**Hardware-Gesundheit:**
- Lüfter-Status (`cefcFanTrayOperStatus`)
- Netzteil-Status (CISCO-ENTITY-FRU-CONTROL-MIB)
- Modul-/Linecards-Status (ENTITY-MIB)

**Interface-Metriken:**
- Traffic In/Out (`ifInOctets` / `ifOutOctets`, IF-MIB)
- Interface-Fehler (`ifInErrors` / `ifOutErrors`)
- Interface-Status (`ifOperStatus`), Auslastung, Discards

**Routing-Protokolle:**
- BGP Session-Status (CISCO-BGP4-MIB), BGP Prefix-Anzahl
- OSPF Neighbor-Status (`ospfNbrState`)

**Layer-2 / VxLAN:**
- VLAN-Nutzung, STP-Status
- VxLAN Tunnel-Status, TCAM-Auslastung

### Abruf-Methoden

| Methode | Protokoll | Typ |
|---|---|---|
| SNMP v2c/v3 | UDP 161 | Pull |
| gNMI Telemetry | gRPC | Push |
| NX-API REST | HTTP/S | Pull |
| CLI via SSH | SSH | Pull |

---

## 2. Telegraf mit gRPC/gNMI

Telegraf unterstützt gRPC mit zwei Plugins:

### `inputs.gnmi` – Dial-in (Pull)
- Telegraf baut Verbindung zum Switch auf
- Unterstützt NX-OS ab 9.3, IOS XR ab 6.5.1, IOS XE ab 16.12
- Aktivierung: `feature grpc` auf dem Switch

### `inputs.cisco_telemetry_mdt` – Dial-out (Push)
- Switch pusht aktiv zu Telegraf
- Transport: TCP oder gRPC (TLS nur bei gRPC)
- Aktivierung: `feature telemetry` auf dem Switch

| | `inputs.gnmi` | `inputs.cisco_telemetry_mdt` |
|---|---|---|
| Modell | Dial-in | Dial-out |
| Standard | OpenConfig/IETF | Cisco-proprietär |
| Empfehlung | Zukunftssicher | Etabliert |

---

## 3. Cisco ACI – Metriken, Produkte und Latenzmessung

### 20 wichtigste ACI-Metriken (Spine & Leaf, ~5.000 Ports)

Primärer Abrufweg: **APIC REST API**

**Fabric-Gesundheit:** Fabric Health Score, Node Health Score, Fault Count, APIC Cluster-Status

**System-Ressourcen:** CPU (`procEntity`), Memory (`procMem`), APIC Disk (`eqptStorage`)

**Interface-Metriken:** Traffic RX/TX (`eqptEgrTotal5min`), Fehler (`rmonEtherStats`), Status (`ethpmPhysIf`), Discards (`eqptEgrDropPkts5min`)

**Fabric-Overlay:** VXLAN Tunnel (`tunnelIf`), EPG Health Score, Bridge Domain, Contract Faults

**Hardware:** TCAM (`eqptcapacityPolUsage5min`), Temperatur, Netzteil/Lüfter

**ISL:** Spine-Uplink-Auslastung

> **Wichtig bei 5.000 Ports:** Zertifikatsbasierte Authentifizierung (X.509) verwenden – vermeidet Rate-Limiting durch APIC NGINX.

### Cisco-Produkte in der ACI-Umgebung

| Kategorie | Produkte |
|---|---|
| Controller | APIC-L1/L2/L3 |
| Leaf-Switches | Nexus 9300-Serie |
| Spine-Switches | Nexus 9500-Serie |
| Management | Nexus Dashboard, NDO, NDI |
| Erweiterung | Multi-Pod, Multi-Site, Remote Leaf, vPod |
| L4-L7 | Firepower, ASA, F5, Citrix |
| Virtuell | ACI Virtual Edge, Cloud APIC |

### Latenzmessung in ACI

| Methode | Granularität | PTP nötig | Lizenz |
|---|---|---|---|
| **NDI Flow Telemetry** | Pro Flow (5-Tupel) | ✅ Ja | NDI-Lizenz |
| **Atomic Counters** | Pro IP-Paar | ✅ Ja | Basis-APIC |
| **ELAM** | Einzelpaket | ❌ Nein | Basis-APIC |

- PTP aktivieren: `System → System Settings → Precision Time Protocol → Enabled`
- Latenz = Egress-Timestamp − Ingress-Timestamp (Auflösung: Mikrosekunden)
- Messung: Ingress-Leaf bis Egress-Leaf (nicht End-to-End inkl. Server-Stack)

---

## 4. Discard-Rate vs. Drop-Rate

| Kriterium | Discard | Drop |
|---|---|---|
| Paket fehlerfrei? | ✅ Ja | ❌ Nein / policy-verletzt |
| Grund | Ressourcenmangel (Puffer voll) | Fehler, Regel, Policy |
| Typische Quelle | Egress-Queue, Buffer, QoS | ACL, Firewall, CRC, TTL, MTU |
| SNMP-Zähler | ifInDiscards, ifOutDiscards | ifInErrors, ifOutErrors |
| Behebung | Mehr Bandbreite, QoS, Load Balancing | Regelwerk, Hardware, Routing |

**Faustregel:**
- **Discards** = Netz ist **überlastet** → Kapazitätsproblem
- **Drops** = Netz ist **defekt oder restriktiv** → Qualitäts-/Konfigurationsproblem

**Alert-Schwellenwerte:**

| Metrik | Warnung | Kritisch |
|---|---|---|
| Discard-Rate | > 0,1 % | > 0,5 % |
| CRC-Fehler-Rate | > 0,01 % | > 0,1 % |
| Input/Output Errors | > 0 dauerhaft | Sofort untersuchen |

---

## 5. Was ist eine Line-Card?

Eine Line-Card ist eine steckbare Erweiterungsplatine in modularen Chassis-Switches.
Sie enthält physische Interfaces, NPU (ASIC für Forwarding), Puffer-Speicher und
das Fabric-Interface zur Backplane.

- **Chassis-Switches:** Nexus 9500, 7000, Catalyst 9600 – Line-Cards wechselbar
- **Fixed Switches:** Nexus 9300, Catalyst 9300 – alles fest verbaut

**Monitoring-Relevanz:**
- TCAM-Auslastung pro Line-Card
- Discard-Raten entstehen in Egress-Queues der Line-Card (Head-of-Line Blocking)
- Temperatur-Metriken pro Line-Card

---

## 6. AWS und STACKIT – Netzwerktopologie-Daten

### AWS

**Strukturdaten:** VPC, Subnetze, Routing-Tabellen, Internet/NAT Gateways, Security Groups, NACLs, VPC Endpoints, Peering, Transit Gateway, Direct Connect, VPN

**Verkehrsdaten:** VPC Flow Logs (IP-Traffic), Transit Gateway Flow Logs, DNS Query Logging

**Analyse-Tools:** Network Manager Dashboard, VPC Reachability Analyzer, Network Access Analyzer

**CloudWatch-Metriken:** NetworkIn/Out, ActiveFlowCount, TunnelState

### STACKIT

**Strukturdaten:** Virtual Networks, Subnetze, Network Areas (SNA), Public IPs, Network Interfaces, VPC Peering

**Sicherheit:** Security Groups, IPsec VPN (IKEv2)

**Observability:** Grafana + Prometheus + Loki (STACKIT Observability Stack)

### Vergleich AWS vs. STACKIT

| Funktion | AWS | STACKIT |
|---|---|---|
| Flow Logs | ✅ | ❌ |
| Reachability Analyzer | ✅ | ❌ |
| Transit Gateway | ✅ | ⚠️ Network Area |
| Observability Stack | CloudWatch | Grafana + Prometheus |
| Datenschutz/DSGVO | ⚠️ US-Anbieter | ✅ Deutsche RZ |

---

## 7. ZTNA – Metriken und Logs

ZTNA-Architektur erzeugt Metriken und Logs aus 5 Ebenen:

### Authentifizierungs-Metriken
`auth_attempts_total`, `auth_failures_total`, `mfa_challenge_rate`, `token_issuance_latency_ms`

### Device Posture Metriken
`device_posture_compliant`, `os_patch_compliance_rate`, `antivirus_enabled_rate`, `certificate_expiry_days`

### Policy Enforcement Metriken
`policy_allow_total`, `policy_deny_total`, `policy_deny_rate`, `lateral_movement_attempts`

### Session-Metriken
`active_sessions_total`, `sessions_terminated_posture`, `step_up_auth_events`

### Gateway-Metriken
`gateway_cpu_usage_pct`, `gateway_throughput_bytes_sec`, `tunnel_setup_latency_ms`

### Log-Typen

| Log-Typ | Ziel |
|---|---|
| Authentication Logs | SIEM |
| Authorization Logs | SIEM |
| Session Logs | SIEM / SOAR |
| Device Posture Logs | MDM / SIEM |
| Anomaly Logs | SIEM / SOC |

### Wichtigste Alerting-Regeln

| Alert | Priorität |
|---|---|
| Impossible Travel | Kritisch |
| Lateral Movement | Kritisch |
| Posture Drift (während Session) | Hoch |
| Brute Force (> 5 Fehler/60s) | Hoch |
| Token-Missbrauch | Kritisch |

---

## 8. Cisco SNA – Monitoring und Datenspeicherung

### Monitoring-Aspekte

1. **NetFlow/IPFIX-Analyse** – Traffic-Volumen, Top Talker, Protokollverteilung, Geo-IP
2. **Encrypted Traffic Analytics (ETA)** – Malware-Erkennung in TLS ohne Entschlüsselung (IDP + SPLT)
3. **Verhaltensbasierte Anomalie-Erkennung** – C&C, Ransomware, DDoS, Lateral Movement, Exfiltration
4. **Performance-Monitoring** – Bandbreite, Retransmissions, App-Erkennung
5. **Identity Context (ISE-Integration)** – User-zu-Flow-Zuordnung, Geräte-Klassifikation, SGT
6. **Cloud Monitoring** – AWS VPC Flow Logs, CloudTrail, Azure (Secure Cloud Analytics)
7. **Forensik & Compliance** – Flow-Archivierung, Incident Timeline, PCI-DSS/HIPAA/GDPR

### Datenspeicherung

**Zwei Modelle:**
- **Distributed:** Flow Collector lokal – für kleine Deployments
- **Central Data Store (DN6300):** empfohlen für große Umgebungen

| Data Store | Max. Flows/s (Analytics) | Max. Unique Hosts | Retention |
|---|---|---|---|
| Single Node DN6300 (M6) | 500.000 | 2,0 Mio. | bis 1–2 Jahre |
| Three Node DN6300 (M6) | 1.000.000 | 1,0 Mio. | bis 1–2 Jahre |

---

## 9. Cisco SNA – Skalierung für 400.000 IPs

### Empfohlene Architektur

| Szenario | Komponenten |
|---|---|
| Einstieg (< 200K FPS) | 1× Manager + 2× FC + 1× Single Node DN6300 |
| Standard (200K–400K FPS) | 1× Manager + 4× FC + 1× Single Node DN6300 |
| Mit DC-Traffic (> 400K FPS) | 1× Manager + 4× FC + 3× Nodes DN6300 |
| Enterprise + HA + Cloud | 2× Manager + 4× FC + 3× Nodes + Telemetry Broker |

**Wichtige Hinweise:**
- 400.000 IPs passen mit Puffer in einen Single Node DN6300 (M6)
- Kritischer Faktor ist die **Flow-Rate (FPS)**, nicht die IP-Anzahl
- Rechenzentrum-Traffic erzeugt überproportional viele Flows → separate Domain empfohlen
- Zertifikatsbasierte Authentifizierung für Cloud-Integration (Telemetry Broker)

---

## 10. Alternativen zu Cisco SNA

### Kommerzielle NDR-Lösungen

| Produkt | Ansatz | Stärke |
|---|---|---|
| **Vectra AI** | ML/KI | Hohe Alert-Präzision |
| **ExtraHop Reveal(x)** | DPI/Wire-Data | L7-Forensik |
| **Darktrace NDR** | Unsupervised ML | Keine Regelwartung |
| **Corelight** | Zeek/Suricata | Open NDR, kein Lock-in |
| **Palo Alto Cortex NDR** | XDR-integriert | PA-Ökosystem |
| **SolarWinds NTA** | NetFlow | Performance-Monitoring |

### Open-Source-Lösungen

| Produkt | Ansatz | Lizenz |
|---|---|---|
| **Zeek** | Protokoll-Logs | BSD |
| **Suricata** | IDS/IPS/NSM | GPL |
| **Arkime** | Full PCAP | Apache 2.0 |
| **ntopng + nProbe** | NetFlow + Pakete | Dual |
| **Security Onion** | Bundle (Zeek+Suricata+Arkime) | Open Source |
| **Wazuh** | SIEM + HIDS | GPL |

### Empfehlung nach Szenario

| Szenario | Empfehlung |
|---|---|
| Cisco-Infrastruktur | Cisco SNA |
| Max. KI ohne Regelwartung | Darktrace oder Vectra AI |
| Forensik & Threat Hunting | Corelight oder ExtraHop |
| Open Source, schneller Start | Security Onion |
| Open Source, NetFlow-Fokus | ntopng + nProbe |

---

## 11. Latenzmessung in komplexen Netzwerken

### Methoden im Überblick

| Methode | Szenario | Genauigkeit |
|---|---|---|
| **MTR / Traceroute** | Schnelle Erstdiagnose | Mittel |
| **INT (In-Band Telemetry)** | Intra-Fabric Hop-Latenz | Sehr hoch |
| **SR-INT** | SDN + Telemetrie kombiniert | Sehr hoch |
| **ThousandEyes** | Internet / WAN / SaaS | Hoch |
| **TWAMP / IP SLA** | Latenz zwischen zwei RZ | Hoch |
| **Streaming Telemetry** | Dauerhaftes Monitoring | Mittel-hoch |
| **Zeek + Arkime** | Historische Forensik | Mittel |

### Typischer Diagnose-Workflow

```
1. MTR → Verdächtigen Hop-Bereich identifizieren
2. Streaming Telemetry → Interface-Utilization und Queue-Drops prüfen
3. INT / Atomic Counters / TWAMP → Exakte Hop-Latenz messen
4. Root Cause identifizieren → Congestion / Misconfiguration / HW-Defekt
5. Dauerhaftes Monitoring → Alert-Threshold setzen
```

### Häufige Latenz-Ursachen

| Ursache | Erkennungsmerkmal |
|---|---|
| Link-Überlastung | Interface-Utilization > 80 % |
| Queue-Overflow | Hohe Egress-Drop-Rate |
| Bufferbloat | Hohes StDev bei MTR |
| Suboptimales Routing | Plötzlich mehr Hops |
| MTU-Mismatch | Fragmentierung, hohe Retransmit-Rate |
| Firewall-Inspektion | Latenz genau am Firewall-Hop |

---

## 12. NetFlow auf Linux mit Telegraf

### Eigenen Traffic als NetFlow exportieren

**softflowd** (empfohlen):
```bash
softflowd -i eth0 -n 127.0.0.1:2055 -v 9 -T full
```

**fprobe** (libpcap-basiert):
```bash
fprobe -i eth0 -f ip 127.0.0.1:2055
```

### Telegraf-Konfiguration

```toml
[[inputs.netflow]]
  service_address = "udp4://:2055"
  netflow_version = "auto"

[[outputs.influxdb_v2]]
  urls = ["http://localhost:8086"]
  bucket = "netflow"
```

**Port-Standards:** NetFlow = 2055, sFlow = 6343, IPFIX = 4739

---

## 13. NetFlow-Dashboards in Grafana/Prometheus

- **Prometheus** ist für NetFlow-Daten nur eingeschränkt geeignet (Cardinality-Problem durch viele Tags)
- **InfluxDB** ist das empfohlene Backend für NetFlow-Visualisierung
- Fertige Grafana-Dashboards: ID **18539** (NetFlow/IPFIX), ID **11201** (sFlow)
- Für Prometheus: Telegraf-Aggregation vorschalten (verliert Flow-Granularität)

---

## 14. Telegraf-Metriken an Splunk senden

### Drei Integrationswege

**Weg 1: Direkt via HEC (empfohlen)**
```toml
[[outputs.http]]
  url = "https://splunk:8088/services/collector"
  data_format = "splunkmetric"
  [outputs.http.headers]
    Authorization = "Splunk <token>"
  [outputs.http.splunkmetric]
    splunkmetric_hec_routing = true
```

**Weg 2: Via Datei + Universal Forwarder**
```toml
[[outputs.file]]
  files = ["/var/log/telegraf/metrics.out"]
  data_format = "splunkmetric"
```

**Weg 3: Via Kafka** (Hochlast, resilienter)

### Wichtig
- Splunk-Index als **Metriken-Index** anlegen (nicht Events-Index)
- `splunkmetric_hec_routing = true` ist Pflicht
- HEC global aktivieren: `Settings → Data Inputs → HTTP Event Collector → Global Settings → Enabled`

---

## 15. Qbilon – Produktmerkmale und CSDM-Vergleich

### Überblick
- Gegründet 2019 in Augsburg, 2023 von Paessler AG übernommen
- IT-Asset-Management mit automatischer Datenfusion aus heterogenen Quellen

### Vier Kernkomponenten
1. **Consolidation Database** – Single Source of Truth, dynamisches Datenmodell
2. **Analytics Engine** – Abhängigkeitskarten, KPI-Reports, Audit-Vorbereitung
3. **Workflow Engine** – Automatisierte Datenpipelines, CMDB-Befüllung
4. **Monitoring Component** – PRTG-Integration, automatisierte Sensor-Verwaltung

### Connectors
AWS, Azure, GCP, Microsoft Entra ID, VMware vSphere, Kubernetes, PRTG, ServiceNow, Darktrace, CSV, Webhooks, LUY

### Alleinstellungsmerkmal gegenüber ServiceNow CSDM

Qbilon ist **kein Konkurrent zur CSDM – sondern ihr Zulieferer:**

| Aufgabe | ServiceNow CSDM allein | Qbilon + CSDM |
|---|---|---|
| Datenstruktur definieren | ✅ | ✅ übernimmt CSDM-Struktur |
| Daten automatisch erfassen | ⚠️ teures Discovery | ✅ Multi-Source automatisch |
| Multi-Cloud konsolidieren | ⚠️ aufwendig | ✅ AWS + Azure + GCP + vSphere |
| CMDB aktuell halten | ⚠️ teuer | ✅ alle 30 Min., bis zu 90 % günstiger |
| OT-Assets erfassen | ❌ | ✅ IT + OT unified |

**Praxisbeispiel RheinEnergie:** 30.000+ Cloud- und 10.000+ On-Premises-Assets alle 30 Minuten automatisch erfasst und in ServiceNow gespiegelt.

---

## 16. NNMi – Datenerfassung und ServiceNow CSDM

### Datenerfassung aus verschiedenen Quellen

| Quelle | NNMi-Unterstützung |
|---|---|
| Netzwerkinfrastruktur | ✅ Sehr gut (3.500+ Geräte, Cisco ACI) |
| VMware vSphere | ✅ Gut (nativ via vSphere API) |
| AWS | ⚠️ Eingeschränkt (Deployment, keine native Discovery) |
| OpenStack | ⚠️ Eingeschränkt (nur physische Infra via SNMP) |

### Übergabe von Strukturdaten an ServiceNow CSDM

**Kein direkter OOTB-Connector vorhanden** – drei Wege:

| Weg | Datentyp | Aufwand |
|---|---|---|
| **NNMi → uCMDB → ServiceNow** | Topologie + CIs | Hoch (2 Produkte, Lizenz) |
| **NNMi → ServiceNow Event Mgmt** | Nur Events/Incidents | Gering (OOTB) |
| **NNMi REST API → Custom** | Flexibel | Sehr hoch |

**Was NNMi via uCMDB an CSDM liefern kann:**

| CSDM-Domäne | CI-Typ in ServiceNow |
|---|---|
| Manage Technical Services | `cmdb_ci_netgear` (Router, Switch, Firewall) |
| Foundation | `cmdb_ci_ip_address`, `cmdb_ci_network_adapter` |
| Design | Relationships, VLANs, L2/L3-Verbindungen |

**Einschränkungen:**
- uCMDB-Lizenz erforderlich (separates OpenText-Produkt)
- Custom Attributes aus NNMi nur eingeschränkt mappbar (ab Version 22.11)
- NNMi liefert technische CIs (Layer 1-3), keine Business-Service-Verknüpfungen
- **Empfehlung:** Qbilon als schlankere Alternative prüfen – direkter ServiceNow-Connector, günstiger, wartungsärmer

---

## Erstellte Dateien (Übersicht)

| Datei | Inhalt |
|---|---|
| `cisco_nexus_metriken_und_telegraf_grpc.md` | Nexus-Metriken + Telegraf gRPC/gNMI |
| `cisco_aci_metriken_spine_leaf_5000ports.md` | ACI-Metriken für 5.000 Ports |
| `cisco_aci_produkte_uebersicht.md` | Cisco ACI Produktökosystem |
| `cisco_aci_latenzmessung_einzelverbindungen.md` | Latenzmessung in ACI |
| `drop_rate_vs_discard_rate.md` | Unterschied Drop vs. Discard |
| `monitoring_netzwerk_gesamtdokument.md` | Monitoring-Gesamtdokument |
| `ztna_metriken_und_logs.md` | ZTNA-Metriken und Logs |
| `cisco_sna_monitoring_und_datenspeicherung.md` | Cisco SNA Monitoring + Speicherung |
| `cisco_sna_skalierung_400k_ips.md` | SNA-Skalierung für 400.000 IPs |
| `alternativen_cisco_sna_ndr.md` | 15 NDR-Alternativen zu Cisco SNA |
| `latenzmessung_uebergaenge_komplexe_netzwerke.md` | Latenzmessung Hop-by-Hop |
| `telegraf_metriken_nach_splunk.md` | Telegraf → Splunk HEC |
| `qbilon_produktmerkmale.md` | Qbilon + CSDM-Vergleich |

---

*Zusammenfassung erstellt am 01. Juni 2026*
