# Alternativen zu Cisco SNA – Kommerziell und Open Source

---

## Kategorie 1: Kommerzielle NDR-Lösungen

### 1. Vectra AI

Vectra AI gehört zu den führenden NDR-Lösungen 2026 und setzt auf KI-gestützte
Angriffserkennung. Der Fokus liegt auf der Erkennung von Lateral Movement,
C&C-Kommunikation und Insider-Bedrohungen in Echtzeit. Besonderheit: Priorisierung
von Alerts nach tatsächlichem Angriffsrisiko statt nach Anomalie-Score.

- **Ansatz:** KI/ML-basierte Verhaltensanalyse
- **Stärken:** Attack-Signal-Intelligence, hohe Alert-Präzision, Cloud-native
- **Schwächen:** Kein natives NetFlow-Monitoring, höherer Preis
- **Ideal für:** Organisationen ohne Cisco-Infrastruktur, die maximale KI-Erkennung suchen

---

### 2. ExtraHop Reveal(x)

ExtraHop ist ideal für Organisationen, die hochperformante Netzwerksichtbarkeit
sowohl für Security- als auch IT-Operationen benötigen, mit besonderem Fokus auf
die Erkennung von Bedrohungen in verschlüsseltem Traffic und forensischer Detailtiefe
für Incident Investigation und Threat Hunting. Technisch basiert ExtraHop auf Deep
Packet Inspection (DPI) statt reinem NetFlow – damit deutlich granularer als SNA,
aber hardwareintensiver.

- **Ansatz:** Wire-Data-Analyse (DPI), ML-basierte Anomalie-Erkennung
- **Stärken:** L7-Sichtbarkeit, Echtzeit-Forensik, sehr gute UX
- **Schwächen:** Hoher Hardware-Bedarf (Sensoren), teuer bei großen Umgebungen
- **Ideal für:** SOCs mit Fokus auf Forensik und Incident Response

---

### 3. Darktrace NDR

Darktrace ist Pionier im Bereich KI-getriebener NDR mit unüberwachtem Machine
Learning, das normale Verhaltensmuster lernt und Anomalien ohne vordefinierte Regeln
erkennt. Die Plattform bietet breite Abdeckung über Netzwerk, Cloud, E-Mail und
Identität. Hinweis: Übernahme durch Thoma Bravo (angekündigt 2024) schafft
Beschaffungsüberlegungen.

- **Ansatz:** Unsupervised ML, Self-Learning AI
- **Stärken:** Keine Regelwartung, breite Coverage (Netzwerk + Cloud + E-Mail)
- **Schwächen:** Hohe False-Positive-Rate in dynamischen Umgebungen, Vendor Lock-in
- **Ideal für:** Organisationen ohne dediziertes Security-Team

---

### 4. Corelight Open NDR Platform

Corelight transformiert Netzwerktraffic in hochstrukturierte, verwertbare Daten.
Gegründet von den Entwicklern der Open-Source-Projekte Zeek und Suricata, fungieren
Corelight-Appliances als Enterprise-Sensoren, die die detailliertesten Netzwerk-Logs
der Branche erzeugen. Der Open NDR-Ansatz verhindert Vendor Lock-in: strukturierte
Zeek-Logs werden direkt in SIEM oder Data Lake gestreamt.

- **Ansatz:** Zeek + Suricata Engine, Open NDR
- **Stärken:** Höchste Log-Qualität, kein Vendor Lock-in, SIEM-Integration
- **Schwächen:** Kein fertiges All-in-One-Dashboard, setzt erfahrene Analysten voraus
- **Ideal für:** Reife SOCs, Threat Hunting, Engineering-Teams

---

### 5. Palo Alto Cortex NDR

Cortex NDR ist tief in Palo Alto Networks' XDR-Strategie eingebunden und bringt
leistungsstarke Netzwerkanalytik in eine einheitliche Security-Operations-Plattform.

- **Ansatz:** XDR-integriert, ML-basiert
- **Stärken:** Nahtlose Integration mit Palo Alto Firewalls, XSOAR, Prisma Cloud
- **Schwächen:** Nur sinnvoll bei bestehender Palo Alto Umgebung
- **Ideal für:** Organisationen mit Palo Alto-Infrastruktur

---

### 6. Arista NDR (früher Awake Security)

Arista NDR bietet verhaltensbasierte Anomalie-Erkennung und nativen Cloud-Support,
mit enger Integration in das Arista-Netzwerk-Ökosystem.

- **Ansatz:** Verhaltensanalytik, ML
- **Stärken:** Gute Integration mit Arista EOS/CloudVision
- **Schwächen:** Weniger bekannt, kleinere Community
- **Ideal für:** Arista-Netzwerkumgebungen

---

### 7. Trend Micro Vision One NDR

Kombiniert IPS-Funktionalität mit NDR-Analytik, gut integriert in das Trend Micro
XDR-Ökosystem.

- **Ansatz:** XDR-integriert, signatur- und verhaltensbasiert
- **Stärken:** Breites Ökosystem, bekannte Threat Intelligence
- **Ideal für:** Bestehende Trend Micro Kunden

---

### 8. SolarWinds NetFlow Traffic Analyzer (NTA)

SolarWinds NTA ist ein bewährtes Werkzeug mit Fokus auf Performance-Monitoring.
Daten können nach Verbindungen und Endpunkten sortiert und visualisiert werden.
Es verbindet sich mit dem SolarWinds Network Performance Monitor für kontinuierliches
Traffic-Tracking neben Gerätezustandsprüfungen.

- **Ansatz:** NetFlow-basiertes Performance-Monitoring
- **Stärken:** Einfache Bedienung, gute NPM-Integration, günstig
- **Schwächen:** Kein ML/AI, kein Security-Fokus, keine Bedrohungserkennung
- **Ideal für:** Netzwerk-Performance-Monitoring ohne Security-Anforderung

---

## Kategorie 2: Open-Source-Lösungen

### 9. Zeek (früher Bro)

Das akademisch-industrielle Fundament des NDR-Bereichs. Zeek analysiert
Netzwerktraffic passiv und erzeugt strukturierte Logs (HTTP, DNS, SSL,
Verbindungsdaten). Grundlage für Corelight (kommerziell). Läuft auf Linux,
kann auf Mirror-Ports oder Network Taps betrieben werden.

- **Ansatz:** Protokoll-Analyse, strukturierte Log-Erzeugung
- **Stärken:** Extrem detaillierte L7-Logs, flexibel skriptbar (Zeek-Scripting)
- **Schwächen:** Kein fertiges Dashboard, kein Alerting out-of-the-box
- **Kombination:** Zeek + Elastic Stack + Kibana für vollständiges Setup
- **Lizenz:** Open Source (BSD)
- **URL:** https://zeek.org

---

### 10. Suricata

Suricata ist ein hochperformantes Network IDS/IPS und Network Security Monitoring
(NSM) Engine – vollständig Open Source. Ergänzt Zeek ideal: Suricata liefert
signaturbasierte Erkennung (IDS/IPS), Zeek verhaltensbasierte Logs. Viele SOCs
betreiben beide parallel.

- **Ansatz:** Signaturbasiertes IDS/IPS + NSM
- **Stärken:** Hohe Performance, Multi-Threading, Regelwerke (ET Open, SNORT-Regeln)
- **Schwächen:** Keine Verhaltensanalyse, Regelwartung nötig
- **Kombination:** Suricata + Zeek + Elastic/Splunk (Security Onion)
- **Lizenz:** Open Source (GPL)
- **URL:** https://suricata.io

---

### 11. Arkime (früher Moloch)

Vollständige Paketerfassung (Full Packet Capture) mit Indexierung und Suche über
Elasticsearch. Ideal für Forensik: Jedes Netzwerkpaket wird gespeichert und ist
durchsuchbar.

- **Ansatz:** Full Packet Capture + Indexierung
- **Stärken:** Maximale forensische Tiefe, Elasticsearch-basierte Suche, PCAP-Export
- **Schwächen:** Sehr hoher Speicherbedarf, kein ML/Anomalie-Erkennung
- **Kombination:** Arkime + Zeek + Suricata (Security Onion)
- **Lizenz:** Open Source (Apache 2.0)
- **URL:** https://arkime.com

---

### 12. ntopng + nProbe

ntopng ist ein webbasiertes Traffic-Analyse-Tool für Netzwerküberwachung auf
Basis von Flow-Daten. nProbe fungiert als NetFlow/IPFIX-Collector, der
Flow-Records von Exportern empfängt und an ntopng weiterleitet. Zusammen bilden
sie ein sehr flexibles Analyse-Paket mit Geo-IP, App-Erkennung und
Anomalie-Erkennung.

- **Ansatz:** NetFlow/IPFIX-Analyse + Paketerfassung
- **Stärken:** Breite Protokollunterstützung, Web-UI, Anomalie-basiertes IDS
- **Community-Version:** Kostenlos (eingeschränkt)
- **Enterprise-Version:** Kostenpflichtig (kostenlos für NGOs/Bildung)
- **Lizenz:** Dual (Open Source Community + kommerziell)
- **URL:** https://www.ntop.org

---

### 13. nfdump + nfsen

Klassischer Open-Source-NetFlow-Stack: nfdump sammelt und speichert Flows
effizient im Binärformat, nfsen visualisiert sie über ein Webinterface mit
RRD-Graphen. Einfach, stabil, ressourcenschonend – aber ohne ML-Analytik.

- **Ansatz:** NetFlow-Erfassung und -Visualisierung
- **Stärken:** Sehr ressourcenschonend, stabil, gut dokumentiert
- **Schwächen:** Veraltetes UI (nfsen), keine Bedrohungserkennung, kein ML
- **Ideal für:** Einfaches NetFlow-Archiv und Baseline-Analyse
- **Lizenz:** Open Source (BSD/GPL)
- **URL:** https://github.com/phaag/nfdump

---

### 14. Wazuh

Primär ein SIEM/Host-IDS, aber mit Netzwerk-Monitoring-Fähigkeiten durch
Integration mit Zeek und Suricata. Vollständig Open Source mit aktiver Community.
Gut geeignet als kostengünstige Gesamtlösung für kleinere Umgebungen.

- **Ansatz:** SIEM + HIDS + Netzwerk-Integration
- **Stärken:** Vollständig Open Source, aktive Community, gute SIEM-Funktionen
- **Schwächen:** Kein natives NDR, Netzwerk-Sichtbarkeit nur über Plugins
- **Kombination:** Wazuh + Zeek + Suricata für vollständiges NDR
- **Lizenz:** Open Source (GPL)
- **URL:** https://wazuh.com

---

### 15. Security Onion (Open-Source-Bundle)

Security Onion ist keine einzelne Anwendung, sondern eine vorkonfigurierte
Linux-Distribution, die Zeek, Suricata, Arkime, Elastic Stack und weitere Tools
zu einem vollständigen NDR-Stack zusammenfasst.

- **Enthält:** Zeek, Suricata, Arkime, Elastic/Kibana, Wazuh, CyberChef
- **Stärken:** Sofort einsatzbereit, gut dokumentiert, aktive Community
- **Schwächen:** Ressourcenintensiv, skaliert nicht auf > 10 Gbps ohne erhebliche Hardware
- **Ideal für:** Mittlere Umgebungen, SOC-Aufbau ohne Budget für kommerzielle Tools
- **Lizenz:** Open Source
- **URL:** https://securityonionsolutions.com

---

## Vergleichsmatrix

| Produkt | Typ | Ansatz | NetFlow | DPI | ML/AI | Skala | Kosten |
|---|---|---|---|---|---|---|---|
| **Cisco SNA** | Kommerziell | NetFlow/IPFIX | ✅✅ | ⚠️ ETA | ✅ | Sehr groß | Hoch |
| **Vectra AI** | Kommerziell | ML/KI | ⚠️ | ✅ | ✅✅ | Groß | Hoch |
| **ExtraHop Reveal(x)** | Kommerziell | DPI/Wire-Data | ⚠️ | ✅✅ | ✅ | Groß | Hoch |
| **Darktrace NDR** | Kommerziell | Unsupervised ML | ✅ | ✅ | ✅✅ | Groß | Hoch |
| **Corelight** | Kommerziell | Zeek/Suricata | ⚠️ | ✅✅ | ✅ | Groß | Mittel-hoch |
| **Palo Alto Cortex NDR** | Kommerziell | XDR-integriert | ✅ | ✅ | ✅ | Groß | Hoch |
| **Arista NDR** | Kommerziell | Verhaltensanalytik | ✅ | ✅ | ✅ | Mittel-groß | Mittel-hoch |
| **SolarWinds NTA** | Kommerziell | NetFlow | ✅✅ | ❌ | ❌ | Mittel | Mittel |
| **Zeek** | Open Source | Protokoll-Logs | ❌ | ✅✅ | ❌ | Groß | Kostenlos |
| **Suricata** | Open Source | IDS/IPS/NSM | ❌ | ✅✅ | ❌ | Groß | Kostenlos |
| **Arkime** | Open Source | Full PCAP | ❌ | ✅✅ | ❌ | Mittel | Kostenlos |
| **ntopng + nProbe** | Open/Kommerziell | NetFlow + Pakete | ✅✅ | ✅ | ⚠️ | Mittel | Frei/Kosten |
| **nfdump + nfsen** | Open Source | NetFlow | ✅✅ | ❌ | ❌ | Klein-Mittel | Kostenlos |
| **Wazuh** | Open Source | SIEM + HIDS | ⚠️ | ⚠️ | ⚠️ | Mittel | Kostenlos |
| **Security Onion** | Open Source | Bundle (Zeek+Suricata+Arkime) | ⚠️ | ✅✅ | ⚠️ | Mittel | Kostenlos |

Legende: ✅✅ = sehr gut, ✅ = gut, ⚠️ = eingeschränkt, ❌ = nicht vorhanden

---

## Empfehlung nach Szenario

| Szenario | Empfehlung |
|---|---|
| Cisco-Infrastruktur vorhanden | Cisco SNA (native NetFlow-Integration) |
| Maximale KI-Erkennung ohne Regelwartung | Darktrace oder Vectra AI |
| Forensik & Threat Hunting (reifes SOC) | Corelight oder ExtraHop Reveal(x) |
| Palo Alto Umgebung | Cortex NDR |
| Open Source, großes erfahrenes Team | Zeek + Suricata + Arkime + Elastic |
| Open Source, kleines Team, schneller Start | Security Onion |
| Open Source, NetFlow-Fokus | ntopng + nProbe oder nfdump |
| Hybrid: OS + SIEM | Wazuh + Zeek + Suricata |
| Nur NetFlow, kein Security-Fokus | SolarWinds NTA oder nfdump |

---

## Quellen

- Gartner Peer Insights – NDR Alternativen zu Cisco SNA:
  https://www.gartner.com/reviews/product/cisco-secure-network-analytics/alternatives
- CyberPress – Best NDR Solutions 2026:
  https://cyberpress.org/best-ndr-solutions/
- GBHackers – Top 10 NDR Solutions 2026:
  https://gbhackers.com/best-ndr-solutions/
- Deepak Gupta – Top 5 NDR Tools 2026:
  https://guptadeepak.com/tools/top-5-ndr-tools-2026/
- Omdia – NDR Market 2026:
  https://omdia.tech.informa.com/blogs/2026/may/network-detection-and-response-ndr-market-2026-navigating-xdr-disruption-platform-consolidation-and-ai-driven-renaissance
- Zeek Project: https://zeek.org
- Suricata: https://suricata.io
- Arkime: https://arkime.com
- ntop: https://www.ntop.org
- Security Onion: https://securityonionsolutions.com
- Wazuh: https://wazuh.com
