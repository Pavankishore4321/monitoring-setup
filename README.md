# monitoring-observability-setup

Monitoring and observability configuration files built from real production
experience at **Incresol Software Services**. Used to monitor AspTax, Node App
high-availability setup, and Spring Boot multi-client deployments.

---

## Repository Structure

```
monitoring-observability-setup/
├── prometheus/
│   ├── prometheus.yml         # Scrape configs for all services
│   └── alert-rules.yml        # Alerting rules — CPU, Disk, Service Down, MongoDB
├── grafana/
│   └── dashboard.json         # AspTax infrastructure dashboard (importable)
├── elk/
│   ├── logstash.conf          # Parses Nginx + Spring Boot logs
│   └── filebeat.yml           # Ships logs from servers to Logstash
├── datadog/
│   └── datadog.yml            # Datadog agent config — Nginx, MongoDB, Docker
├── docker-compose-monitoring.yml   # Full stack: Prometheus + Grafana + ELK
└── README.md
```

---

## What's Monitored

| Service | Tool | Metrics Collected |
|---|---|---|
| EC2 servers | Prometheus + Node Exporter | CPU, RAM, Disk, Network |
| Nginx | Prometheus Nginx Exporter | Request rate, Active connections, 5xx errors |
| Spring Boot | Prometheus + Actuator | Response time, Request count, JVM heap |
| MongoDB | Prometheus MongoDB Exporter | Replica lag, Connections, Operations/sec |
| Docker containers | cAdvisor | Per-container CPU/RAM/network |
| HTTP endpoints | Blackbox Exporter | Uptime, Response time, SSL expiry |
| App logs | ELK Stack (Filebeat→Logstash→Elasticsearch→Kibana) | Error rate, Stack traces |
| All services | Datadog Agent | APM, Logs, Infrastructure metrics |

---

## Quick Start — Spin Up Full Stack

```bash
# Clone repo
git clone https://github.com/your-username/monitoring-observability-setup.git
cd monitoring-observability-setup

# Start all monitoring services
docker-compose -f docker-compose-monitoring.yml up -d

# Check all containers are running
docker-compose -f docker-compose-monitoring.yml ps
```

### Access the dashboards

| Tool | URL | Default Login |
|---|---|---|
| Grafana | http://localhost:3000 | admin / admin |
| Prometheus | http://localhost:9090 | — |
| Alertmanager | http://localhost:9093 | — |
| Kibana | http://localhost:5601 | — |
| Elasticsearch | http://localhost:9200 | — |

---

## Prometheus Setup

### prometheus.yml
Scrapes metrics from:
- **Node Exporter** (port 9100) — server CPU, RAM, disk metrics
- **Nginx Exporter** (port 9113) — request rates, connections
- **Spring Boot Actuator** (`/actuator/prometheus`) — app metrics
- **MongoDB Exporter** (port 9216) — DB metrics and replica set health
- **cAdvisor** — Docker container resource usage
- **Blackbox Exporter** — HTTP endpoint uptime monitoring

### alert-rules.yml
Alerts configured for:

| Alert | Threshold | Severity |
|---|---|---|
| HighCPUUsage | CPU > 80% for 5m | Warning |
| CriticalCPUUsage | CPU > 95% for 2m | Critical |
| HighMemoryUsage | Memory > 85% for 5m | Warning |
| DiskSpaceWarning | Disk > 75% | Warning |
| DiskSpaceCritical | Disk > 90% | Critical |
| ServerDown | Node unreachable for 1m | Critical |
| NginxDown | Nginx unreachable for 1m | Critical |
| NginxHighErrorRate | 5xx errors > 10/sec | Warning |
| SpringBootDown | App unreachable for 2m | Critical |
| MongoDBDown | MongoDB unreachable for 1m | Critical |
| MongoDBReplicationLag | Lag > 30s | Warning |
| AppEndpointDown | HTTP check failed for 2m | Critical |
| SSLCertExpiringSoon | SSL expires in < 30 days | Warning |

---

## Grafana Dashboard

Import the dashboard from `grafana/dashboard.json`:

1. Open Grafana → **Dashboards** → **Import**
2. Click **Upload JSON file**
3. Select `grafana/dashboard.json`
4. Select your Prometheus data source
5. Click **Import**

**Dashboard panels:**
- CPU Usage % (current + over time)
- Memory Usage % (current + over time)
- Disk Usage %
- Services Up/Down status
- Nginx request rate and active connections

---

## ELK Stack Setup

### logstash.conf
Parses two log types:
- **Nginx access logs** — extracts IP, method, URL, status code, response time
- **Spring Boot logs** — extracts log level, thread, class, message
- Tags 5xx responses as `nginx_error` for quick Kibana filtering

### filebeat.yml
Install on each application server to ship logs:
```bash
apt install filebeat
cp elk/filebeat.yml /etc/filebeat/filebeat.yml
systemctl enable filebeat
systemctl start filebeat
```

Configured to ship:
- Nginx access and error logs
- Spring Boot / Tomcat logs (with multiline stack trace support)
- System logs (syslog, auth.log)

---

## Datadog

### datadog.yml
Configures Datadog agent to:
- Collect logs from Nginx and Spring Boot
- Monitor Nginx status endpoint
- Monitor MongoDB replica set
- Enable APM (Application Performance Monitoring)
- Enable Docker container monitoring

```bash
# Install Datadog agent
DD_API_KEY=your-key bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"

# Copy config
cp datadog/datadog.yml /etc/datadog-agent/datadog.yaml
systemctl restart datadog-agent
```

---

## Real Projects This Monitoring Supported

| Project | Monitoring Used | What Was Monitored |
|---|---|---|
| AspTax | Prometheus + Grafana + CloudWatch | EC2, ALB, RDS, Nginx, Spring Boot |
| Node App HA | Prometheus + ELK | MongoDB replica set, Node.js, Nginx LB |
| P-Collab | Azure Monitor + ELK | Azure App Service, MongoDB Atlas |
| Spring Boot clients | ELK + CloudWatch | Tomcat, app logs, DB performance |
| On-Premises KVM | Prometheus + Grafana | Host CPU/RAM, VM status |

---

## Tools & Versions

| Tool | Version |
|---|---|
| Prometheus | 2.48.0 |
| Grafana | 10.2.0 |
| Alertmanager | 0.26.0 |
| Elasticsearch | 8.11.0 |
| Logstash | 8.11.0 |
| Kibana | 8.11.0 |
| Filebeat | 8.11.0 |
| Datadog Agent | 7.x |
| Node Exporter | 1.7.0 |
| cAdvisor | 0.47.2 |

---

## Author

**Pavan Kishore Nakka**
DevOps & Cloud Engineer | 3+ Years Experience
AWS Certified Solutions Architect – Associate | AWS Certified Cloud Practitioner

[LinkedIn](https://www.linkedin.com/in/nakka-pavan-kishore)
