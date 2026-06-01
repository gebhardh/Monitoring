# Qbilon – wesentliche Produktmerkmale

---

## Überblick

Qbilon wurde 2019 in Augsburg gegründet und 2023 von Paessler AG übernommen.
Die Software fusioniert bisher isolierte Datenquellen automatisch zu einem
umfassenden und konsistenten Gesamtbild der IT-Landschaft.

Qbilon bietet vollständige Sichtbarkeit und aktuelle Architektur-Einblicke in
die IT-Landschaften großer Unternehmen durch nahtlose Datenintegration. Der hohe
Automatisierungsgrad und das effiziente Datenmanagement für On-Premises- und
Cloud-Systeme reduziert den manuellen Aufwand und verbessert die Datengenauigkeit
und -zuverlässigkeit.

- **Gründung:** 2019, Augsburg, Deutschland
- **Übernahme:** Mai 2023 durch Paessler AG
- **Mitarbeiter:** ca. 14 (Stand 2026)
- **Webseite:** https://www.qbilon.io
- **Kontakt:** Hermanstraße 5, 86150 Augsburg · +49 821 71 04 09 70

---

## Die vier Kernkomponenten der Qbilon Suite

Die Qbilon Suite arbeitet über vier Kernkomponenten: die **Consolidation Database**,
die **Analytics Engine**, die **Workflow Engine** und die **Monitoring Component**
auf Basis von Paessler PRTG.

---

### 1. Consolidation Database – Single Source of Truth

Die Consolidation Database bietet:

- Anpassbare Datenpipeline für den Import und die Konsolidierung hybrider IT-Daten
- Aktuelles und effizientes IT-Asset-Management
- Dynamisch generiertes Datenmodell, das sich an beliebige Asset-Daten anpasst
- Nahtlose Integration mit beliebigen Systemen ohne Einschränkungen

---

### 2. Analytics Engine – Sichtbarkeit und Abhängigkeiten

Die Analytics Engine ermöglicht:

- Vollständige Transparenz über die gesamte Hardware- und Software-Landschaft
- Schnelle Dokumentengenerierung und einfache Audit-Vorbereitung (z. B. NIS-2)
- Transformation komplexer Asset-Daten in klare Echtzeit-Abhängigkeitskarten
- Angepasste Inventarlisten und KPI-Reports für mühelose Analyse
- Klare Identifikation von Verantwortlichen je Asset

---

### 3. Workflow Engine – Automatisierung

Die Workflow Engine ermöglicht:

- Erstellung und Anpassung leistungsstarker Workflows, die automatisch Daten aus
  mehreren Quellen integrieren, konsolidieren und extrahieren
- Befüllung der CMDB und Sicherstellung der Konsistenz in Drittsystemen
- Integration von Docker-Tasks in Qbilon-Datenworkflows
- Freie Definition von Ausführungsfrequenz und -reihenfolge aller Tasks

---

### 4. Monitoring Component – PRTG-Integration

Die Monitoring Component (powered by Paessler PRTG) liefert:

- Echtzeit-Monitoring-Einblicke für bessere Entscheidungen
- Automatisiertes Monitoring-Coverage und Streamlining für PRTG-Sensoren
- Mehrkanaliges Alarm- und Benachrichtigungssystem
- Automatisierte Identifikation von Netzwerkgeräten und Systemen
- Automatische Sensor-Erstellung und -Entfernung für optimale Abdeckung

---

## Wichtigste Anwendungsfälle

| Anwendungsfall | Beschreibung |
|---|---|
| **Automated Inventory** | Automatische Erfassung aller IT/OT-Assets ohne manuelle Arbeit |
| **Network Monitoring** | Echtzeit-Überwachung integriert mit Paessler PRTG |
| **CMDB Reconciliation** | Automatischer Abgleich und Befüllung von ServiceNow CMDB |
| **SLA-Management** | Überwachung von Service Levels und Abhängigkeiten |
| **NIS-2-Compliance** | Dokumentation und Audit-Vorbereitung für Cyberrichtlinien |
| **Dependency Mapping** | Visualisierung aller Abhängigkeiten zwischen Assets |
| **IT/OT-Konvergenz** | Einheitliche Verwaltung von IT- und OT-Assets |

---

## Integrationen (Ready-to-use Connectors)

Qbilon integriert Daten aus verschiedenen Quellen und sorgt für eine einheitliche
Ansicht der IT-Assets sowie präzises Abhängigkeitsmapping, das zur Befüllung
anderer Systeme wie der ServiceNow CMDB genutzt werden kann.

| Connector | Kategorie |
|---|---|
| Amazon Web Services (AWS) | Cloud |
| Microsoft Azure | Cloud |
| Google Cloud Platform (GCP) | Cloud |
| Microsoft Entra ID | Identity |
| VMware vSphere | Virtualisierung |
| Kubernetes | Container |
| Paessler PRTG | Monitoring |
| ServiceNow | ITSM / CMDB |
| Darktrace | Security / NDR |
| CSV Files | Datenimport |
| Webhooks | Integration |
| LUY | EAM |

---

## Technische Besonderheiten

### Automatische Datenfusion

Qbilon erfasst automatisch und über Bereichsgrenzen hinweg alle Daten über
IT-Assets – von IT-Infrastruktur-Bausteinen wie Servern über Anwendungen und
Dienste bis hin zu Produkten und Zuständigkeiten. Die aus verschiedenen Quellen
kommenden Informationen werden zu einem umfassenden und stimmigen Live-Bild der
IT-Landschaft fusioniert, sodass alle bestehenden Abhängigkeiten erstmals sichtbar
werden.

### Performance (ab Version 1.17)

Ab Version 1.17 wurden Merger- und Linker-Funktionen um **60 % beschleunigt**
sowie der Speicherbedarf und die Verarbeitungszeit bei Linking- und
Merging-Operationen großer Datensätze signifikant reduziert. Datenbankprozesse
wurden zusätzlich durch verbesserte Housekeeping-Routinen optimiert.

---

## Deployment und Preise

| Option | Details |
|---|---|
| **Deployment** | On-Premises oder SaaS |
| **Setup** | In Minuten einsatzbereit, kein Scripting/Coding erforderlich |
| **Mindestlaufzeit** | 1 Jahr |
| **Einstiegspreis** | ab € 659/Jahr (Qbilon Suite) / ab € 4.499/Jahr (vollständig) |
| **Skalierung** | Anzahl der verwalteten Elemente jährlich flexibel anpassbar |
| **Free Trial** | Verfügbar unter https://www.qbilon.io/request-trial/ |

---

## Einordnung im Monitoring-Kontext

Qbilon positioniert sich als **ITAM/EAM-Plattform mit integriertem Monitoring** –
kein reines NDR- oder NetFlow-Tool, sondern eine übergreifende
Asset-Daten-Management-Plattform:

| Dimension | Qbilon | Cisco SNA | Telegraf + Grafana |
|---|---|---|---|
| Asset-Inventar | ✅ Kernfunktion | ❌ | ❌ |
| Abhängigkeitsmapping | ✅ Kernfunktion | ❌ | ❌ |
| CMDB-Befüllung (ServiceNow) | ✅ | ❌ | ❌ |
| Netzwerk-Monitoring | ✅ (via PRTG) | ✅ | ✅ |
| Bedrohungserkennung | ⚠️ (via Darktrace) | ✅ | ❌ |
| NetFlow-Analyse | ❌ | ✅ | ✅ |
| NIS-2-Compliance | ✅ | ❌ | ❌ |
| IT/OT-Konvergenz | ✅ | ⚠️ | ❌ |
| Cloud-Integration | ✅ AWS/Azure/GCP | ⚠️ | ⚠️ |

---

## Referenzkunden

- RheinEnergie
- Stadtwerke München (SWM)
- KTR Systems GmbH
- Hemmersbach
- soffico GmbH
- ParkRaum-Management PRM GmbH

---

## Alleinstellungsmerkmal gegenüber ServiceNow CSDM

### Kurze Antwort

Qbilon ist **kein Konkurrent** zur ServiceNow CSDM – sondern ein **Zulieferer**.
Das eigentliche Alleinstellungsmerkmal ist die **automatisierte Datenerfassung und
-konsolidierung aus heterogenen Quellen**, die ServiceNow selbst nicht leisten kann
oder zu deutlich höheren Kosten leistet.

---

### Das Kernproblem der ServiceNow CSDM

Das CSDM ist ein Framework und Leitfaden für die Servicemodellierung in der
ServiceNow CMDB. Es richtet Konfigurationselemente (CIs) und Services an der
Geschäftsstrategie aus und schafft eine gemeinsame Terminologie für Entscheidungen
zu Strategie, Planung, Fehlerbehebung und Changes.

Das CSDM definiert damit **wie** Daten strukturiert sein sollen – aber **nicht**,
wie sie automatisch und vollständig **befüllt** werden. Genau hier liegt das Problem:

- Multi-Cloud-Komplexität macht es schwierig, Infrastruktur, Verantwortlichkeiten
  und Kosten über diverse Plattformen hinweg zu tracken
- CMDB-Genauigkeit – konsistente, genaue Daten zu Ownership, Kosten und Services –
  ist für effektives ITSM entscheidend
- Häufige Ressourcenänderungen überlasten manuelle Updates und stören ITSM durch
  veraltete Abhängigkeiten

---

### Qbilon als automatisierter CMDB-Zulieferer

Qbilon reduziert die CMDB-Management-Kosten um bis zu **90 % im Vergleich zu
ServiceNow Discovery**. Für ServiceNow-Nutzer bietet Qbilon eine Out-of-the-Box-Lösung
mit automatisierten Mappings und vorkonfigurierten Einstellungen.

| Aufgabe | ServiceNow CSDM allein | Qbilon + ServiceNow CSDM |
|---|---|---|
| **Datenstruktur definieren** | ✅ Kernfunktion | ✅ übernimmt CSDM-Struktur |
| **Daten automatisch erfassen** | ⚠️ nur via teurem ServiceNow Discovery | ✅ Multi-Source automatisch |
| **Multi-Cloud konsolidieren** | ⚠️ aufwendig, teuer | ✅ AWS + Azure + GCP + vSphere |
| **Duplikate erkennen/mergen** | ❌ manuell oder Discovery-MID | ✅ regelbasierter Merger |
| **Abhängigkeiten visualisieren** | ⚠️ Service Mapping (teuer) | ✅ automatisch, graphbasiert |
| **CMDB aktuell halten** | ⚠️ Discovery-Intervalle, teuer | ✅ alle 30 Min., günstig |
| **OT-Assets erfassen** | ❌ nicht vorgesehen | ✅ IT + OT in einem Modell |
| **Netzwerk-Monitoring** | ❌ kein Bestandteil | ✅ via PRTG-Integration |

---

### Praxisbeispiel: RheinEnergie AG (50.000 IT-Assets)

RheinEnergie migrierte Infrastruktur und Workloads in eine Multi-Cloud-Umgebung
(AWS, Azure, GCP). ServiceNow benötigte konsistente Updates für Assets, Owner,
Kostenstellen und Services. Self-Service-Provisioning verursachte hohe
Ressourcenvolatilität, die manuelle Updates unmöglich machte.

**Lösung mit Qbilon:**
- Automatisierte Datenerfassung aus Cloud (AWS, Azure, GCP) und On-Premises (vSphere)
- Regelbasierte Verknüpfung von Assets mit Ownern und Services in ServiceNow
- Kontinuierliche Aktualisierung der Datenbank alle 30 Minuten

**Ergebnis:**
- 30.000+ Cloud- und 10.000+ On-Premises-Assets automatisch erfasst
- Vollständige Eliminierung manueller CMDB-Updates
- Self-updating Service Status Map für Management-Reporting

---

### Kostenperspektive

ServiceNow Discovery erfordert MID-Server-Infrastruktur, Discovery-Lizenzen und
erheblichen Konfigurationsaufwand. Qbilon ersetzt oder ergänzt diesen aufwendigen
Weg durch einen deutlich schlankeren, vorkonfigurierten Ansatz mit einer
Kostenersparnis von bis zu 90 %.

---

### Fazit: Komplementär, nicht konkurrierend

Qbilon und ServiceNow CSDM erfüllen **unterschiedliche Aufgaben** in derselben
Wertschöpfungskette:

```
Heterogene Quellen          Qbilon                ServiceNow CSDM
(AWS, Azure, vSphere,  →  Automatische      →   Strukturierte,
 PRTG, Darktrace, ...)    Erfassung &            serviceorientierte
                          Konsolidierung         CMDB / ITSM-Prozesse
```

Das Alleinstellungsmerkmal von Qbilon ist die **automatisierte, günstige und
quellenübergreifende Datenbefüllung und -aktualisierung**, die ServiceNow aus
eigener Kraft entweder gar nicht oder nur zu deutlich höheren Kosten leisten kann.

---

## Quellen

- Qbilon Produktwebseite: https://www.qbilon.io
- Qbilon Suite Details: https://www.qbilon.io/qbilon-suite/
- Qbilon Platform: https://www.qbilon.io/qbilon-platform/
- Connector-Liste: https://www.qbilon.io/connectors/
- Paessler + Qbilon Case Study (KTR Systems):
  https://www.cbinsights.com/company/qbilon
- SourceForge Reviews 2026: https://sourceforge.net/software/product/Qbilon/
- GoodFirms Reviews 2026: https://www.goodfirms.co/software/qbilon
