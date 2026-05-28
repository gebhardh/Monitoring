# ZTNA-Architektur – Metriken und Logs im Überblick

Eine ZTNA-Architektur besteht aus mehreren Schichten: Authentifizierung über einen
Identity Provider, Gerätebewertung (Device Posture Assessment), Policy-Auswertung
durch ein Control Plane und der eigentlichen verschlüsselten Session zwischen Nutzer
und Applikation. Jede dieser Schichten erzeugt eigene Metriken und Logs.

---

## Kategorie 1: Authentifizierungs-Metriken (Identity Plane)

Zu den wichtigsten Metriken im Authentifizierungsbereich gehören: Auth-Versuche,
MFA-Events, Device-Posture-Signale sowie Token-Ausstellungs-Latenz und
Token-Ausstellungs-Fehler. Diese sollten über IDP-Audit-Logs erfasst werden.

| Metrik | Beschreibung | Schwellenwert |
|---|---|---|
| `auth_attempts_total` | Gesamtzahl Anmeldeversuche | Baseline ± 3σ |
| `auth_failures_total` | Fehlgeschlagene Authentifizierungen | > 5/min → Alert |
| `mfa_challenge_rate` | Anteil MFA-pflichtiger Sessions | < 100 % → Policy-Problem |
| `mfa_failures_total` | Fehlgeschlagene MFA-Challenges | > 3 pro User → Sperren |
| `token_issuance_latency_ms` | Latenz der Token-Ausstellung | > 500 ms → Problem |
| `token_issuance_failures` | Fehlgeschlagene Token-Ausstellungen | > 0 → sofort untersuchen |
| `sso_session_duration_sec` | Sitzungsdauer je User | Ausreißer → Anomalie |

---

## Kategorie 2: Device Posture Metriken (Endpoint-Bewertung)

Beim ZTNA-Telemetrie-Modell werden Geräteinformationen, User-Logon-Informationen
und Security-Posture-Daten kontinuierlich vom Endpunkt an den Controller übertragen.
Basierend darauf werden Geräte mit Zero-Trust-Tags versehen, die in Echtzeit mit dem
Policy Enforcement Point synchronisiert werden.

| Metrik | Beschreibung |
|---|---|
| `device_posture_compliant` | Anzahl complianter Geräte (0/1 pro Gerät) |
| `device_posture_failures` | Geräte, die Posture-Check nicht bestehen |
| `posture_check_latency_ms` | Dauer der Gerätebewertung |
| `os_patch_compliance_rate` | Anteil Geräte mit aktuellem Patchstand |
| `antivirus_enabled_rate` | Anteil Geräte mit aktivem AV |
| `disk_encryption_rate` | Anteil Geräte mit Festplattenverschlüsselung |
| `certificate_expiry_days` | Verbleibende Gültigkeit der Client-Zertifikate |

> **Hinweis:** Ändert sich der Posture-Status eines Endpunkts während einer aktiven
> Session, werden alle laufenden ZTNA-Sessions sofort neu bewertet und bei
> Non-Compliance terminiert.

---

## Kategorie 3: Policy Enforcement Metriken

ZTNA-Lösungen erzeugen detaillierte Telemetrie über Applikationszugriffe,
einschließlich Nutzeridentität, Gerätestatus, Zugriffszeitpunkt und
Policy-Entscheidungen.

| Metrik | Beschreibung |
|---|---|
| `policy_allow_total` | Anzahl erlaubter Zugriffsanfragen |
| `policy_deny_total` | Anzahl abgelehnter Zugriffsanfragen |
| `policy_deny_rate` | Anteil Ablehnungen gesamt (%) |
| `policy_eval_latency_ms` | Auswertungszeit je Policy-Entscheidung |
| `access_by_application` | Zugriffe je Applikation (Top-N) |
| `access_by_user` | Zugriffe je Nutzer (Anomalie-Erkennung) |
| `access_by_location` | Zugriffe je Standort / Land |
| `lateral_movement_attempts` | Zugriffsversuche auf nicht-berechtigte Ressourcen |

---

## Kategorie 4: Session-Metriken

ZTNA-Lösungen überwachen kontinuierlich Nutzerverhalten, Gerätestatus und
Umgebungsfaktoren während der gesamten Session. Bei verdächtigen Aktivitäten –
z. B. plötzlicher Standortwechsel, ungewöhnliche Zugriffsmuster oder ein Abfall
des Device-Posture-Status – können automatisierte Antworten ausgelöst werden:
Step-up-Authentifizierung, reduzierte Zugriffsrechte oder sofortige
Session-Terminierung.

| Metrik | Beschreibung |
|---|---|
| `active_sessions_total` | Aktuell aktive Sessions |
| `session_duration_avg_sec` | Durchschnittliche Sitzungsdauer |
| `sessions_terminated_policy` | Sessions durch Policy-Änderung terminiert |
| `sessions_terminated_posture` | Sessions durch Posture-Verlust terminiert |
| `step_up_auth_events` | Ausgelöste Step-up-Authentifizierungen |
| `concurrent_sessions_per_user` | Gleichzeitige Sessions je Nutzer |
| `session_bandwidth_bytes` | Übertragenes Datenvolumen je Session |

---

## Kategorie 5: Gateway / Broker Metriken

Policy Enforcement Points – oft als Gateways oder Proxies implementiert – sind die
Durchsetzungsinstanzen, die auf Basis von Nutzeridentität, Gerätestatus und Kontext
entscheiden, ob ein Zugriff erlaubt oder verweigert wird.

| Metrik | Beschreibung |
|---|---|
| `gateway_cpu_usage_pct` | CPU-Auslastung des ZTNA-Gateways |
| `gateway_memory_usage_pct` | Memory-Auslastung |
| `gateway_throughput_bytes_sec` | Durchsatz in Bytes/s |
| `gateway_connection_count` | Aktive Verbindungen am Gateway |
| `gateway_tls_handshake_errors` | TLS-Fehler bei Verbindungsaufbau |
| `gateway_availability_pct` | Verfügbarkeit (SLA-Messung) |
| `tunnel_setup_latency_ms` | Aufbauzeit des verschlüsselten Tunnels |

---

## Kategorie 6: Log-Typen in der ZTNA-Architektur

| Log-Typ | Inhalt | Ziel |
|---|---|---|
| **Authentication Logs** | User, Timestamp, MFA-Status, IDP-Antwort, Erfolg/Fehler | SIEM |
| **Authorization Logs** | User, Ressource, Policy-Entscheidung (Allow/Deny), Grund | SIEM |
| **Session Logs** | Session-ID, Start/Ende, Bytes, App, Gerät, Standort | SIEM / SOAR |
| **Device Posture Logs** | Gerät, Compliance-Status, fehlgeschlagene Checks, Tags | MDM / SIEM |
| **Policy Change Logs** | Wer hat welche Policy wann geändert | Audit-Log |
| **Anomaly Logs** | Ungewöhnliche Zugriffsmuster, Impossible Travel, Brute Force | SIEM / SOC |
| **Certificate Logs** | Ausstellung, Widerruf, Ablauf von Client-Zertifikaten | PKI / SIEM |
| **Admin Audit Logs** | Konfigurationsänderungen, User-Management | Compliance |

---

## Kategorie 7: Anomalie-Indikatoren (Typische ZTNA-Alerting-Regeln)

| Alert | Bedingung | Priorität |
|---|---|---|
| **Impossible Travel** | Login aus zwei Ländern < 1h Abstand | Kritisch |
| **Brute Force** | > 5 Auth-Fehler in 60s vom selben User | Hoch |
| **Posture Drift** | Gerät verliert Compliance während aktiver Session | Hoch |
| **Unbekanntes Gerät** | Zugriff ohne registriertes Client-Zertifikat | Hoch |
| **Policy-Deny-Spike** | Plötzlicher Anstieg der Deny-Rate > 200 % | Mittel |
| **Off-Hours-Zugriff** | Zugriff außerhalb definierter Zeiten | Mittel |
| **Lateral Movement** | Zugriff auf nicht-berechtigte Applikationen | Kritisch |
| **Token-Missbrauch** | Token-Nutzung von anderer IP als Ausstellung | Kritisch |

---

## Integrationsarchitektur: ZTNA + Monitoring-Stack

```
Identity Provider (IdP)          Endpoint (MDM/EDR)
       │                                 │
       └──────── ZTNA Control Plane ─────┘
                        │
              ZTNA Gateway / Broker
                        │
          ┌─────────────┼─────────────┐
          │             │             │
        SIEM          SOAR      Observability
     (Splunk/        (XSOAR/    (Prometheus/
      Elastic)       Sentinel)   Grafana)
          │
        SOC / Alert
```

---

## Vergleich: Was ZTNA zusätzlich zu klassischem Netz-Monitoring liefert

| Bereich | Klassisches Netz-Monitoring | ZTNA zusätzlich |
|---|---|---|
| Identität | ❌ Keine | ✅ User, Rolle, MFA-Status |
| Gerätestatus | ❌ Keine | ✅ Posture, Patch, AV, Zertifikat |
| Applikationszugriff | ❌ Nur IP/Port | ✅ Pro App, pro Session |
| Policy-Entscheidungen | ❌ Keine | ✅ Allow/Deny mit Begründung |
| Continuous Verification | ❌ Nein | ✅ Laufende Session-Bewertung |
| Lateral Movement | ⚠️ Schwer erkennbar | ✅ Per Design verhindert + geloggt |

---

## Empfohlene Open-Source-Tools für ZTNA-Monitoring

| Tool | Einsatz |
|---|---|
| **Prometheus** | Metriken-Erfassung (Gateway, Auth, Sessions) |
| **Grafana** | Dashboards für alle ZTNA-Metriken |
| **Loki** | Log-Aggregation (Auth-, Session-, Policy-Logs) |
| **OpenTelemetry** | Standardisierte Telemetrie-Übertragung |
| **Elastic SIEM** | Anomalie-Erkennung, Alerting, SOC-Integration |
| **Falco** | Runtime-Anomalie-Erkennung auf Endpunkten |

---

## Quellen

- Fortinet ZTNA Telemetry & Tags:
  https://docs.fortinet.com/document/fortigate/7.0.0/ztna-architecture/145655/ztna-telemetry-tags-and-policy-enforcement
- Palo Alto Networks – What is ZTNA:
  https://www.paloaltonetworks.com/cyberpedia/what-is-zero-trust-network-access-ztna
- Exabeam – ZTNA Solutions Key Capabilities:
  https://www.exabeam.com/explainers/zero-trust/ztna-solutions-key-capabilities-and-9-options-to-know/
- DevSecOps School – ZTNA Guide 2026:
  https://devsecopsschool.com/blog/ztna/
- Fortinet – Posture Check Verification:
  https://docs.fortinet.com/document/fortigate/7.0.0/new-features/580880/posture-check-verification-for-active-ztna-proxy-session-7-0-2
