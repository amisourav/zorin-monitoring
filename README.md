# zorin-monitoring
Monitor Zorin OS using docker stack consisting Grafana, Prometheus, Node-Exporter and Alert Manager
## Table of Contents

| Section | Description |
|---------|-------------|
| [Project Overview](#project-overview) | What the project does |
| [Architecture](#architecture) | How the stack is built |
| [Prerequisites](#prerequisites) | What you need to run it |
| [Installation & Setup](#installation--setup) | Get the stack up and running |
| [Configuration](#configuration) | Tweak Prometheus, Grafana, and Alertmanager |
| [Dashboards & Alerts](#dashboards--alerts) | View metrics & create alerts |
| [Troubleshooting](#troubleshooting) | Common problems |
| [Contributing](#contributing) | How to help |
| [License](#license) | The license terms |

---

## Project Overview

**Zorin‑Monitoring** is a Docker‑Compose based monitoring solution tailored for Zorin OS.  
It brings together:

- **Prometheus** – data collection & storage  
- **Node‑Exporter** – export Zorin OS system metrics  
- **Alertmanager** – notify on critical events  
- **Grafana** – visualize the data

All you need is Docker, and the stack is ready to run in minutes.

---

## Architecture

```
┌─────────────────────┐
│    Grafana          │
│ (Dashboard, Alert)  │
└───────┬───────┬───────┘
        │       │
        │       │
┌───────▼───────▼───────┐
│   Prometheus          │
│ (Storage, Query API)  │
└───────┬───────┬───────┘
        │       │
┌───────▼───────▼───────┐
│   Node‑Exporter       │
│ (Zorin OS metrics)    │
└───────────────────────┘
```

- **Grafana** connects to Prometheus via its HTTP API.
- **Prometheus** scrapes metrics from the **Node‑Exporter**.
- **Alertmanager** receives alert rules from Prometheus and routes notifications.

---

## Prerequisites

| Item | Version |
|------|---------|
| Docker Engine | 20.10+ |
| Docker Compose | 2.5+ |
| Zorin OS 16+ | (tested) |

> **Tip:** If you’re on an older Docker Compose v1, replace `docker compose` with `docker-compose`.

---

## Installation & Setup

> All commands assume you are in the root of the repository where `docker-compose.yml` lives.

### 1. Clone the repo

```bash
git clone https://github.com/your-username/zorin-monitoring.git
cd zorin-monitoring
```

### 2. Build / Pull the images

```bash
docker compose pull
# Or build locally
# docker compose build
```

### 3. Start the stack

```bash
docker compose up -d
```

> ⚠️ `-d` runs the stack in **detached mode** (background).  
> To see logs: `docker compose logs -f`.

### 4. Verify services

```bash
docker compose ps
```

You should see:

| Service        | Status   |
|----------------|----------|
| grafana        | Up |
| prometheus     | Up |
| node_exporter  | Up |
| alertmanager   | Up |

---

## Configuration

### 1. Prometheus

- `prometheus/prometheus.yml`  
  - Add/modify scrape targets if you have multiple nodes.
  - Adjust `scrape_interval`, `evaluation_interval`, etc.

### 2. Node‑Exporter

- Runs on the host machine; no extra config required for basic metrics.

### 3. Alertmanager

- `alertmanager/alertmanager.yml`  
  - Set up email, Slack, or other webhook integrations.
  - Example Slack config:

```yaml
global:
  resolve_timeout: 5m
route:
  receiver: slack-notifications
receivers:
- name: slack-notifications
  slack_configs:
  - api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
    channel: '#alerts'
```

### 4. Grafana

- Default credentials: `admin / admin`  
- After first login, go to **Configuration → Data Sources** → select `Prometheus` and point it to `http://prometheus:9090`.
- Import dashboards from the `dashboards/` folder or Grafana.com.

---

## Dashboards & Alerts

### Grafana Dashboards

Import the pre‑configured dashboards:

1. In Grafana → **Create** → **Import**.  
2. Upload `dashboards/system.json`.  
3. Set the data source to `Prometheus`.

Key panels:

- **CPU & Memory** – per‑core usage, swap, usage, load
- **Disk I/O** – read/write bytes, latency
- **Network** – packets, errors, throughput
- **Zorin OS Specific** – uptime, package updates, etc.

### Alert Rules

Prometheus Alertmanager uses a separate `alert.rules.yml` file.

Example rule (CPU > 80%):

```yaml
groups:
- name: system
  rules:
  - alert: HighCPU
    expr: node_cpu_seconds_total{mode="idle"} < 0.20
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "CPU usage high on {{ $labels.instance }}"
      description: "CPU usage exceeded 80% for 5 minutes."
```

> Reload Prometheus after editing: `docker compose exec prometheus kill -HUP 1`

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Grafana login fails | Wrong credentials | Reset via Grafana admin UI or `docker compose exec grafana grafana-cli admin reset-admin-password <newpass>` |
| Prometheus scrape errors | Node‑Exporter not reachable | Ensure `node_exporter` container is running; check firewall/port 9100 |
| Alertmanager silent | Slack webhook wrong | Verify `api_url` and channel; check logs `docker compose logs alertmanager` |
| Metrics missing | Scrape interval too low | Increase `scrape_interval` in `prometheus.yml` |

---

## Contributing

Feel free to submit issues or PRs.  
**Pull‑request checklist:**

1. Updated documentation (if needed)
2. Added tests (if applicable)
3. Linting passes (`make lint` or `docker compose run --rm lint`)
4. No breaking changes without notice

---

## License

MIT © 2025 [Your Name]  
See the [LICENSE](LICENSE) file for details.

---

### Need help?

Open an issue or reach out on our [Discord](https://discord.gg/your-invite) channel.

Happy monitoring! 🚀
