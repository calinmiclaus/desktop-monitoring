
## **Monitoring that I use for my daily driver (Kubuntu)** ##
This monitoring setup uses Docker containers to collect system metrics and logs, visualize them in Grafana, and send alerts if something goes wrong.

### **Containers and Their Roles** ###
node_exporter──►Collects Linux host metrics──►CPU usage, memory usage, disk I/O, network traffic, filesystem stats, per-process stats
Prometheus──►Metrics database and query engine──►Scrapes metrics from node_exporter (and Prometheus itself), stores time-series metrics
Alertmanager──►Handles alerts from Prometheus──►Receives alert events (CPU spikes, memory usage, disk IO), routes them to destinations like email, Slack, webhooks
Loki──►Log storage and query engine──►Stores logs shipped from Promtail, enables querying and visualization of logs
Promtail──►Log collector──►Reads systemd journal logs from the host, sends them to Loki, adds metadata (like systemd unit)
Grafana──►Dashboard and visualization platform──►Visualizes metrics and logs, provides alerting interface, dashboards, and panels

node_exporter ──► Prometheus ──► Grafana
systemd journal ──► Promtail ──► Loki ──► Grafana
Prometheus ──► Alertmanager ──► Email/Slack/Webhooks

node_exporter → Prometheus: Prometheus scrapes metrics over HTTP (port 9100).

Prometheus → Alertmanager: Prometheus evaluates alert rules and pushes alerts to Alertmanager via HTTP (port 9093).

systemd journal → Promtail → Loki: Promtail reads logs from /var/log/journal or /run/log/journal and pushes them to Loki (port 3100).

Grafana → Prometheus/Loki: Grafana queries Prometheus for metrics and Loki for logs to display dashboards and panels.

### **What You Can See** ###

CPU usage: Per-core or overall load, percentage usage.

Memory usage: Total, free, cached, buffers.

Disk I/O: Read/write rates, IOPS, disk latency.

Network traffic: Throughput, errors, dropped packets.

Filesystem usage: Free space, used space per mount point.

Processes: Per-process CPU/memory usage (if collected).

Systemd logs: All journal logs, filtered by unit (e.g., sshd.service), priority, or time.


### **Alerts and Notification** ###

**You have Prometheus alerts configured for:**

High CPU usage: >80% for 2 minutes.

High memory usage: >80% for 2 minutes.

High disk I/O: >50% utilization over 1 minute.

**Alerts are sent to Alertmanager, which can be configured to:**

Send emails.

Send Slack or Telegram notifications.

Trigger webhooks for custom actions.

Group and silence alerts based on severity or other labels.

Grafana can also define alert rules independently on metrics panels.



### **Further Configurable Options** ###

**Node_exporter:**
Expose additional metrics like sensors, temperatures, or process stats.
Run with --collector.<name> options to enable specific collectors.

**Prometheus:**
Add more scrape_configs to monitor other hosts or containers.
Define new alert rules in alerts.yml.

**Alertmanager:**
Configure multiple routes for different severity levels.
Add receivers for email, Slack, webhook, PagerDuty, or Opsgenie.
Promtail / Loki:
Add custom labels to logs.
Parse structured logs (JSON, syslog).
Set retention policies in Loki to limit storage.

**Grafana:**
Create dashboards, panels, and queries.
Define alerts on any Prometheus metric or Loki log pattern.
Use variables for dynamic dashboards (e.g., filter by hostname or systemd unit).
Set up folders, user roles, and authentication for team usage.


### **Access Points** ###
Component	URL / Port	Notes
Grafana UI	http://localhost:3000	Login: admin/admin
Prometheus UI	http://localhost:9090	Query metrics, test alert rules
Alertmanager UI	http://localhost:9093	See active/firing alerts
Loki API	http://localhost:3100	Queried by Grafana for logs

### **To make it run** ###

**Start everything:**
docker compose up -d

**Add data sources and dashboards**
In Grafana → Connections → Add data source:
Add Prometheus with URL http://prometheus:9090.
Add Loki with URL http://loki:3100.
Import dashboards (Node Exporter Full 1860, Loki Systemd Journal 13639)
