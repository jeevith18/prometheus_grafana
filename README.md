# Monitoring Stack with Docker Compose

A comprehensive monitoring solution built with Prometheus, Grafana, Alertmanager, and supporting services for infrastructure monitoring and alerting.

## 🚀 Overview

This project demonstrates a complete observability stack that provides:
- **Metrics Collection**: Prometheus with Node Exporter and cAdvisor
- **Alerting**: Alertmanager with email and webhook notifications
- **Visualization**: Grafana dashboards for monitoring insights
- **Service Discovery**: File-based service discovery for dynamic target management
- **Alert Testing**: MailHog for email testing and webhook logger for webhook testing

## 📋 Architecture

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│  Prometheus │────│ Alertmanager │────│   MailHog   │
│   :9090     │    │    :9093     │    │   :8025     │
└─────────────┘    └──────────────┘    └─────────────┘
       │                    │
       │                    │
       ▼                    ▼
┌─────────────┐    ┌──────────────┐
│   Grafana   │    │ Webhook Log  │
│   :3001     │    │   :8081      │
└─────────────┘    └──────────────┘
       │
       ▼
┌─────────────┐    ┌──────────────┐
│Node Exporter│    │   cAdvisor   │
│   :9100     │    │   :8080      │
└─────────────┘    └──────────────┘
```

## 🛠 Services

| Service | Port | Purpose |
|---------|------|---------|
| Prometheus | 9090 | Metrics collection and storage |
| Grafana | 3001 | Visualization and dashboards |
| Alertmanager | 9093 | Alert routing and notifications |
| Node Exporter | 9100 | Host system metrics |
| cAdvisor | 8080 | Container metrics |
| MailHog | 8025 | Email testing (SMTP server) |
| Webhook Logger | 8081 | Webhook testing and logging |

## 🚦 Quick Start

### Prerequisites
- Docker and Docker Compose installed
- Minimum 4GB RAM recommended
- Ports 3001, 8025, 8080, 8081, 9090, 9093, 9100 available

### 1. Clone and Navigate
```bash
cd /path/to/your/monitoring_stack
```

### 2. Start the Stack
```bash
docker-compose up -d
```

### 3. Verify Services
```bash
docker-compose ps
```

### 4. Access Applications
- **Grafana**: http://localhost:3001 (admin/admin)
- **Prometheus**: http://localhost:9090
- **Alertmanager**: http://localhost:9093
- **MailHog**: http://localhost:8025
- **cAdvisor**: http://localhost:8080

## 📊 Monitoring Features

### Metrics Collection
- **Node Metrics**: CPU, memory, disk, network from Node Exporter
- **Container Metrics**: Docker container resources from cAdvisor
- **Service Metrics**: Prometheus and Alertmanager self-monitoring

### Recording Rules
Pre-computed metrics for better performance:
- `instance:node_cpu_utilisation:rate5m`
- `instance:node_memory_utilisation:ratio`
- `cluster:container_cpu:sum_rate5m`

### Alert Rules
Configured alerts with severity levels:
- **Critical**: Node Down, High CPU Usage (>80%)
- **Warning**: High Memory Usage (>80%), High Container Memory

### Service Discovery
File-based service discovery using `targets.json`:
```json
[
  {
    "targets": ["node-exporter:9100"],
    "labels": {"job": "node-exporter", "env": "prod"}
  }
]
```

## 🔔 Alerting Configuration

### Routing Strategy
- **Critical Alerts**: Sent to webhook logger
- **Warning Alerts**: Sent to email (MailHog)
- **Inhibition**: NodeDown suppresses other alerts from same instance

### Notification Channels
1. **Email via MailHog**: View at http://localhost:8025
2. **Webhook Logging**: View logs with `docker-compose logs webhook-logger`

### Testing Alerts
```bash
# Stop node-exporter to trigger NodeDown alert
docker-compose stop node-exporter

# Check Alertmanager UI
open http://localhost:9093

# Check MailHog for emails
open http://localhost:8025
```

## 📈 Grafana Dashboards

### Infrastructure Monitoring Dashboard
Create panels with these queries:

#### 1. Node CPU Usage (%)
```promql
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

#### 2. Node Memory Usage (%)
```promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

#### 3. Container CPU Usage (%)
```promql
sum(rate(container_cpu_usage_seconds_total{name!=""}[5m])) by (name) * 100
```

#### 4. Active Alerts
```promql
ALERTS{alertstate="firing"}
```

### Dashboard Setup Steps
1. Login to Grafana (admin/admin)
2. Add Prometheus data source: `http://prometheus:9090`
3. Create new dashboard
4. Add panels with queries above
5. Configure visualizations and units

## 📁 File Structure

```
monitoring_stack/
├── docker-compose.yml          # Main orchestration file
├── prometheus/
│   ├── prometheus.yml          # Prometheus configuration
│   ├── targets.json           # Service discovery targets
│   ├── rules/
│   │   ├── recording_rules.yml # Performance optimization rules
│   │   └── alert_rules.yml    # Alerting rules
├── alertmanager/
│   └── alertmanager.yml       # Alert routing configuration
└── README.md                  # This file
```

## 🔧 Configuration Details

### Prometheus Configuration
- **Scrape Interval**: 15 seconds
- **Evaluation Interval**: 15 seconds
- **Retention**: 15 days (default)
- **Service Discovery**: File-based with relabeling

### Alertmanager Configuration
- **Group Wait**: 10 seconds
- **Group Interval**: 5 minutes
- **Repeat Interval**: 12 hours
- **Inhibition Rules**: Configured for NodeDown alerts

## 🐛 Troubleshooting

### Common Issues

#### Port Conflicts
```bash
# Check if ports are in use
lsof -i :9090
netstat -an | grep :9090
```

#### Service Not Starting
```bash
# Check logs
docker-compose logs prometheus
docker-compose logs alertmanager
```

#### No Metrics Appearing
```bash
# Verify targets in Prometheus
curl http://localhost:9090/api/v1/targets

# Check service discovery file
cat prometheus/targets.json
```

#### Alerts Not Firing
```bash
# Check alert rules syntax
promtool check rules prometheus/rules/alert_rules.yml

# Verify Alertmanager config
docker-compose exec alertmanager amtool config show
```

### Useful Commands

```bash
# View all containers
docker-compose ps

# Follow logs
docker-compose logs -f prometheus

# Restart specific service
docker-compose restart alertmanager

# Scale down/up
docker-compose down
docker-compose up -d

# Execute commands in containers
docker-compose exec prometheus promtool query instant 'up'
```

## 📚 Learning Resources

### Key Concepts Covered
1. **Docker Compose**: Multi-container orchestration
2. **Prometheus**: Time-series database and monitoring
3. **PromQL**: Prometheus Query Language
4. **Service Discovery**: Dynamic target management
5. **Recording Rules**: Performance optimization
6. **Alert Rules**: Proactive monitoring
7. **Alertmanager**: Alert routing and suppression
8. **Grafana**: Visualization and dashboards

### Next Steps
- Explore advanced PromQL queries
- Set up real SMTP for production alerts
- Add more exporters (blackbox, mysql, etc.)
- Implement Grafana alerting
- Add log aggregation with Loki
- Explore distributed tracing with Jaeger

## 🤝 Contributing

Feel free to submit issues and enhancement requests!

## 📄 License

This project is for educational purposes. Use at your own risk in production environments.

---
**Happy Monitoring!** 🎯📊
