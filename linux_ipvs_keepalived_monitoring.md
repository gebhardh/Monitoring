# Linux Kernel-basiertes Load Balancing & Monitoring

---

## Frage 1: Wie funktioniert mit Linux ein kernelbasiertes Load Balancing?

## Kernel-basiertes Load Balancing unter Linux

Linux bietet mehrere Mechanismen direkt im Kernel für Load Balancing, ohne zusätzliche Userspace-Software.

---

### 1. IPVS (IP Virtual Server) — `netfilter`-Modul

Das mächtigste kernel-native Load Balancing. Teil des **LVS (Linux Virtual Server)** Projekts.

**Funktionsprinzip:**
```
Client → Virtual IP (VIP) → IPVS im Kernel → Real Server 1
                                            → Real Server 2
                                            → Real Server 3
```

**Einrichten mit `ipvsadm`:**
```bash
# Modul laden
modprobe ip_vs

# Virtuellen Service anlegen (TCP, Port 80, Round Robin)
ipvsadm -A -t 192.168.1.100:80 -s rr

# Real Server hinzufügen
ipvsadm -a -t 192.168.1.100:80 -r 10.0.0.1:80 -m  # Masquerading
ipvsadm -a -t 192.168.1.100:80 -r 10.0.0.2:80 -m
ipvsadm -a -t 192.168.1.100:80 -r 10.0.0.3:80 -m

# Status anzeigen
ipvsadm -L -n
```

**Scheduling-Algorithmen:**

| Kürzel | Algorithmus | Beschreibung |
|--------|-------------|--------------|
| `rr` | Round Robin | Gleichmäßig reihum |
| `wrr` | Weighted RR | Gewichtetes Round Robin |
| `lc` | Least Connection | Wenigste aktive Verbindungen |
| `wlc` | Weighted LC | Gewichtet + Least Connection |
| `sh` | Source Hash | Sticky Sessions per Client-IP |
| `dh` | Destination Hash | Selbe Ziel-IP immer gleicher Server |
| `lblc` | Locality-Based LC | Für Cache-Cluster |

**Forwarding-Modi:**
- **NAT** (`-m`): Kernel übersetzt Adressen — einfach, aber Bottleneck
- **DR** (`-g`, Direct Routing): Server antwortet direkt an Client — sehr performant
- **TUN** (`-i`, IP Tunneling): Server in anderem Netz erreichbar

---

### 2. eBPF / XDP — modernes Hochleistungs-LB

Seit Kernel 4.8+. Pakete werden **vor dem Netzwerk-Stack** verarbeitet — extrem schnell.

```c
// Vereinfachtes XDP-Beispiel (C)
SEC("xdp")
int xdp_load_balancer(struct xdp_md *ctx) {
    // Paket parsen
    // Backend aus Map wählen
    // IP/MAC umschreiben
    return XDP_TX;  // Paket direkt zurückschicken
}
```

```bash
# eBPF-Programm laden
ip link set dev eth0 xdp obj lb.o sec xdp

# Backend-Map befüllen
bpftool map update ...
```

**Wird eingesetzt von:** Cilium, Katran (Facebook), Cloudflare

---

### 3. Nftables / iptables mit Statistik-Modul

Einfaches LB direkt in der Firewall:

```bash
# Mit nftables (modern)
nft add table ip lb
nft add chain ip lb prerouting '{ type nat hook prerouting priority -100; }'

# Round Robin über 3 Server
nft add rule ip lb prerouting \
    tcp dport 80 counter \
    numgen random mod 3 vmap { \
        0 : dnat to 10.0.0.1, \
        1 : dnat to 10.0.0.2, \
        2 : dnat to 10.0.0.3  \
    }
```

```bash
# Mit iptables (klassisch) — statistic-Modul
iptables -t nat -A PREROUTING -p tcp --dport 80 \
    -m statistic --mode nth --every 3 --packet 0 \
    -j DNAT --to-destination 10.0.0.1:80

iptables -t nat -A PREROUTING -p tcp --dport 80 \
    -m statistic --mode nth --every 2 --packet 0 \
    -j DNAT --to-destination 10.0.0.2:80

iptables -t nat -A PREROUTING -p tcp --dport 80 \
    -j DNAT --to-destination 10.0.0.3:80
```

---

### 4. CPU-seitiges Load Balancing (SMP)

Der Kernel verteilt auch intern Last:

- **RPS** (Receive Packet Steering): Netzwerkpakete auf CPU-Kerne verteilen
- **RFS** (Receive Flow Steering): Pakete zum CPU mit dem zugehörigen Prozess leiten
- **XPS** (Transmit Packet Steering): Sendewarteschlangen pro CPU

```bash
# RPS aktivieren (alle CPUs für eth0)
echo ff > /sys/class/net/eth0/queues/rx-0/rps_cpus
```

---

### Vergleich der Ansätze

| Methode | Schicht | Performance | Komplexität | Health Checks |
|---------|---------|-------------|-------------|---------------|
| IPVS | L4 | ★★★★ | mittel | mit `keepalived` |
| XDP/eBPF | L2/L3 | ★★★★★ | hoch | manuell |
| nftables | L4 | ★★★ | gering | nein |
| iptables | L4 | ★★ | gering | nein |

> **Für Produktion** empfiehlt sich IPVS + `keepalived` (für HA und Health Checks) oder bei sehr hohem Durchsatz ein eBPF-basiertes System wie **Cilium**.

---

## Frage 2: Wie kann ich die Variante IPVS + `keepalived` monitoren?

## IPVS + Keepalived Monitoring

Es gibt mehrere Ebenen, die überwacht werden sollten: IPVS-Verbindungen, Keepalived-Status (VRRP) und die Real Server.

---

### 1. Basis-Monitoring mit Bordmitteln

**IPVS-Status live anzeigen:**
```bash
# Alle virtuellen Services und Real Server
ipvsadm -L -n

# Mit Verbindungszählern
ipvsadm -L -n --stats

# Mit Raten (Pakete/Bytes pro Sekunde)
ipvsadm -L -n --rate

# Aktive Verbindungen live verfolgen
watch -n1 'ipvsadm -L -n --stats'
```

**Beispielausgabe `--stats`:**
```
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port               Conns   InPkts  OutPkts  InBytes OutBytes
  -> RemoteAddress:Port
TCP  192.168.1.100:80               104523  1823401        0   2.1G        0
  -> 10.0.0.1:80                     35012   614230        0  720M        0
  -> 10.0.0.2:80                     34988   601842        0  698M        0
  -> 10.0.0.3:80                     34523   607329        0  712M        0
```

**Keepalived-Status:**
```bash
# Service-Status
systemctl status keepalived

# VRRP-State (MASTER / BACKUP)
journalctl -u keepalived -f

# Kernel-Logs für VRRP-Transitions
dmesg | grep -i vrrp

# Keepalived gibt Status per Signal aus
kill -USR1 $(pidof keepalived)   # Status in /tmp/keepalived.stats
cat /tmp/keepalived.stats
```

---

### 2. `/proc`-Schnittstellen direkt lesen

```bash
# IPVS-Verbindungstabelle (direkt aus dem Kernel)
cat /proc/net/ip_vs

# Statistiken pro Service
cat /proc/net/ip_vs_stats

# Erweiterte Stats
cat /proc/net/ip_vs_stats_percpu
```

---

### 3. Keepalived-eigenes Monitoring

**`keepalived.conf` — Logging & Notifications konfigurieren:**
```nginx
global_defs {
    notification_email {
        admin@example.com
    }
    notification_email_from keepalived@lb01.example.com
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id LB_NODE_01
    enable_syslog
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100

    notify_master "/etc/keepalived/notify.sh MASTER"
    notify_backup "/etc/keepalived/notify.sh BACKUP"
    notify_fault  "/etc/keepalived/notify.sh FAULT"
}
```

**Notify-Skript `/etc/keepalived/notify.sh`:**
```bash
#!/bin/bash
STATE=$1
HOSTNAME=$(hostname)
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

logger -t keepalived "[$TIMESTAMP] $HOSTNAME wechselt zu: $STATE"

# Optional: Webhook / Alert senden
curl -s -X POST https://hooks.slack.com/... \
  -d "{\"text\": \"⚠️ $HOSTNAME → $STATE at $TIMESTAMP\"}"
```

---

### 4. Prometheus + Grafana (empfohlen für Produktion)

**`keepalived_exporter` installieren:**
```bash
wget https://github.com/gen2brain/keepalived_exporter/releases/latest/download/keepalived_exporter_linux_amd64.tar.gz
tar xzf keepalived_exporter_linux_amd64.tar.gz
mv keepalived_exporter /usr/local/bin/

cat > /etc/systemd/system/keepalived-exporter.service <<EOF
[Unit]
Description=Keepalived Exporter
After=network.target

[Service]
ExecStart=/usr/local/bin/keepalived_exporter
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl enable --now keepalived-exporter
# Metrics auf: http://localhost:9165/metrics
```

**`node_exporter` mit IPVS-Collector:**
```bash
node_exporter --collector.ipvs
```

**Wichtige PromQL-Queries:**
```promql
# Aktive Verbindungen pro Backend
node_ipvs_backend_connections_active{local_address="192.168.1.100:80"}

# Verbindungsrate
rate(node_ipvs_backend_connections_total[1m])

# Keepalived VRRP State (1=MASTER, 2=BACKUP, 3=FAULT)
keepalived_vrrp_state

# Failover-Zähler
increase(keepalived_vrrp_transitions_total[1h])
```

**Alerting-Regeln:**
```yaml
groups:
  - name: ipvs_keepalived
    rules:
      - alert: BackendDown
        expr: node_ipvs_backend_weight == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "Backend {{ $labels.remote_address }} ist ausgefallen"

      - alert: KeepalivedNotMaster
        expr: keepalived_vrrp_state{instance="lb01"} != 1
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "lb01 ist nicht mehr MASTER"

      - alert: HighConnectionRate
        expr: rate(node_ipvs_connections_total[1m]) > 10000
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Hohe Verbindungsrate: {{ $value }}/s"

      - alert: UnbalancedLoad
        expr: |
          stddev(rate(node_ipvs_backend_connections_total[5m]))
          / avg(rate(node_ipvs_backend_connections_total[5m])) > 0.3
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Load ungleichmäßig verteilt (>30% Abweichung)"
```

---

### 5. Schnelles Diagnose-Skript

```bash
#!/bin/bash
# ipvs-status.sh — Schnellübersicht

echo "=== VRRP State ==="
grep -r "State" /tmp/keepalived.stats 2>/dev/null || \
    journalctl -u keepalived --no-pager -n5

echo ""
echo "=== Virtuelle Services ==="
ipvsadm -L -n --stats

echo ""
echo "=== Backend Gewichte ==="
ipvsadm -L -n | awk '/->/{printf "  %-20s Gewicht: %s\n", $2, $4}'

echo ""
echo "=== Aktive Verbindungen gesamt ==="
cat /proc/net/ip_vs | wc -l

echo ""
echo "=== Keepalived Prozesse ==="
ps aux | grep keepalived | grep -v grep
```

---

### Übersicht der Monitoring-Schichten

```
┌─────────────────────────────────────────┐
│           Grafana Dashboard             │  ← Visualisierung
├─────────────────────────────────────────┤
│            Prometheus                   │  ← Speicherung + Alerting
├──────────────────┬──────────────────────┤
│ keepalived_      │   node_exporter      │  ← Metriken sammeln
│ exporter :9165   │   --collector.ipvs   │
├──────────────────┴──────────────────────┤
│     Keepalived + IPVS (Kernel)          │  ← Datenquelle
└─────────────────────────────────────────┘
```

---

## Frage 3: Wie sieht die Lösung mit Telegraf aus?

## IPVS + Keepalived Monitoring mit Telegraf

---

### Architektur

```
┌─────────────────────────────────────────────┐
│         Keepalived + IPVS (Kernel)          │
└──────────┬──────────────────┬───────────────┘
           │                  │
    /proc/net/ip_vs     keepalived.stats /
    ip_vs_stats         notify-Skripte
           │                  │
┌──────────▼──────────────────▼───────────────┐
│              Telegraf Agent                 │
│  ┌──────────┐ ┌──────────┐ ┌─────────────┐ │
│  │  inputs. │ │ inputs.  │ │  inputs.    │ │
│  │   ipvs   │ │   exec   │ │  procstat   │ │
│  └──────────┘ └──────────┘ └─────────────┘ │
└──────────────────┬──────────────────────────┘
                   │
        ┌──────────▼──────────┐
        │   InfluxDB / Cloud  │
        └──────────┬──────────┘
                   │
        ┌──────────▼──────────┐
        │       Grafana       │
        └─────────────────────┘
```

---

### 1. Installation

```bash
curl -s https://repos.influxdata.com/influxdata-archive_compat.key | \
    gpg --dearmor > /etc/apt/trusted.gpg.d/influxdata.gpg

echo "deb https://repos.influxdata.com/debian stable main" \
    > /etc/apt/sources.list.d/influxdata.list

apt update && apt install -y telegraf

# Telegraf braucht Kernel-Zugriff auf IPVS
usermod -aG adm telegraf
```

---

### 2. IPVS Plugin konfigurieren

```toml
# /etc/telegraf/telegraf.d/ipvs.conf

[[inputs.ipvs]]
  ## Keine weiteren Optionen nötig — liest aus /proc/net/ip_vs*
  ## Benötigt Root oder CAP_NET_ADMIN
```

**Gelieferte Felder:**

| Measurement | Felder |
|---|---|
| `ipvs_virtual_server` | `connections`, `pkts_in`, `pkts_out`, `bytes_in`, `bytes_out` |
| `ipvs_real_server` | `connections`, `pkts_in`, `pkts_out`, `bytes_in`, `bytes_out`, `active_connections`, `inactive_connections` |

---

### 3. Keepalived-Status per `exec` Plugin

**Skript `/etc/telegraf/scripts/keepalived-state.sh`:**
```bash
#!/bin/bash
kill -USR1 $(pidof keepalived) 2>/dev/null
sleep 0.2

STATS_FILE="/tmp/keepalived.stats"

if [ ! -f "$STATS_FILE" ]; then
    echo "keepalived_state state=0i"
    exit 0
fi

while IFS= read -r line; do
    if [[ $line =~ ^"VRRP Instance: "(.*) ]]; then
        INSTANCE="${BASH_REMATCH[1]}"
    fi
    if [[ $line =~ ^"  State: "(.*) ]]; then
        RAW_STATE="${BASH_REMATCH[1]}"
        case "$RAW_STATE" in
            MASTER) STATE=1 ;;
            BACKUP) STATE=2 ;;
            FAULT)  STATE=3 ;;
            *)      STATE=0 ;;
        esac
        echo "keepalived_vrrp,instance=${INSTANCE} state=${STATE}i,state_name=\"${RAW_STATE}\""
    fi
    if [[ $line =~ ^"  Became master: "(.*) ]]; then
        echo "keepalived_vrrp,instance=${INSTANCE} became_master=${BASH_REMATCH[1]}i"
    fi
    if [[ $line =~ ^"  Released master: "(.*) ]]; then
        echo "keepalived_vrrp,instance=${INSTANCE} released_master=${BASH_REMATCH[1]}i"
    fi
done < "$STATS_FILE"
```

```bash
chmod +x /etc/telegraf/scripts/keepalived-state.sh
```

**Telegraf-Konfiguration:**
```toml
# /etc/telegraf/telegraf.d/keepalived.conf

[[inputs.exec]]
  commands = ["/etc/telegraf/scripts/keepalived-state.sh"]
  timeout = "5s"
  data_format = "influx"
  interval = "10s"
```

---

### 4. Keepalived-Prozess überwachen

```toml
[[inputs.procstat]]
  pattern = "keepalived"
  pid_finder = "native"
```

---

### 5. Systemd-Service-Status

```toml
[[inputs.systemd_units]]
  unittype = "service"
  pattern = "keepalived.service"
```

---

### 6. Vollständige Telegraf-Konfiguration

```toml
# /etc/telegraf/telegraf.conf

[global_tags]
  host = "lb01"
  role = "loadbalancer"

[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  flush_interval = "10s"
  hostname = ""
  omit_hostname = false

# ── Output ──────────────────────────────────────────────
[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"]
  token = "dein-influx-token"
  organization = "myorg"
  bucket = "loadbalancer"

# ── IPVS ────────────────────────────────────────────────
[[inputs.ipvs]]

# ── Keepalived VRRP State ────────────────────────────────
[[inputs.exec]]
  commands = ["/etc/telegraf/scripts/keepalived-state.sh"]
  timeout = "5s"
  data_format = "influx"
  interval = "10s"

# ── Keepalived Prozess ───────────────────────────────────
[[inputs.procstat]]
  pattern = "keepalived"
  pid_finder = "native"

# ── Systemd Service ─────────────────────────────────────
[[inputs.systemd_units]]
  pattern = "keepalived.service"

# ── Netzwerk-Interface ───────────────────────────────────
[[inputs.net]]
  interfaces = ["eth0"]
  ignore_protocol_stats = false

# ── System allgemein ─────────────────────────────────────
[[inputs.cpu]]
  percpu = false
  totalcpu = true

[[inputs.mem]]

[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs"]
```

---

### 7. Telegraf Berechtigungen setzen

```bash
# Telegraf braucht Zugriff auf IPVS-Kernel-Strukturen
setcap cap_net_admin+ep /usr/bin/telegraf

# sudoers für keepalived stats
echo "telegraf ALL=(ALL) NOPASSWD: /bin/kill -USR1 *" \
    >> /etc/sudoers.d/telegraf

systemctl enable --now telegraf
journalctl -u telegraf -f
```

---

### 8. Testen

```bash
# Plugin-Lauf testen
telegraf --test --input-filter ipvs

# Keepalived-Skript direkt testen
/etc/telegraf/scripts/keepalived-state.sh

# Erwartete Ausgabe:
# keepalived_vrrp,instance=VI_1 state=1i,state_name="MASTER"
# keepalived_vrrp,instance=VI_1 became_master=3i
```

---

### 9. Grafana Dashboard — wichtige Flux-Queries

```sql
-- Aktive Verbindungen pro Backend
from(bucket: "loadbalancer")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "ipvs_real_server")
  |> filter(fn: (r) => r._field == "active_connections")
  |> aggregateWindow(every: 30s, fn: mean)

-- VRRP Failover-Ereignisse
from(bucket: "loadbalancer")
  |> range(start: -24h)
  |> filter(fn: (r) => r._measurement == "keepalived_vrrp")
  |> filter(fn: (r) => r._field == "state")
  |> difference()
  |> filter(fn: (r) => r._value != 0)

-- Traffic-Rate pro Backend
from(bucket: "loadbalancer")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "ipvs_real_server")
  |> filter(fn: (r) => r._field == "bytes_in")
  |> derivative(unit: 1s, nonNegative: true)
```

---

### Zusammenfassung der Telegraf-Plugins

| Plugin | Überwacht | Intervall |
|--------|-----------|-----------|
| `inputs.ipvs` | Verbindungen, Traffic pro VIP/Backend | 10s |
| `inputs.exec` | VRRP State (MASTER/BACKUP/FAULT) | 10s |
| `inputs.procstat` | Keepalived CPU/RAM | 10s |
| `inputs.systemd_units` | Service up/down | 30s |
| `inputs.net` | Interface-Traffic | 10s |
