# 20 wichtigste Metriken für Cisco ACI (Spine & Leaf, ~5.000 Ports)

ACI ist als zweischichtige Leaf-Spine-Topologie aufgebaut. Leaf Switches verbinden
sich mit Servern und Storage, während Spine Switches die Leaf Switches zusammen in
ein größeres Fabric packen. Typische Hardware sind Nexus-9300-Leafs und
Nexus-9500-Spines, gesteuert durch einen oder mehrere APIC-Controller.

Der primäre Abrufweg ist bei ACI immer die **APIC REST API** – nicht SNMP direkt
auf den Switches.

---

## Kategorie 1: Fabric-Gesundheit (ACI-spezifisch)

**1. Fabric Health Score (gesamt)**

Alle Health Scores entstammen der `healthInst`-Klasse im MIT und lassen sich per
REST API abrufen. Der System Health Score berechnet sich als gewichteter Durchschnitt
aller Leafs, Spines und Endpoints im Fabric.

```
GET /api/node/mo/topology/health.json
```

**2. Node Health Score (pro Spine/Leaf)**

Gibt den Health Score jedes einzelnen Nodes zurück (Skala 0–100).

```
GET /api/node/class/topSystem.json
    ?query-target=subtree&target-subtree-class=healthInst
```

**3. Fault Count nach Schweregrad (Critical / Major / Minor)**

Fault Counts werden nach Typ, Domain und Schweregrad (Critical, Major, Minor)
kategorisiert und pro Switch oder Tenant ausgewertet.

```
GET /api/node/class/faultSummary.json
```

**4. APIC Cluster-Status**

Der APIC läuft in einem Cluster von 3, 5 oder 7 Knoten abhängig von der
Fabric-Größe. Der Cluster-Status lässt sich per REST API abfragen.

```
GET /api/node/class/infraWiNode.json
```

---

## Kategorie 2: System-Ressourcen (Spine & Leaf)

**5. CPU-Auslastung (pro Node)**

Die `procEntity`-Klasse liefert CPU- und Memory-Auslastung für APIC und Switches.
Sie ist ein Container für alle laufenden Prozesse mit detaillierten Kennzahlen.

```
GET /api/node/class/procEntity.json
```

**6. Memory-Auslastung (pro Node)**

Liefert used/free Memory pro Spine und Leaf.

```
GET /api/node/class/procMem.json
```

**7. APIC Disk-Auslastung**

Der APIC enthält mehrere Disks und Dateisysteme. Per REST API lässt sich die
Disk-Auslastung aller Partitionen abrufen.

```
GET /api/node/class/eqptStorage.json
```

---

## Kategorie 3: Interface-Metriken (bei 5.000 Ports kritisch)

**8. Interface Traffic (RX/TX Bytes, Raten)**

Die REST API stellt Interface-Statistiken bereit, darunter RX/TX-Zähler,
30-Sekunden-Raten, 5-Minuten-Raten sowie Unicast- und Multicast-Pakete.

```
GET /api/node/class/l1PhysIf.json
    ?query-target=subtree&target-subtree-class=ethpmPhysIf
```

**9. Interface-Fehler (Input/Output Errors, CRC)**

Liefert CRC-Fehler, Runts, Giants, Collisions pro Port.

```
GET /api/node/class/rmonEtherStats.json
```

**10. Interface-Status (Up/Down/Admin)**

Bei 5.000 Ports essenziell für schnelle Ausfallerkennung.

```
GET /api/node/class/ethpmPhysIf.json
```

**11. Port-Auslastung / Bandbreite in % (Utilization)**

Aggregierte Ingress/Egress-Statistiken pro Interface im 5-Minuten-Intervall.

```
GET /api/node/class/eqptEgrTotal5min.json
GET /api/node/class/eqptIngrTotal5min.json
```

**12. Discards (Buffer-Drops pro Interface)**

Kritisch bei großen Fabrics: zeigt Überlast an Uplinks oder Spine-Ports an.

```
GET /api/node/class/eqptEgrDropPkts5min.json
```

---

## Kategorie 4: Fabric-Overlay (ACI-spezifisch)

**13. VXLAN Tunnel-Status / VTEP-Erreichbarkeit**

Zeigt aktive VXLAN-Tunnel zwischen Leafs über die Spine-Fabric.

```
GET /api/node/class/tunnelIf.json
```

**14. Endpoint Group (EPG) Health Score**

ACI stellt Health Scores auf Tenant-, EPG- und Fabric-Ebene bereit. Über die
APIC REST API lassen sich alle Ebenen granular abfragen.

```
GET /api/node/mo/uni/tn-{tenant}/ap-{app}/epg-{epg}/health.json
```

**15. Bridge Domain / VLAN-Nutzung**

Zeigt aktive Bridge Domains, zugehörige VLANs und Endpoints.

```
GET /api/node/class/fvBD.json
```

**16. Contract Fault / Policy-Fehler**

Zeigt Policy-Verletzungen und nicht aufgelöste Contracts.

```
GET /api/node/class/fvAEPg.json
    ?query-target=subtree&target-subtree-class=faultInst
```

---

## Kategorie 5: Hardware-Gesundheit

**17. TCAM-Auslastung (kritisch bei großen Fabrics)**

Bei 5.000 Ports und vielen EPGs/Contracts schnell kritisch. Unbedingt überwachen.

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

---

## Kategorie 6: ISL / Fabric-Uplinks (Spine-Leaf-Links)

**20. ISL-Link-Auslastung (Spine-Uplinks)**

Bei 5.000 Ports ist die Auslastung der Spine-Uplinks der häufigste Engpass.
Gefiltert auf Fabric-Ports (Uplink-Ports zwischen Leaf und Spine).

```
GET /api/node/class/eqptEgrTotal5min.json
    ?query-target-filter=eq(eqptEgrTotal5min.portT,"fab")
```

---

## Abruf-Methoden im Vergleich

| Methode | Protokoll | Empfohlen für |
|---|---|---|
| **APIC REST API** | HTTPS | Primärer Weg für alle ACI-Metriken |
| **SNMP direkt auf Nexus** | UDP 161 | Ergänzend für Hardware-OIDs |
| **Telegraf `inputs.http`** | HTTPS | REST-API-Polling via Telegraf |
| **Prometheus ACI Exporter** | HTTPS | Open-Source-Stack |

---

## Wichtiger Hinweis für 5.000 Ports: Rate-Limiting

Bei user/password-basierten API-Aufrufen greift ein Rate-Limit durch den
NGINX-Prozess des APIC. Bei großen Fabrics mit vielen Interfaces empfiehlt sich
**zertifikatsbasierte Authentifizierung (X.509)**, da diese nicht rate-limitiert wird.

Zertifikat generieren:

```bash
openssl req -new -newkey rsa:4096 -days 3650 -nodes -x509 \
  -keyout telegraf.key -out telegraf.crt \
  -subj '/CN=telegraf/O=MyOrg/C=DE'
```

Zertifikat in APIC registrieren:

```
# Im APIC GUI:
# Admin → AAA → Users → telegraf → X.509 Certificates → Add
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

---

## Empfohlener Open-Source-Stack für ACI

```
APIC REST API
    │
    └── Telegraf inputs.http (zertifikatsbasiert)
            │
        InfluxDB / Prometheus
            │
         Grafana
         (Dashboard: ACI Fabric Overview)
```

---

## Quellen

- Cisco APIC REST API Configuration Guide:
  https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/4-x/rest-api-config/
- DCLessons – Monitoring ACI via REST API: https://www.dclessons.com/monitoring-aci-via-rest-api
- NWMichl Blog – Monitor Cisco ACI via REST API: https://nwmichl.wordpress.com/2020/04/19/monitor-cisco-aci-via-rest-api/
- Cisco ACI/APIC Dynatrace Extension Docs: https://docs.dynatrace.com/docs/observe/infrastructure-observability/extensions/cisco-aciapic
