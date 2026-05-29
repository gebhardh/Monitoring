# Latenzmessung von Einzelverbindungen in Cisco ACI

## Kurze Antwort

**Ja**, ACI kann Latenzen einzelner Flows messen – aber nur mit den richtigen
Komponenten und Voraussetzungen.

---

## Methode 1: Nexus Dashboard Insights (NDI) – Flow Telemetry

Das ist der primäre und leistungsfähigste Weg.

Nexus Dashboard Insights (NDI) liefert tiefe Einblicke auf Flow-Ebene,
einschließlich durchschnittlicher Latenz und Paket-Drop-Indikatoren. Es erkennt
Anomalien, wenn die Latenz eines Flows ansteigt oder Pakete aufgrund von Überlast
oder Weiterleitungsfehlern verworfen werden. Jeder Flow hat einen Paketzähler,
der die Anzahl der Pakete repräsentiert, die in einem definierten
Aggregationsintervall in den ASIC eintreten. Die Aggregation kann im ASIC,
in der Switch-Software und in der Server-Software stattfinden.

Die Latenz misst die Gesamtzeit in Mikrosekunden, die ein Paket zwischen dem
Ingress- und Egress-Leaf-Switch für einen spezifischen Traffic-Flow benötigt.
Die Latenz wird sowohl für eingehenden als auch ausgehenden Traffic zwischen
einem Service-Endpoint und seinen Clients verfolgt. Ein Anomalie-Alert wird
ausgelöst, wenn die Performance-Metriken Latenz, Congestion oder Drops von
der Baseline abweichen.

---

## Methode 2: Atomic Counters – Gezielte Latenz-/Drop-Messung

Atomic Counters sind eine ACI-native Funktion für gezielte Messungen zwischen
zwei Endpoints:

- Funktioniert zwischen zwei konkreten Endpoints (IP-Paare)
- Liefert: Paketzähler, Drop-Zähler, Latenz (mit aktiviertem PTP)
- Wird über APIC GUI oder REST API konfiguriert
- Geeignet für gezielte Troubleshooting-Messungen, nicht für dauerhaftes
  Monitoring aller Flows

---

## Voraussetzung: PTP (Precision Time Protocol)

Das ist der kritische Punkt für genaue Latenzmessungen.

PTP wurde eingeführt, um das Latenz-Messfeature zu ermöglichen, das in Cisco
APIC Release 3.0(1) eingeführt wurde. Wenn PTP global aktiviert ist, werden
alle Leaf- und Spine-Switches als PTP Boundary Clocks konfiguriert. PTP wird
automatisch auf allen Fabric-Ports aktiviert, die vom ftag-Tree mit ID 0
verwendet werden.

Für genaue Latenzmessungen in Multi-Pod- und Multi-Site-Fabrics wird dringend
empfohlen, ein externes Gerät als PTP-Grandmaster zu verwenden. Nexus Dashboard
benötigt dabei nur Mikrosekunden-Genauigkeit – der externe PTP-Grandmaster kann
daher auch ein Switch, ein Router oder ein Linux-Server sein.

PTP aktivieren im APIC:
```
System → System Settings → Precision Time Protocol → Admin State: Enabled
```

---

## Was genau gemessen wird

| Metrik | Beschreibung |
|---|---|
| **Latenz** | Zeit in Mikrosekunden vom Ingress-Leaf bis Egress-Leaf für einen spezifischen Flow |
| **Congestion** | Netzwerk-Bandbreitenauslastung, QoS-Mechanismen, PFC- und ECN-Zähler |
| **Drops** | Anteil verworfener Pakete vs. übertragene Pakete (CRC, Kabel, Forwarding-Fehler) |
| **Performance Score** | Aggregierter Wert pro Conversation, hochgerechnet auf Endpoint-Ebene |

Ein Anomalie-Alert wird ausgelöst bei:
- Abweichung der Latenz vom historischen Durchschnitt
- Anstieg der Drop-Rate
- Anhaltende Congestion (PFC/ECN aktiv)

---

## Einschränkungen in großen Fabrics (~5.000 Ports)

| Einschränkung | Details |
|---|---|
| **Nicht alle Flows erfassbar** | Flow Telemetry erfasst Flows auf ASIC-Ebene – bei sehr hohem Traffic werden Flows nach Priorität/Sampling ausgewählt |
| **Aggregationsintervall** | Latenz wird als Durchschnitt über ein Intervall gemeldet, kein Echtzeit-Einzelpaket-Wert |
| **Hardware-Voraussetzung** | Flow Telemetry nur auf neueren Nexus-9000-ASICs (Cloud Scale ASIC: EX/FX/GX-Generation) |
| **PTP erforderlich** | Ohne PTP keine präzise Latenz – nur ungefähre Werte |
| **NDI-Lizenz** | Nexus Dashboard Insights ist eine separate, kostenpflichtige Lizenz |
| **Multi-Site Genauigkeit** | Für Multi-Pod/-Site: externer PTP-Grandmaster dringend empfohlen |

---

## Architektur für Latenzmessung in großen ACI-Fabrics

```
Server A (Endpoint)
     │
  Leaf 1 (Ingress)
     │  ← Zeitstempel beim Eintritt (PTP-synchronisiert)
  Spine
     │
  Leaf 2 (Egress)
     │  ← Zeitstempel beim Austritt (PTP-synchronisiert)
Server B (Endpoint)

Latenz = Egress-Timestamp − Ingress-Timestamp
         (Auflösung: Mikrosekunden)

         ↓ Flow Telemetry
Nexus Dashboard Insights
  → Latenz-Anzeige pro Flow
  → Anomalie-Alert bei Abweichung
  → Historische Trendanalyse
```

---

## Konfigurationsschritte (Kurzübersicht)

### 1. NTP konfigurieren (Basis für Zeitkorrelation)

```
APIC → Fabric → Fabric Policies → Date and Time Policy → NTP Servers
```

NTP ist Voraussetzung für die korrekte Korrelation von Log-Nachrichten, Faults,
Events und internen Atomic Counters. Nexus Dashboard Insights benötigt NTP,
um Informationen korrekt zu korrelieren und aussagekräftige Anomalien anzuzeigen.

### 2. PTP aktivieren (Voraussetzung für µs-genaue Latenz)

```
APIC → System → System Settings → Precision Time Protocol → Admin State: Enabled
```

Bei Multi-Pod / Multi-Site zusätzlich:
```
Externen PTP-Grandmaster konfigurieren (Switch, Router oder Linux-Server)
→ Benötigte Genauigkeit: Mikrosekunden-Ebene
```

### 3. Flow Telemetry aktivieren (in NDI)

```
Nexus Dashboard → Insights → Flow Analytics → Enable per Fabric
```

### 4. Atomic Counter konfigurieren (für gezielte Einzelmessungen)

```
APIC → Troubleshooting Wizard → Atomic Counter
  Source:      IP-Adresse Endpoint A
  Destination: IP-Adresse Endpoint B
  Protokoll:   TCP/UDP/ICMP (wählbar)
```

---

## Vergleich der Messmethoden

| Methode | Granularität | Dauerbetrieb | PTP nötig | Lizenz |
|---|---|---|---|---|
| **NDI Flow Telemetry** | Pro Flow (5-Tupel) | ✅ Ja | ✅ Ja | NDI-Lizenz |
| **Atomic Counters** | Pro IP-Paar | ⚠️ Manuell | ✅ Ja | Basis-APIC |
| **APIC REST API** (eqptIngrTotal) | Pro Interface | ✅ Ja | ❌ Nein | Basis-APIC |
| **ELAM (Embedded Logic Analyzer)** | Einzelpaket | ❌ Nein | ❌ Nein | Basis-APIC |

---

## ELAM – Für Einzelpaket-Diagnose

ELAM (Embedded Logic Analyzer Module) ist ein spezielles Diagnose-Tool für
Einzelpakete, das direkt im ASIC des Leaf-Switches arbeitet:

```
# ACI ELAM aktivieren (Beispiel für Nexus 9300 EX)
module-1# trigger reset
module-1# trigger init in-select 13 out-select 0
module-1# set outer ipv4 src_ip 10.1.1.1 dst_ip 10.2.2.2
module-1# start
# → Warten bis Paket getriggert wird
module-1# report
```

ELAM liefert:
- Exakte Forwarding-Entscheidung für ein einzelnes Paket
- VXLAN-Encapsulation-Details
- Drop-Ursache wenn Paket verworfen wurde
- **Keine Latenz** – aber vollständige Paketpfad-Analyse

NDI integriert ELAM-Triggering grafisch:
```
Nexus Dashboard → Insights → Troubleshoot → ELAM (aktiver Flow erforderlich)
```

---

## Fazit: Wann welche Methode?

| Anwendungsfall | Empfohlene Methode |
|---|---|
| Dauerhaftes Latenz-Monitoring aller Flows | NDI Flow Telemetry + PTP |
| Latenz-Anomalie-Alerting | NDI Insights (automatisch) |
| Gezieltes Troubleshooting zwischen 2 Hosts | Atomic Counters |
| Paketpfad-Analyse (wird das Paket überhaupt weitergeleitet?) | ELAM |
| Schnelle Baseline ohne NDI-Lizenz | APIC REST API (Interface-Statistiken) |

> **Wichtig:** Die Latenz wird als **Ingress-Leaf bis Egress-Leaf** gemessen –
> nicht End-to-End inklusive Server-Stack (Kernel, Applikation). Für
> vollständige End-to-End-Latenzmessung sind zusätzliche Agenten auf den
> Endpunkten erforderlich (z. B. Cisco AppDynamics, Thousand Eyes oder
> Open-Source-Tools wie hping3 oder iperf3).

---

## Quellen

- Cisco NDI Flows für ACI (Release 6.5.x):
  https://www.cisco.com/c/en/us/td/docs/dcn/ndi/6x/articles-651/ndi-flows-aci.html
- Cisco NDI Analysis Hub für ACI (Release 6.5.x):
  https://www.cisco.com/c/en/us/td/docs/dcn/ndi/6x/articles-651/ndi-analysis-hub-aci.html
- Cisco ACI Latency and Precision Time Protocol:
  https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/kb/b_Cisco_ACI_Latency_and_Precision_Time_Protocol.html
- Cisco APIC System Management Configuration Guide – PTP (Release 4.2.x):
  https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/4-x/system-management-configuration/cisco-apic-system-management-configuration-guide-42x/m-precision-time-protocol.html
- Getting ACI Ready for Nexus Dashboard Insights:
  https://www.cisco.com/c/en/us/td/docs/dcn/whitepapers/getting-aci-ready-for-ndi.html
- Cisco Nexus Dashboard Insights FAQ:
  https://www.cisco.com/c/en/us/products/collateral/data-center-analytics/nexus-insights/q-and-a-c67-742795.html
