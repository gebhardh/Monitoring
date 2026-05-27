# Cisco Nexus – Metriken und Telegraf gRPC/gNMI

---

## Teil 1: Die 20 wichtigsten Metriken für Cisco Nexus

NX-OS bietet mehrere Mechanismen zur Datenerhebung: SNMP, CLI, Syslog und moderne
Streaming Telemetry (Push-Modell). SNMP nutzt das Pull-Modell, Telemetry liefert
near-real-time Daten im Push-Modell.

---

### Kategorie 1: System-Ressourcen

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

---

### Kategorie 2: Hardware-Gesundheit

**6. Lüfter-Status**
- SNMP OID: `ENTITY-MIB::entPhysicalTable` + `cefcFanTrayOperStatus`
- CLI: `show environment fan`

**7. Netzteil-Status**
- SNMP OID: `ENTITY-MIB::entPhysicalTable` (CISCO-ENTITY-FRU-CONTROL-MIB)
- CLI: `show environment power`

**8. Modul-/Linecards-Status**
- Das ENTITY-MIB liefert Details über jedes Modul, jede Stromversorgung und jeden
  Fan-Tray innerhalb eines Switch-Chassis und ermöglicht eine vollständige Chassis-Übersicht.
- CLI: `show module`

---

### Kategorie 3: Interface-Metriken

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

---

### Kategorie 4: Routing-Protokolle

**14. BGP Session-Status**
- SNMP: CISCO-BGP4-MIB (`cbgpPeer2AddrFamilyPrefixTable`)
- CLI: `show bgp summary`
- Hinweis: Bei Nexus-Switches muss BGP-SNMP über VRF konfiguriert werden, da
  Standard-OIDs wie `bgpEstablished (1.3.6.1.2.1.15.7.1)` ohne VRF-Kontext nicht antworten.

**15. BGP Prefix-Anzahl (empfangene/angekündigte Routen)**
- SNMP: `CISCO-BGP4-MIB::cbgpPeer2AddrFamilyPrefixTable`
- CLI: `show bgp ipv4 unicast summary`

**16. OSPF Neighbor-Status**
- SNMP OID: `OSPF-MIB::ospfNbrState (1.3.6.1.2.1.14.10.1.6)`
- CLI: `show ospf neighbors`

---

### Kategorie 5: Layer-2 / Switching

**17. VLAN-Nutzung / MAC-Table**
- SNMP OID: `CISCO-VLAN-MEMBERSHIP-MIB`
- CLI: `show mac address-table count`

**18. Spanning Tree Status (STP)**
- SNMP OID: `BRIDGE-MIB::dot1dStpPortState`
- CLI: `show spanning-tree summary`

---

### Kategorie 6: VxLAN / Fabric

**19. VxLAN Tunnel-Status / VTEP-Erreichbarkeit**
- gNMI Path: `sys/epId/nvo/VxLAN`
- CLI: `show nve peers`

**20. TCAM-Auslastung (Hardware-Ressourcen)**
- CLI: `show hardware access-list resource utilization`
- gNMI Path: `sys/epId/resource`
- Kein Standard-SNMP-OID – NX-API oder gNMI bevorzugt

---

### Abruf-Methoden im Vergleich

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

## Teil 2: Telegraf mit gRPC/gNMI für Cisco Nexus

Telegraf unterstützt gRPC mit zwei dedizierten Plugins für Cisco Nexus. Beide sind
direkt in Telegraf integriert und benötigen keine separate Installation.

---

### Plugin 1: `inputs.gnmi` – Dial-in (Pull via gRPC)

Das gNMI-Plugin wurde für Cisco-Geräte optimiert und unterstützt Cisco NX-OS ab
Version 9.3, Cisco IOS XR ab 6.5.1 und Cisco IOS XE ab 16.12. Es ist seit
Telegraf v1.15.0 verfügbar.

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

---

### Plugin 2: `inputs.cisco_telemetry_mdt` – Dial-out (Push via gRPC)

Das MDT-Plugin empfängt Streaming-Telemetrie, die der Switch selbst aktiv zum
Telegraf-Server pusht. Der Transport kann TCP oder gRPC sein, TLS wird nur bei
gRPC unterstützt.

```toml
[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57000"   # Telegraf hört auf eingehende Verbindungen
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

---

### Vergleich der beiden Plugins

| | `inputs.gnmi` | `inputs.cisco_telemetry_mdt` |
|---|---|---|
| Modell | Dial-in (Pull) | Dial-out (Push) |
| Verbindungsaufbau | Telegraf → Switch | Switch → Telegraf |
| Protokoll | gNMI über gRPC | Cisco MDT über gRPC |
| NX-OS Feature | `feature grpc` | `feature telemetry` |
| Standard | OpenConfig / IETF | Cisco-proprietär |
| Empfehlung | Flexibler, zukunftssicher | Etabliert, weit verbreitet |

---

### Gesamtarchitektur

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

Die gRPC/gNMI-Konfiguration auf dem Switch lässt sich mit
`show grpc gnmi service statistics` überprüfen – dort sind Port, VRF und
aktive Subscriptions sichtbar.

---

## Quellen

- Telegraf gNMI Plugin Dokumentation: https://docs.influxdata.com/telegraf/v1/input-plugins/gnmi/
- Telegraf Cisco MDT Plugin: https://docs.influxdata.com/telegraf/v1/input-plugins/cisco_telemetry_mdt/
- Cisco Blog – OpenConfig Telemetry auf NX-OS: https://blogs.cisco.com/datacenter/hot-off-the-press-introducing-openconfig-telemetry-on-nx-os-with-gnmi-and-telegraf
- Cisco White Paper – Data Center Telemetry mit gNMI: https://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/white-paper-c11-744191.html
- Cisco NX-OS MIB Quick Reference: https://www.cisco.com/c/en/us/td/docs/switches/datacenter/sw/mib/quickreference/cisco-nexus-7000-series-and-9000-series-nx-os-mib-quick-reference.html
