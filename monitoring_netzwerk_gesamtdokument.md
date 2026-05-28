# Netzwerk-Monitoring – Fragen & Antworten

Dieses Dokument fasst alle Fragen und Antworten zum Thema Netzwerk-Monitoring
aus der Beratungssession zusammen.

---

# Inhaltsverzeichnis

1. [Cisco Nexus – Die 20 wichtigsten Metriken](#1-cisco-nexus--die-20-wichtigsten-metriken)
2. [Telegraf mit gRPC/gNMI für Cisco Nexus](#2-telegraf-mit-grpcgnmi-für-cisco-nexus)
3. [Cisco ACI – Die 20 wichtigsten Metriken (Spine & Leaf, ~5.000 Ports)](#3-cisco-aci--die-20-wichtigsten-metriken-spine--leaf-5000-ports)
4. [Cisco-Produkte in einer ACI-Umgebung](#4-cisco-produkte-in-einer-aci-umgebung)
5. [Was bedeutet eine Discard-Rate von 0,9?](#5-was-bedeutet-eine-discard-rate-von-09)
6. [Was ist eine Line-Card in einem Switch?](#6-was-ist-eine-line-card-in-einem-switch)
7. [AWS – Netzwerktopologie-Daten](#7-aws--netzwerktopologie-daten)
8. [STACKIT – Netzwerktopologie-Daten](#8-stackit--netzwerktopologie-daten)

---

# 1. Cisco Nexus – Die 20 wichtigsten Metriken

**Frage:** Nenne die 20 wichtigsten Metriken für einen Cisco Nexus. Wie kann ich diese Metriken abrufen?

NX-OS bietet mehrere Mechanismen zur Datenerhebung: SNMP, CLI, Syslog und moderne
Streaming Telemetry (Push-Modell). SNMP nutzt das Pull-Modell, Telemetry liefert
near-real-time Daten im Push-Modell.

## Kategorie 1: System-Ressourcen

**1. CPU-Auslastung**
- SNMP OID: `1.3.6.1.4.1.9.9.109.1.1.1.1.8` (cpmCPUTotal5minRev, CISCO-PROCESS-MIB)
- CLI: `show processes cpu`
- gNMI Path: `sys/procsys/sysload`

**2. Memory-Auslastung (used/free)**
- SNMP OID: `1.3.6.1.4.1.9.9.109.1.1.1.1.12` (cpmCPUMemoryUsed) / `.1.1.1.13` (free)
- CLI: `show system resources`
- gNMI Path: `sys/procsys/memused`

**3. Buffer-Auslastung**
- Über CISCO-BGP4-MIB und Buffer-MIBs werden Hits, Misses und Failures für verschiedene
  Buffer-Größen (small, big, medium, large, huge) überwacht.
- CLI: `show buffers`

**4. Uptime / System-Verfügbarkeit**
- SNMP OID: `1.3.6.1.2.1.1.3.0` (sysUpTime, RFC1213-MIB)
- CLI: `show version`

**5. Temperatur (Chassis/Module)**
- SNMP OID: `1.3.6.1.4.1.9.9.91` (CISCO-ENTITY-SENSOR-MIB)
- CLI: `show environment temperature`

## Kategorie 2: Hardware-Gesundheit

**6. Lüfter-Status**
- SNMP OID: `ENTITY-MIB::entPhysicalTable` + `cefcFanTrayOperStatus`
- CLI: `show environment fan`

**7. Netzteil-Status**
- SNMP OID: `ENTITY-MIB::entPhysicalTable` (CISCO-ENTITY-FRU-CONTROL-MIB)
- CLI: `show environment power`

**8. Modul-/Linecards-Status**
- Das ENTITY-MIB liefert Details über jedes Modul, jede Stromversorgung und jeden
  Fan-Tray innerhalb eines Switch-Chassis.
- CLI: `show module`

## Kategorie 3: Interface-Metriken

**9. Interface-Traffic (In/Out Bytes)**
- SNMP OID: `1.3.6.1.2.1.2.2.1.10` (ifInOctets) / `.16` (ifOutOctets) – IF-MIB
- CLI: `show interface`

**10. Interface-Fehler (Input/Output Errors)**
- Input-Errors: OID `1.3.6.1.2.1.2.2.1.14` (ifInErrors)
- Output-Errors: OID `1.3.6.1.2.1.2.2.1.20` (ifOutErrors) – IF-MIB
- CLI: `show interface counters errors`

**11. Interface-Status (Up/Down)**
- SNMP OID: `1.3.6.1.2.1.2.2.1.8` (ifOperStatus) – IF-MIB
- CLI: `show interface status`

**12. Interface-Auslastung / Bandbreite**
- SNMP OID: `1.3.6.1.2.1.2.2.1.5` (ifSpeed) + ifIn/OutOctets zur Berechnung
- gNMI Path: `sys/intf/phys/stats`

**13. Discards (verworfene Pakete)**
- SNMP OID: `1.3.6.1.2.1.2.2.1.13` (ifInDiscards) / `.19` (ifOutDiscards)
- CLI: `show interface counters`

## Kategorie 4: Routing-Protokolle

**14. BGP Session-Status**
- SNMP: CISCO-BGP4-MIB (`cbgpPeer2AddrFamilyPrefixTable`)
- CLI: `show bgp summary`
- Hinweis: Bei Nexus-Switches muss BGP-SNMP über VRF konfiguriert werden.

**15. BGP Prefix-Anzahl (empfangene/angekündigte Routen)**
- SNMP: `CISCO-BGP4-MIB::cbgpPeer2AddrFamilyPrefixTable`
- CLI: `show bgp ipv4 unicast summary`

**16. OSPF Neighbor-Status**
- SNMP OID: `OSPF-MIB::ospfNbrState (1.3.6.1.2.1.14.10.1.6)`
- CLI: `show ospf neighbors`

## Kategorie 5: Layer-2 / Switching

**17. VLAN-Nutzung / MAC-Table**
- SNMP OID: `CISCO-VLAN-MEMBERSHIP-MIB`
- CLI: `show mac address-table count`

**18. Spanning Tree Status (STP)**
- SNMP OID: `BRIDGE-MIB::dot1dStpPortState`
- CLI: `show spanning-tree summary`

## Kategorie 6: VxLAN / Fabric

**19. VxLAN Tunnel-Status / VTEP-Erreichbarkeit**
- gNMI Path: `sys/epId/nvo/VxLAN`
- CLI: `show nve peers`

**20. TCAM-Auslastung (Hardware-Ressourcen)**
- CLI: `show hardware access-list resource utilization`
- gNMI Path: `sys/epId/resource`
- Kein Standard-SNMP-OID – NX-API oder gNMI bevorzugt

## Abruf-Methoden im Vergleich

| Methode | Protokoll | Typ | Empfohlen für |
|---|---|---|---|
| **SNMP v2c/v3** | UDP 161 | Pull | Ältere Nexus, einfaches Setup |
| **gNMI Telemetry** | gRPC | Push | Nexus 9k ab NX-OS 9.3+, Echtzeit |
| **NX-API REST** | HTTP/S | Pull | Skripte, Ansible, ad-hoc |
| **CLI via SSH** | SSH | Pull | Debugging, Fallback |

### Schnellstart: SNMP aktivieren auf Nexus

```
feature snmp
snmp-server community PUBLIC ro
snmp-server host 192.168.1.100 traps version 2c PUBLIC
```

### Schnellstart: gNMI Telemetry aktivieren

```
feature telemetry
telemetry
  destination-group 1
    ip address 192.168.1.100 port 57000 protocol gRPC encoding GPB
  sensor-group 1
    path sys/intf depth unbounded
  subscription 1
    dst-grp 1
    snsr-grp 1 sample-interval 30000
```

---

# 2. Telegraf mit gRPC/gNMI für Cisco Nexus

**Frage:** Unterstützt der Agent Telegraf auch gRPC?

Ja, Telegraf unterstützt gRPC mit zwei dedizierten Plugins für Cisco Nexus. Beide sind
direkt in Telegraf integriert und benötigen keine separate Installation.

## Plugin 1: `inputs.gnmi` – Dial-in (Pull via gRPC)

Das gNMI-Plugin unterstützt Cisco NX-OS ab Version 9.3, Cisco IOS XR ab 6.5.1
und Cisco IOS XE ab 16.12. Es ist seit Telegraf v1.15.0 verfügbar.

Telegraf baut aktiv eine gRPC-Verbindung zum Switch auf und abonniert Sensor-Pfade:

```toml
[[inputs.gnmi]]
  addresses = ["192.168.1.1:57777"]
  username = "admin"
  password = "cisco123"
  encoding = "proto"           # oder "json_ietf"
  redial = "10s"               # Wiederverbindung bei Ausfall

  [[inputs.gnmi.subscription]]
    name = "interface"
    origin = "openconfig"
    path = "/interfaces/interface/state/counters"
    subscription_mode = "sample"
    sample_interval = "10s"

  [[inputs.gnmi.subscription]]
    name = "cpu"
    origin = "device"
    path = "sys/procsys/sysload"
    subscription_mode = "sample"
    sample_interval = "30s"
```

Aktivierung auf dem Nexus:

```
feature grpc
grpc gnmi max-concurrent-calls 16
grpc use-vrf default
```

## Plugin 2: `inputs.cisco_telemetry_mdt` – Dial-out (Push via gRPC)

Das MDT-Plugin empfängt Streaming-Telemetrie, die der Switch selbst aktiv zum
Telegraf-Server pusht. TLS wird nur bei gRPC unterstützt.

```toml
[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57000"
  max_msg_size = 4000000
  # tls_cert = "/etc/telegraf/cert.pem"
  # tls_key  = "/etc/telegraf/key.pem"
```

Aktivierung auf dem Nexus:

```
feature telemetry
telemetry
  destination-group 1
    ip address 192.168.1.100 port 57000 protocol gRPC encoding GPB
  sensor-group 1
    path sys/intf depth unbounded
  subscription 1
    dst-grp 1
    snsr-grp 1 sample-interval 30000
```

## Vergleich der beiden Plugins

| | `inputs.gnmi` | `inputs.cisco_telemetry_mdt` |
|---|---|---|
| Modell | Dial-in (Pull) | Dial-out (Push) |
| Verbindungsaufbau | Telegraf → Switch | Switch → Telegraf |
| Protokoll | gNMI über gRPC | Cisco MDT über gRPC |
| NX-OS Feature | `feature grpc` | `feature telemetry` |
| Standard | OpenConfig / IETF | Cisco-proprietär |
| Empfehlung | Flexibler, zukunftssicher | Etabliert, weit verbreitet |

## Gesamtarchitektur

```
Cisco Nexus
    │
    ├── gNMI (Dial-in)   ──→  Telegraf inputs.gnmi
    │                               │
    └── MDT  (Dial-out)  ──→  Telegraf inputs.cisco_telemetry_mdt
                                    │
                              InfluxDB / Prometheus
                                    │
                                 Grafana
```

Prüfung der Konfiguration auf dem Switch:

```
show grpc gnmi service statistics
```

---

# 3. Cisco ACI – Die 20 wichtigsten Metriken (Spine & Leaf, ~5.000 Ports)

**Frage:** Nenne die 20 wichtigsten Metriken für Cisco ACI mit Spine und Leaf-Switches mit ca. 5.000 Ports.

ACI ist als zweischichtige Leaf-Spine-Topologie aufgebaut. Der primäre Abrufweg
ist bei ACI immer die **APIC REST API** – nicht SNMP direkt auf den Switches.

## Kategorie 1: Fabric-Gesundheit (ACI-spezifisch)

**1. Fabric Health Score (gesamt)**
```
GET /api/node/mo/topology/health.json
```

**2. Node Health Score (pro Spine/Leaf)** (Skala 0–100)
```
GET /api/node/class/topSystem.json
    ?query-target=subtree&target-subtree-class=healthInst
```

**3. Fault Count nach Schweregrad (Critical / Major / Minor)**
```
GET /api/node/class/faultSummary.json
```

**4. APIC Cluster-Status**
```
GET /api/node/class/infraWiNode.json
```

## Kategorie 2: System-Ressourcen (Spine & Leaf)

**5. CPU-Auslastung (pro Node)**
```
GET /api/node/class/procEntity.json
```

**6. Memory-Auslastung (pro Node)**
```
GET /api/node/class/procMem.json
```

**7. APIC Disk-Auslastung**
```
GET /api/node/class/eqptStorage.json
```

## Kategorie 3: Interface-Metriken (bei 5.000 Ports kritisch)

**8. Interface Traffic (RX/TX Bytes, Raten)**
```
GET /api/node/class/l1PhysIf.json
    ?query-target=subtree&target-subtree-class=ethpmPhysIf
```

**9. Interface-Fehler (Input/Output Errors, CRC)**
```
GET /api/node/class/rmonEtherStats.json
```

**10. Interface-Status (Up/Down/Admin)**
```
GET /api/node/class/ethpmPhysIf.json
```

**11. Port-Auslastung / Bandbreite in % (Utilization)**
```
GET /api/node/class/eqptEgrTotal5min.json
GET /api/node/class/eqptIngrTotal5min.json
```

**12. Discards (Buffer-Drops pro Interface)**
```
GET /api/node/class/eqptEgrDropPkts5min.json
```

## Kategorie 4: Fabric-Overlay (ACI-spezifisch)

**13. VXLAN Tunnel-Status / VTEP-Erreichbarkeit**
```
GET /api/node/class/tunnelIf.json
```

**14. Endpoint Group (EPG) Health Score**
```
GET /api/node/mo/uni/tn-{tenant}/ap-{app}/epg-{epg}/health.json
```

**15. Bridge Domain / VLAN-Nutzung**
```
GET /api/node/class/fvBD.json
```

**16. Contract Fault / Policy-Fehler**
```
GET /api/node/class/fvAEPg.json
    ?query-target=subtree&target-subtree-class=faultInst
```

## Kategorie 5: Hardware-Gesundheit

**17. TCAM-Auslastung (kritisch bei großen Fabrics)**
```
GET /api/node/class/eqptcapacityPolUsage5min.json
```

**18. Temperatur (Spine-Chassis / Leaf-Module)**
```
GET /api/node/class/eqptTemperature.json
```

**19. Netzteil- und Lüfter-Status**
```
GET /api/node/class/eqptFan.json
GET /api/node/class/eqptPsu.json
```

## Kategorie 6: ISL / Fabric-Uplinks (Spine-Leaf-Links)

**20. ISL-Link-Auslastung (Spine-Uplinks)**
```
GET /api/node/class/eqptEgrTotal5min.json
    ?query-target-filter=eq(eqptEgrTotal5min.portT,"fab")
```

## Abruf-Methoden im Vergleich

| Methode | Protokoll | Empfohlen für |
|---|---|---|
| **APIC REST API** | HTTPS | Primärer Weg für alle ACI-Metriken |
| **SNMP direkt auf Nexus** | UDP 161 | Ergänzend für Hardware-OIDs |
| **Telegraf `inputs.http`** | HTTPS | REST-API-Polling via Telegraf |
| **Prometheus ACI Exporter** | HTTPS | Open-Source-Stack |

## Wichtiger Hinweis für 5.000 Ports: Rate-Limiting

Bei user/password-basierten API-Aufrufen greift ein Rate-Limit durch den NGINX-Prozess
des APIC. Bei großen Fabrics empfiehlt sich **zertifikatsbasierte Authentifizierung (X.509)**,
da diese nicht rate-limitiert wird.

```bash
openssl req -new -newkey rsa:4096 -days 3650 -nodes -x509 \
  -keyout telegraf.key -out telegraf.crt \
  -subj '/CN=telegraf/O=MyOrg/C=DE'
```

Telegraf-Konfiguration mit Zertifikat:

```toml
[[inputs.http]]
  urls = ["https://<apic-ip>/api/node/class/eqptEgrTotal5min.json"]
  method = "GET"
  tls_cert = "/etc/telegraf/telegraf.crt"
  tls_key  = "/etc/telegraf/telegraf.key"
  insecure_skip_verify = false
  interval = "60s"
  data_format = "json"
```

## Empfohlener Open-Source-Stack für ACI

```
APIC REST API
    │
    └── Telegraf inputs.http (zertifikatsbasiert)
            │
        InfluxDB / Prometheus
            │
         Grafana (Dashboard: ACI Fabric Overview)
```

---

# 4. Cisco-Produkte in einer ACI-Umgebung

**Frage:** Welche Cisco-Produkte können Teil von ACI sein?

## Controller (Pflicht)

**Cisco APIC** – Zentrale Policy-Verwaltung, physische Appliance auf UCS-Basis.

| Modell | Einsatz |
|---|---|
| APIC-L1 | Kleine Fabrics (bis ~80 Nodes) |
| APIC-L2 | Mittlere Fabrics |
| APIC-L3 | Große Fabrics (bis ~500 Nodes) |

## Fabric-Switches (Pflicht)

**Leaf-Switches (Nexus 9300-Serie)**

| Modell | Ports | Einsatz |
|---|---|---|
| N9K-C93180YC-EX | 48× 25GE + 6× 100GE | Standard Leaf |
| N9K-C93240YC-FX2 | 48× 25GE + 12× 100GE | High-Density Leaf |
| N9K-C9348GC-FXP | 48× 1GE + 4× 10GE | Access Leaf |
| N9K-C9364C | 64× 100GE | High-Speed Leaf / Spine |

**Spine-Switches (Nexus 9500-Serie)**

| Modell | Slots | Einsatz |
|---|---|---|
| N9K-C9504 | 4 Line-Card-Slots | Kleine Fabrics |
| N9K-C9508 | 8 Line-Card-Slots | Mittlere Fabrics |
| N9K-C9516 | 16 Line-Card-Slots | Große Fabrics / ~5.000 Ports |

## Management & Orchestrierung

- **Cisco Nexus Dashboard (ND)** – Zentrale Management-Plattform
- **Cisco Nexus Dashboard Orchestrator (NDO)** – Multi-Site Policy-Management
- **Cisco Nexus Dashboard Insights** – KI-gestützte Anomalie-Erkennung und Flow-Telemetrie

## Erweiterungsarchitekturen

| Architektur | Beschreibung |
|---|---|
| **Multi-Pod** | Mehrere Pods, ein APIC-Cluster, über IPN verbunden |
| **Multi-Site** | Separate Fabrics, durch NDO verwaltet, WAN-fähig |
| **Remote Leaf** | Einzelne Leafs an entfernten Standorten ohne eigene Spines |
| **vPod** | Virtuelle Pods auf ESXi ohne physische ACI-Hardware |

## L4–L7-Services (integrierbar per Service Graph)

| Produkt | Hersteller | Funktion |
|---|---|---|
| Cisco Firepower / FTD | Cisco | Next-Generation Firewall |
| Cisco ASA | Cisco | Firewall (klassisch) |
| F5 BIG-IP | F5 | Application Delivery Controller |
| Citrix NetScaler | Citrix | Load Balancer / ADC |

## Produktkarte

```
┌──────────────────────────────────────────────────────────┐
│                   Cisco ACI Ökosystem                    │
│  Management:   APIC  │  Nexus Dashboard  │  NDO         │
│  Fabric:       Nexus 9500 (Spine) │ Nexus 9300 (Leaf)   │
│  Erweiterung:  Multi-Pod │ Multi-Site │ Remote Leaf      │
│  Virtuell:     ACI Virtual Edge │ Cloud APIC            │
│  L4-L7:        Firepower │ ASA │ F5 │ Citrix            │
│  Compute:      UCS (APIC-Basis + Leaf-Anbindung)        │
└──────────────────────────────────────────────────────────┘
```

---

# 5. Was bedeutet eine Discard-Rate von 0,9?

**Frage:** Was bedeutet eine Discard-Rate von 0,9?

## Interpretation

Die Discard-Rate von 0,9 ist typischerweise ein **Prozentwert**: Von 1.000 Paketen
werden 9 verworfen. Das liegt direkt an der Warnschwelle.

## Bewertungsskala

| Discard-Rate | Bewertung |
|---|---|
| 0,0 % | Ideal |
| < 0,1 % | Akzeptabel (kurzfristige Bursts) |
| 0,1 – 0,5 % | Beobachten, Trend analysieren |
| **0,5 – 1,0 %** | **Warnschwelle – Ursache untersuchen** |
| > 1,0 % | Kritisch – sofortiger Handlungsbedarf |

## Wo tritt die Discard-Rate auf?

| Ort | Bedeutung bei 0,9 % |
|---|---|
| **Leaf-Downlink (Serverport)** | Server sendet zu schnell, Egress-Queue läuft voll |
| **Leaf-Uplink zum Spine** | Uplink-Überlastung, ggf. mehr Uplinks nötig (LAG) |
| **Spine-Fabric-Port** | Fabric-Überlastung, kritisch bei großen Fabrics |

## Diagnose-Befehle

```
# ACI REST API
GET /api/node/class/eqptEgrDropPkts5min.json

# NX-OS CLI
show queuing interface ethernet 1/1
show interface counters
```

## Ursachen und Maßnahmen

1. **Egress-Queue-Überlauf** → Queue-Statistiken prüfen
2. **Uplink dauerhaft > 70–80 % ausgelastet** → LAG oder zusätzliche Uplinks
3. **QoS-Policy** → werden Traffic-Klassen aktiv gedroppt?
4. **Mikroburst-Analyse** → Intervall auf 30 Sekunden reduzieren (5-min-Durchschnitt verschleiert Spitzen)

---

# 6. Was ist eine Line-Card in einem Switch?

**Frage:** Was ist eine Line-Card in einem Switch?

## Grundprinzip

Eine Line-Card ist eine steckbare Erweiterungsplatine in modularen, chassisbasierten
Switches. Sie stellt die eigentlichen Netzwerkports bereit.

## Modularer vs. Fixed Switch

| | **Modularer Switch (Chassis)** | **Fixed Switch (z. B. Nexus 9300)** |
|---|---|---|
| Aufbau | Gehäuse + steckbare Karten | Alles fest verbaut |
| Line-Cards | Ja, wechselbar | Nein |
| Beispiele | Nexus 9500, 7000, Catalyst 9600 | Nexus 9300, Catalyst 9300 |

## Inhalt einer Line-Card

- **Physische Interfaces** – z. B. 48× 10GE, 12× 400GE
- **NPU (Network Processing Unit)** – ASICs für Forwarding, ACLs, QoS
- **Puffer-Speicher** – für Queue-Management
- **Fabric-Interface** – Verbindung zur Switch-Fabric (Backplane)

## Architektur eines Chassis-Switch

```
┌─────────────────────────────────────┐
│           Chassis (z. B. Nexus 9508) │
│                                      │
│  ┌──────────┐    ┌──────────────┐   │
│  │Supervisor│    │  Fabric-     │   │
│  │  (SUP)   │    │  Module      │   │
│  └──────────┘    └──────────────┘   │
│                                      │
│  ┌──────────┐  ┌──────────┐         │
│  │Line-Card │  │Line-Card │  ...    │
│  │48× 10GE  │  │12× 400GE │         │
│  └──────────┘  └──────────┘         │
└─────────────────────────────────────┘
```

## Relevanz für das Monitoring

- **TCAM-Auslastung** wird pro Line-Card gemessen
- **Discard-Raten** entstehen häufig in den Egress-Queues der Line-Card
  (Head-of-Line Blocking bei Fabric-Überlast)
- **Temperatur-Metriken** werden pro Line-Card erfasst
- Bei ACI Nexus-9500-Spines kann eine überlastete Line-Card den gesamten Fabric-Pfad beeinträchtigen

## Cisco Nexus Beispiele

| Modell | Chassis | Typische Line-Cards |
|---|---|---|
| Nexus 9504 | 4 Slots | 48× 100GE, 12× 400GE |
| Nexus 9508 | 8 Slots | 48× 10GE, 36× 100GE |
| Nexus 9516 | 16 Slots | Mix aus 10G/40G/100G/400G |

---

# 7. AWS – Netzwerktopologie-Daten

**Frage:** Welche Daten bekomme ich bei AWS über meine genutzte Netzwerktopologie?

## Kategorie 1: VPC-Strukturdaten (statische Topologie)

Abrufbar über AWS Console, CLI oder API:

| Datenkategorie | Inhalt |
|---|---|
| **VPC** | VPC-ID, CIDR-Block (IPv4/IPv6), Tenancy, DNS-Einstellungen |
| **Subnetze** | Subnetz-ID, CIDR, Availability Zone, öffentlich/privat, verfügbare IPs |
| **Routing-Tabellen** | Routen, Subnetz-Zuordnungen, Gateway-Zuordnungen |
| **Internet Gateways** | Zugeordnete VPCs, Status |
| **NAT Gateways** | Subnetz, Elastic IP, Status, Datenübertragung |
| **Security Groups** | Inbound/Outbound-Regeln, Zuordnung zu Ressourcen |
| **Network ACLs** | Regeln pro Subnetz, Allow/Deny |
| **VPC Endpoints** | Gateway- oder Interface-Endpoints, zugeordnete Services |
| **VPC Peering** | Peering-Verbindungen zwischen VPCs |
| **Transit Gateway** | Attachments, Routing-Tabellen, Inter-VPC-Verbindungen |
| **Direct Connect** | Virtuelle Interfaces, BGP-Sessions, Bandbreite |
| **VPN Connections** | IPSec-Tunnel-Status, Customer Gateway, BGP-Routen |

## Kategorie 2: Verkehrsdaten – VPC Flow Logs

Flow Logs erfassen IP-Traffic zu und von Netzwerkschnittstellen.
Jeder Eintrag enthält:

```
version account-id interface-id srcaddr dstaddr srcport dstport
protocol packets bytes start end action log-status
```

Zusätzlich: Transit Gateway Flow Logs, DNS Query Logging via Route 53.

## Kategorie 3: Analyse- und Diagnose-Tools

| Tool | Funktion |
|---|---|
| **Network Manager Dashboard** | Globale Topologie-Visualisierung, CloudWatch-Metriken, Route-Analyse |
| **VPC Reachability Analyzer** | Pfadanalyse zwischen Ressourcen, prüft Security Groups, NACLs, Routing-Tabellen |
| **Network Access Analyzer** | Identifiziert ungewollte Netzwerkzugänge |

## Kategorie 4: CloudWatch-Netzwerkmetriken

| Metrik | Quelle | Inhalt |
|---|---|---|
| `NetworkIn` / `NetworkOut` | EC2 | Bytes pro Interface |
| `NetworkPacketsIn/Out` | EC2 | Pakete pro Interface |
| `ActiveFlowCount` | NAT Gateway | Aktive Verbindungen |
| `BytesInFromSource` | Transit Gateway | Datenvolumen |
| `TunnelState` | VPN | 0 = down, 1 = up |

## AWS CLI – Beispiele

```bash
# Alle VPCs
aws ec2 describe-vpcs

# Alle Subnetze
aws ec2 describe-subnets

# Routing-Tabellen
aws ec2 describe-route-tables

# Security Groups
aws ec2 describe-security-groups

# Transit Gateway Attachments
aws ec2 describe-transit-gateway-attachments

# Flow Logs aktivieren
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxxxxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs
```

## Was AWS bewusst NICHT preisgibt

Der implizite VPC-Router ist für den Benutzer nicht sichtbar und wird vollständig
von AWS verwaltet. Nicht einsehbar: physische Switch-Topologie, Backbone-Routing,
Inter-AZ-Latenzen auf Hardware-Ebene, interne BGP-Pfade zwischen Regionen.

---

# 8. STACKIT – Netzwerktopologie-Daten

**Frage:** Die gleiche Frage noch einmal, aber zu STACKIT.

STACKIT Network ist ein Software-Defined Network (SDN) auf Subnetz-Basis. Das
Grundprinzip ist ähnlich wie bei AWS VPC, aber die Tiefe der abrufbaren Daten
ist derzeit geringer.

## Kategorie 1: Netzwerkstruktur-Daten (statische Topologie)

Abrufbar über STACKIT Portal, CLI oder IaaS-API:

| Datenkategorie | Inhalt | API-Endpunkt |
|---|---|---|
| **Virtual Networks** | Netzwerk-ID, CIDR, Gateway, DNS-Server | `GET /v1/projects/{id}/networks` |
| **Subnetze** | Subnetz-ID, CIDR, IP-Bereich, Präfix-Länge | `GET /v1/projects/{id}/networks/{networkId}` |
| **Network Areas (SNA)** | Organisationsweite Netzwerkbereiche, Routing-Tabellen | IaaS-API |
| **Public IP Addresses** | Zugewiesene öffentliche IPs, Floating IPs | `GET /v1/projects/{id}/public-ips` |
| **Network Interfaces** | Interface-ID, MAC, IP, zugeordnete VM | `GET /v1/projects/{id}/network-interfaces` |
| **VPC Peering** | Peering zwischen zwei Private Networks | Portal + API |

## Kategorie 2: Sicherheits-Konfigurationsdaten

STACKIT Network Security kombiniert SDN mit Stateful Firewalls über Security Groups
und Site-to-Site VPN für Hybrid-Szenarien.

| Datenkategorie | Inhalt |
|---|---|
| **Security Groups** | Inbound/Outbound-Regeln, Protokoll, Port, CIDR |
| **Security Group Members** | Zugeordnete VMs und Interfaces |
| **IPsec VPN** | Tunnel-Status, IKEv2-Konfiguration, Remote-Netz |

## Kategorie 3: Konnektivitätsdaten

STACKIT unterstützt MPLS- und VPN-Direktverbindungen sowie IPsec (IKEv2) über die API.

## Kategorie 4: Observability & Monitoring

STACKIT Observability basiert auf Grafana, Prometheus, Thanos, Loki und Tempo.

| Metrik | Tool | Inhalt |
|---|---|---|
| VM-Netzwerk-Traffic | Prometheus | Bytes/s pro Interface |
| Load Balancer Metriken | Prometheus | Requests, Connections, Fehlerrate |
| DNS-Abfragen | STACKIT DNS Logs | Abgefragte Domains, Antwortzeiten |
| Audit Logs | Telemetry Router (OTLP) | API-Calls, Konfigurationsänderungen |

## STACKIT CLI – Beispiele

```bash
# Alle Netzwerke eines Projekts
stackit beta network list --project-id <project-id>

# Details zu einem Netzwerk
stackit beta network describe <network-id> --project-id <project-id>

# Public IPs
stackit beta public-ip list --project-id <project-id>

# Security Groups
stackit beta security-group list --project-id <project-id>

# VPN Verbindungen
stackit vpn ipsec-connection list --project-id <project-id>
```

## Vergleich AWS vs. STACKIT

| Funktion | AWS | STACKIT |
|---|---|---|
| **Flow Logs** (IP-Traffic-Aufzeichnung) | ✅ VPC Flow Logs | ❌ Nicht verfügbar |
| **Reachability Analyzer** | ✅ | ❌ Nicht verfügbar |
| **Network Access Analyzer** | ✅ | ❌ Nicht verfügbar |
| **Traffic Mirroring** | ✅ | ❌ Nicht verfügbar |
| **Transit Gateway** | ✅ | ⚠️ Network Area (ähnlich, eingeschränkter) |
| **Routing-Tabellen einsehbar** | ✅ vollständig | ⚠️ eingeschränkt |
| **Globales Netzwerk-Dashboard** | ✅ Network Manager | ❌ Nicht verfügbar |
| **DNS Query Logging** | ✅ Route 53 Logs | ⚠️ DNS Resolver in Public Preview |
| **Observability Stack** | CloudWatch | Grafana + Prometheus + Loki |
| **Datenschutz / DSGVO** | ⚠️ US-Anbieter | ✅ Deutsche Rechenzentren |

---

## Quellen

### Cisco Nexus / gNMI
- Telegraf gNMI Plugin: https://docs.influxdata.com/telegraf/v1/input-plugins/gnmi/
- Telegraf Cisco MDT Plugin: https://docs.influxdata.com/telegraf/v1/input-plugins/cisco_telemetry_mdt/
- Cisco NX-OS MIB Quick Reference: https://www.cisco.com/c/en/us/td/docs/switches/datacenter/sw/mib/quickreference/cisco-nexus-7000-series-and-9000-series-nx-os-mib-quick-reference.html

### Cisco ACI
- Cisco APIC REST API Configuration Guide: https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/4-x/rest-api-config/
- Cisco ACI Design Guide: https://www.cisco.com/c/en/us/td/docs/dcn/whitepapers/cisco-application-centric-infrastructure-design-guide.html
- Cisco ACI Multi-Site White Paper: https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-739609.html

### AWS
- Amazon VPC Dokumentation: https://docs.aws.amazon.com/de_de/vpc/latest/userguide/
- VPC Flow Logs: https://docs.aws.amazon.com/de_de/vpc/latest/userguide/flow-logs.html
- VPC Reachability Analyzer: https://docs.aws.amazon.com/vpc/latest/reachability/

### STACKIT
- STACKIT Core Networking Dokumentation: https://docs.stackit.cloud/de/products/network/core-networking/
- STACKIT Observability: https://docs.stackit.cloud/products/logging-and-monitoring/observability/
- STACKIT API Referenz: https://docs.api.eu01.stackit.cloud/
