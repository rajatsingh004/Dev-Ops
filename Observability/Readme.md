# Observability Setup

This repository contains a **demo Kubernetes observability stack** that provides visibility into **metrics, logs, and traces** using open-source tools. The stack is deployed in the `observability` namespace and integrates multiple components to collect, store, and visualize telemetry data.

The goal of this setup is to demonstrate a **complete observability pipeline** that can be used for learning and experimentation.

---

# Architecture Overview

The stack is built around the three pillars of observability:
- **Metrics**
- **Logs**
- **Traces**

Visualization is handled centrally using :contentReference[oaicite:0]{index=0}.
## Components Used

| Signal | Tool |
|------|------|
| Metrics | :contentReference[oaicite:1]{index=1} |
| Logs | :contentReference[oaicite:2]{index=2} |
| Traces | :contentReference[oaicite:3]{index=3} |
| Trace Processing | :contentReference[oaicite:4]{index=4} |
| Log Collection | :contentReference[oaicite:5]{index=5} |
| Node Metrics | :contentReference[oaicite:6]{index=6} |
| Kubernetes State Metrics | :contentReference[oaicite:7]{index=7} |
| Alerting | :contentReference[oaicite:8]{index=8} |
| Visualization | :contentReference[oaicite:9]{index=9} |

---

# High-Level Architecture
                +-----------------------+
                |        Grafana        |
                | Dashboards / Queries  |
                +-----------+-----------+
                            |
    -------------------------------------------------------
    |                        |                           |
  Metrics                  Logs                        Traces
    |                        |                           |
    v                        v                           v

+---------------+ +---------------+ +----------------+
| Prometheus | | Loki | | Jaeger |
+-------+-------+ +-------+-------+ +--------+-------+
| | |
v v v
+---------------+ +---------------+ +---------------------+
| node-exporter | | Promtail | | OpenTelemetry |
| kube-state- | | (DaemonSet) | | Collector |
| metrics | +-------+-------+ +----------+----------+
+-------+-------+ | |
| v v
v Container Logs Applications
Kubernetes (/var/log/containers) (instrumented)


---

# Metrics Pipeline

The metrics pipeline is responsible for collecting system and Kubernetes metrics.

## Flow
Kubernetes Nodes / Cluster
↓
node-exporter + kube-state-metrics
↓
Prometheus
↓
Grafana


## Components

### node-exporter
Runs as a **DaemonSet** and exposes **node-level metrics** such as:
- CPU usage
- Memory usage
- Disk usage
- Network statistics

Example metrics:
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_filesystem_size_bytes


---

### kube-state-metrics
Exports **Kubernetes resource metrics** by reading the Kubernetes API.

Examples:
kube_pod_status_phase
kube_deployment_status_replicas
kube_node_status_condition


---

### Prometheus

Prometheus scrapes metrics every **15 seconds** from:
- `node-exporter`
- `kube-state-metrics`
- annotated Kubernetes pods

Prometheus is also configured to send alerts to **Alertmanager**.

---

### Alertmanager

Alertmanager receives alerts from Prometheus and manages alert routing.
Current configuration includes a **default receiver**.

---

# Logs Pipeline
The logs pipeline collects container logs from Kubernetes nodes and stores them in Loki.

## Flow


Container Logs
↓
/var/log/containers
↓
Promtail
↓
Loki
↓
Grafana


## Components

### Promtail

Promtail runs as a **DaemonSet** and collects logs from:


/var/log/containers/*.log


It enriches logs with Kubernetes metadata such as:

- namespace
- pod
- container

Logs are then pushed to Loki.

---

### Loki

Loki stores logs and allows them to be queried efficiently.

Logs can be explored in Grafana using queries such as:


{namespace="default"}


---

# Tracing Pipeline

The tracing pipeline captures **distributed traces** from applications.

## Flow


Application
↓
OTLP
↓
OpenTelemetry Collector
↓
Jaeger
↓
Grafana / Jaeger UI


## Components

### OpenTelemetry Collector

The collector receives telemetry via:

- **OTLP gRPC (4317)**
- **OTLP HTTP (4318)**

Current pipeline processes **traces only**.

Pipeline stages:

- Receiver → OTLP
- Processor → Batch
- Exporter → Jaeger

---

### Jaeger

Jaeger stores and visualizes distributed traces.

Using the **all-in-one image**, it provides:

- Collector
- Query service
- Storage
- UI

Traces can be explored in the Jaeger UI or through Grafana.

---

# Grafana Integration

Grafana is configured using **provisioned datasources**.

The following datasources are automatically configured:

| Datasource | Endpoint |
|------------|---------|
| Prometheus | http://prometheus:9090 |
| Loki | http://loki:3100 |
| Jaeger | http://jaeger:16686 |

This allows Grafana to query **metrics, logs, and traces from a single interface**.
