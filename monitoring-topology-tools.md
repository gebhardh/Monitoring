# Monitoring-Tools: IT-Topologie, Health-State und Service-Map-Drilldown

---

## 1. Tools für IT-Topologie + Health-State in einer GUI

### Übersicht

Folgende Tools kombinieren IT-Topologie als Graph **und** Health-State-Visualisierung
in einer gemeinsamen Oberfläche:

---

### Open Source

#### Zabbix
- „Single Pane of Glass"-Überblick: Daten und Monitoring-Events in Graphen, Listen,
  Geomaps und **Network Topology Maps**
- Automatische Topologieerkennung via LLDP/CDP
- Health-States direkt in der Map farblich dargestellt
- Verschachtelte Maps (Map-in-Map) ermöglichen hierarchischen Drilldown

#### Checkmk
- Eigene **Network Layer Visualization** (NVDCT-Plugin)
- Topologie wird aus HW/SW-Inventory-Daten automatisch aufgebaut
- Service-Health-States direkt in die Topology-Map integriert
- Wird explizit für Topologie-Visualisierung und Flexibilität gelobt

#### OpenNMS
- Modular, enterprise-fokussiert, gut skalierbar
- Eigene Topology-Map-Ansicht mit farbkodierten Health-States

#### LibreNMS
- Automatische Layer-2/3-Topologieerkennung via LLDP/CDP/STP
- Integrierte Health-Visualisierung
- Sehr ressourceneffizient

---

### Kommerziell

#### ManageEngine OpManager
- Business Views, 3D-Datacenter-Ansichten, Topology Maps, Heat Maps
- Anpassbare Dashboards mit direktem Blick auf den Netzwerkstatus
- Hierarchischer Drilldown in Geräte-Ebene

#### SolarWinds NPM / Network Topology Mapper
- Breiteste Auswahl an Discovery-Protokollen (SNMP, ICMP, ARP, CDP, LLDP, WMI)
- Health-States live in die Map eingeblendet
- NetPath-Feature für Pfadanalyse

#### Domotz
- Automatisierte Topology Maps, Interface-Monitoring, Multi-Site-Alerting
- Seit Januar 2026: **Topology Snapshots** für punktgenauen Zustandsvergleich
- STP-Status-Checks seit März 2025

#### OpenText NNMi (Network Node Manager i)
- Klassisches Enterprise-Tool
- Tiefes Layer-2/3-Topologie-Graph mit integriertem Health-State-Management

---

### Vergleich

| Tool | Topologie-Graph | Health-State | Open Source | Besonderheit |
|---|---|---|---|---|
| **Zabbix** | ✅ LLDP/CDP auto | ✅ in Map | ✅ | Weit verbreitet, gute SNMP-Abdeckung |
| **Checkmk** | ✅ NVDCT-Plugin | ✅ in Map | ✅ Community Ed. | Sehr gute Autodiscovery |
| **LibreNMS** | ✅ L2/L3 | ✅ | ✅ | Ressourcenschonend |
| **OpenNMS** | ✅ | ✅ | ✅ | Enterprise-Skalierung |
| **OpManager** | ✅ inkl. 3D-View | ✅ | ❌ | Sehr umfangreich |
| **SolarWinds NPM** | ✅ | ✅ | ❌ | Breiteste Protokollunterstützung |
| **NNMi** | ✅ | ✅ | ❌ | Klassischer Enterprise-Standard |

---

## 2. Tools mit Service-Map → Infrastruktur-Topologie Drilldown

Der echte Drilldown von einer Business-Service-Ansicht in die darunterliegende
Infrastruktur-Topologie (per Klick navigierbar) ist eine spezifische Fähigkeit,
die nicht alle Tools beherrschen.

---

### Klare Empfehlungen (nativer Drilldown)

#### Dynatrace — stärkste native Lösung
- **Smartscape®** mappt automatisch jeden Service und jede Abhängigkeit:
  Business-Applikation → Application Service → Prozess → Host → Netzwerk
- Drilldown in 1 Klick, vollautomatisch ohne manuelle Konfiguration
- Integration mit ServiceNow CMDB über Service Graph Connector:
  real-time Topologie-Daten, Dependency Mapping, automatische CI-Bindung
- Davis® AI für deterministische Root-Cause-Analyse

#### ServiceNow ITOM (Service Mapping)
- **Multi-Source Service Mapping**: kombiniert Topology-Daten aus mehreren
  Quellen gleichzeitig (Dynatrace, Datadog, AWS, Azure, eigene Discovery)
- Drilldown führt von der Business-Service-View direkt in die CI-Infrastruktur
- CMDB als zentrale „Source of Truth" für alle Abhängigkeiten
- Erfordert ITOM-Lizenz und Discovery-Lizenz

#### Datadog
- Native **Service Map** mit Drilldown in Infrastructure Map
- Agent-basierte Erkennung aller Abhängigkeiten
- Verbindet APM-Traces mit Infrastruktur-Metriken

---

### Gut geeignet (mit Konfiguration)

#### Zabbix
- Verschachtelte Maps (Map-in-Map) ermöglichen manuell konfigurierten Drilldown
- Business-Service-Map verlinkt auf Infrastruktur-Topology-Map
- Erfordert manuelle Pflege der Map-Hierarchie
- Keine automatische Service-Abhängigkeitserkennung

#### Checkmk
- Business-Process-Monitoring mit Statusinformation
- Direktlinks auf Host- und Service-Ansichten
- Network Layer Visualization (NVDCT) für Infrastruktur-Topologie
- Übergang ist Link-basiert, kein automatischer Drilldown

#### ManageEngine OpManager
- Business Views hierarchisch aufbaubar
- Drilldown in Geräte-Ebene mit Topology- und Health-Ansicht
- Discovery-basierte Automatisierung

---

### Vergleich nach Drilldown-Tiefe

| Tool | Service-Map | Drilldown | Infra-Topologie | Automatisierung |
|---|---|---|---|---|
| **Dynatrace** | ✅ Smartscape® | ✅ nativ, 1 Klick | ✅ vollständig | ✅ vollautomatisch |
| **ServiceNow ITOM** | ✅ Service Graph | ✅ nativ, CMDB | ✅ Multi-Source | ✅ Discovery-basiert |
| **Datadog** | ✅ Service Map | ✅ nativ | ✅ Infrastructure Map | ✅ Agent-basiert |
| **Zabbix** | ✅ Map-in-Map | ⚠️ manuell | ✅ | ⚠️ manuell |
| **Checkmk** | ✅ Business Process | ⚠️ Link-basiert | ✅ NVDCT | ⚠️ semi-automatisch |
| **OpManager** | ✅ Business Views | ✅ hierarchisch | ✅ | ✅ Discovery |
| **NNMi** | ⚠️ begrenzt | ⚠️ L2/L3 only | ✅ Netzwerk | ✅ SNMP-Discovery |

---

## Fazit und Empfehlung

**Vollständiger nativer Drilldown** (Service → Infrastruktur, automatisch):
→ **Dynatrace** (kommerziell) oder **Datadog** (kommerziell)

**Open-Source mit manuellem Drilldown:**
→ **Zabbix** (Map-in-Map) oder **Checkmk** (Business Process + NVDCT)

**Enterprise CMDB-zentriert mit Multi-Source-Topologie:**
→ **ServiceNow ITOM** in Kombination mit Dynatrace oder Datadog als Datenquelle

Da NNMi bereits im Stack vorhanden ist, wäre **Checkmk** oder **Zabbix** die
naheliegendste Open-Source-Ergänzung mit vergleichbarem Topologie- und
Health-Feature-Set – beide lassen sich zudem mit NetBox und VictoriaMetrics
integrieren.
