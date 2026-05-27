# Cisco-Produkte in einer ACI-Umgebung

ACI ist kein einzelnes Produkt, sondern ein **Ökosystem** aus Hardware, Software und
Cloud-Integrationen. Kernstück der ACI-Fabric sind Switches der Nexus-9000-Serie, die
in einer Spine-Leaf-Topologie konfiguriert werden. Richtlinien werden im APIC
(Application Policy Infrastructure Controller) beschrieben.

---

## 1. Controller (Pflicht)

### Cisco APIC (Application Policy Infrastructure Controller)

Der APIC ist der zentrale Konfigurationspunkt für Policies sowie der Speicher- und
Verarbeitungsort für Statistiken, Telemetrie und Anwendungs-Health-Informationen. Er
ist eine physische Appliance auf Basis eines Cisco-UCS-Rack-Servers mit zwei Interfaces
zur Anbindung an Leaf-Switches sowie Gigabit-Ethernet-Interfaces für Out-of-Band-Management.

| Modell | Einsatz |
|---|---|
| APIC-L1 | Kleine Fabrics (bis ~80 Nodes) |
| APIC-L2 | Mittlere Fabrics |
| APIC-L3 | Große Fabrics (bis ~500 Nodes) |

Cluster: typischerweise 3, 5 oder 7 Knoten für Redundanz.

---

## 2. Fabric-Switches (Pflicht)

Zu einer ACI-Fabric gehören in der Regel Leaf Switches vom Typ Nexus 9300 und
Spine Switches vom Typ Nexus 9500.

### Leaf-Switches (Nexus 9300-Serie)

| Modell | Ports | Einsatz |
|---|---|---|
| N9K-C93180YC-EX | 48× 25GE + 6× 100GE | Standard Leaf |
| N9K-C93240YC-FX2 | 48× 25GE + 12× 100GE | High-Density Leaf |
| N9K-C9348GC-FXP | 48× 1GE + 4× 10GE | Access Leaf |
| N9K-C9364C | 64× 100GE | High-Speed Leaf / Spine |

### Spine-Switches (Nexus 9500-Serie)

| Modell | Slots | Einsatz |
|---|---|---|
| N9K-C9504 | 4 Line-Card-Slots | Kleine Fabrics |
| N9K-C9508 | 8 Line-Card-Slots | Mittlere Fabrics |
| N9K-C9516 | 16 Line-Card-Slots | Große Fabrics / ~5.000 Ports |

---

## 3. Management & Orchestrierung

### Cisco Nexus Dashboard (ND)

Zentrale Management-Plattform für ACI, als virtuelle oder physische Appliance.
Basis für alle nachgelagerten Dashboard-Services wie Insights und Orchestrator.

### Cisco Nexus Dashboard Orchestrator (NDO, früher MSO)

Der NDO-Service läuft auf einem Nexus-Dashboard-Cluster (3 virtuelle oder physische
Nodes) und ermöglicht es, Tenant-Policies über alle ACI-Sites zu stretchen.
Pflicht für Multi-Site-Deployments.

### Cisco Nexus Dashboard Insights

Anomalie-Erkennung, Health-Analyse und Flow-Telemetrie. Läuft als Service auf dem
Nexus Dashboard und ergänzt das reine Monitoring durch KI-gestützte Auswertung.

---

## 4. Erweiterungsarchitekturen

### Multi-Pod

Bei ACI Multi-Pod wird ein einziger APIC-Cluster für mehrere separate ACI-Pods
eingesetzt. Die Pods sind über ein IP-Routed-Netzwerk (IPN) verbunden und
funktionieren operativ als einzelne ACI-Fabric.

### Multi-Site

ACI Multi-Site adressiert die Notwendigkeit zur Fault-Domain-Isolation über
verschiedene ACI-Fabrics, die über ein IP-Netzwerk (auch WAN) verbunden sind –
ohne dass Multicast-Routing im IP-Netzwerk nötig ist. Separate Sites werden
durch den Nexus Dashboard Orchestrator zentral verwaltet.

Skalierung: Bis zu 500 Leaf-Switches pro Fabric / Site.

### Remote Leaf

Einzelne Leaf-Switches an entfernten Standorten, die über WAN mit der Haupt-Fabric
verbunden sind, ohne eigene Spines vor Ort.

### vPod (Virtual Pod)

ACI vPod erweitert eine Multi-Pod-Fabric an entfernte Standorte, wo mindestens zwei
ESXi-Server verfügbar sind und keine physische ACI-Hardware gewünscht wird. Benötigt
werden virtuelle Spine- und Leaf-Appliances sowie die Cisco ACI Virtual Edge.

---

## 5. Virtuelle und Cloud-Komponenten

### Cisco ACI Virtual Edge (AVE)

Virtueller Distributed Switch für VMware-Umgebungen. Bringt das ACI-Policy-Modell
direkt auf den Hypervisor und ermöglicht Mikrosegmentierung auf VM-Ebene.

### Cisco ACI für Public Cloud

Integration mit AWS, Azure und GCP über den Nexus Dashboard Orchestrator.
Ermöglicht konsistente Policy-Durchsetzung über On-Premises und Cloud hinweg.

### Cisco Cloud APIC

APIC-Instanz als virtuelle Maschine in der Public Cloud. Verwaltet Cloud-native
Netzwerkkonstrukte mit denselben ACI-Policy-Modellen wie On-Premises.

---

## 6. L4–L7-Services (integrierbar per Service Graph)

ACI kann externe Netzwerkdienste per **Service Graph** automatisiert einbinden und
in den Traffic-Pfad zwischen Endpoint Groups einfügen:

| Produkt | Hersteller | Funktion |
|---|---|---|
| **Cisco Firepower / FTD** | Cisco | Next-Generation Firewall (L4–L7) |
| **Cisco ASA** | Cisco | Firewall (klassisch) |
| **Cisco ACI Load Balancer** | Cisco | L4-Loadbalancing via Service Graph |
| **F5 BIG-IP** | F5 | Application Delivery Controller |
| **Citrix NetScaler** | Citrix | Load Balancer / ADC |

---

## 7. Server-Infrastruktur

### Cisco UCS (Unified Computing System)

Der APIC basiert physisch auf einem Cisco-UCS-Rack-Server. UCS-Server können
außerdem als Compute-Nodes direkt an ACI-Leaf-Switches angeschlossen werden und
profitieren dabei von automatisierter Port-Provisionierung über APIC-Policies.

---

## Zusammenfassung: ACI-Produktkarte

```
┌──────────────────────────────────────────────────────────┐
│                   Cisco ACI Ökosystem                    │
│                                                          │
│  Management:   APIC  │  Nexus Dashboard  │  NDO         │
│                Nexus Dashboard Insights                  │
│                                                          │
│  Fabric:       Nexus 9500 (Spine)                       │
│                Nexus 9300 (Leaf)                        │
│                                                          │
│  Erweiterung:  Multi-Pod  │  Multi-Site  │  Remote Leaf │
│                vPod                                      │
│                                                          │
│  Virtuell:     ACI Virtual Edge  │  Cloud APIC          │
│                ACI für AWS / Azure / GCP                 │
│                                                          │
│  L4-L7:        Firepower  │  ASA  │  F5  │  Citrix      │
│                                                          │
│  Compute:      UCS (APIC-Basis + direkte Leaf-Anbindung)│
└──────────────────────────────────────────────────────────┘
```

---

## Quellen

- Cisco ACI Design Guide:
  https://www.cisco.com/c/en/us/td/docs/dcn/whitepapers/cisco-application-centric-infrastructure-design-guide.html
- Cisco ACI Multi-Site White Paper:
  https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-739609.html
- Cisco ACI Multi-Pod White Paper:
  https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-737855.html
- Cisco ACI Remote Leaf White Paper:
  https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-740861.html
- Cisco ACI Lizenzierungsleitfaden:
  https://www.cisco.com/site/de/de/products/networking/cloud-networking/application-centric-infrastructure/licensing.html
