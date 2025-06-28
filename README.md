# 💫 Grafana OpenTelemetry Infrastructure

A complete observability stack using Grafana, Alloy, Loki, and Tempo for collecting, storing, and visualizing logs, metrics, and traces.

## 🌐 Architecture Overview

This Docker Compose setup provides a full observability stack with the following components:

- **Grafana**: Visualization and monitoring dashboard
- **Alloy**: OpenTelemetry collector for receiving and forwarding telemetry data
- **Loki**: Log aggregation system
- **Tempo**: Distributed tracing backend
- **Memcached**: Cache layer for Tempo performance optimization

## 🔧 Components

### Grafana

**Purpose**: Web-based visualization and monitoring dashboard for logs, metrics, and traces

- **Container**: `grafana`
- **Image**: `grafana/grafana`
- **Exposed Port**: `3000:3000` (HTTP Web UI)
- **Access**: [localhost:3000](http://localhost:3000)
- **Volume Mounts**:
  - Data storage: `./volumes/grafana_data`

### Alloy (OpenTelemetry Collector)

**Purpose**: Receives telemetry data (logs, metrics, traces) via OTLP and forwards them to appropriate backends

- **Container**: `alloy`
- **Image**: `grafana/alloy`
- **Exposed Ports**:
  - `12345:12345` (HTTP Management/UI)
  - `4319:4319` (OTLP gRPC Receiver)
- **Access**:
  - Management UI: [localhost:12345](http://localhost:12345)
  - OTLP Endpoint: [localhost:4319](http://localhost:4319) (gRPC)
- **Volume Mounts**:
  - Configuration: `./configs/config.alloy`
  - Data storage: `./volumes/alloy_data`
  - System logs: `/var/log` (read-only)

### Loki (Logs and Metrics Aggregation)

**Purpose**: Log and metrics aggregation system for storing and querying application logs

- **Container**: `loki`
- **Image**: `grafana/loki`
- **Exposed Port**: `3100:3100` (HTTP API)
- **Private Port**: `9096` (gRPC - internal only)
- **Access**: [localhost:3100](http://localhost:3100)
- **Volume Mounts**:
  - Configuration: `./configs/loki-config.yaml`
  - Data storage: `./volumes/loki_data`
  - Runtime data: `./volumes/loki_data_var`
  - WAL storage: `./volumes/loki_data_wal`

### Tempo (Tracing backend)

**Purpose**: Distributed tracing backend for storing and querying trace data

- **Container**: `tempo`
- **Image**: `grafana/tempo`
- **Exposed Ports**:
  - `3200:3200` (HTTP API for queries)
  - `4317:4317` (OTLP gRPC Receiver)
- **Access**: [localhost:3200](http://localhost:3200)
- **Volume Mounts**:
  - Configuration: `./configs/tempo.yaml`
  - Data storage: `./volumes/tempo_data`

### Memcached

**Purpose**: Caching layer to improve Tempo query performance

- **Container**: `memcached`
- **Image**: `memcached`
- **Exposed Port**: `11211:11211` (Memcached protocol)
- **Private Access**: Used internally by Tempo
- **Configuration**:
  - Max Memory: 64MB
  - Threads: 4

## 🌐 Port Summary

| Service | Port | Protocol | Access | Purpose |
|---------|------|----------|--------|---------|
| Grafana | 3000 | HTTP | **Public** | Web UI Dashboard |
| Alloy | 12345 | HTTP | **Public** | Management UI |
| Alloy | 4319 | gRPC | **Public** | OTLP Receiver |
| Loki | 3100 | HTTP | **Public** | API Endpoint |
| Loki | 9096 | gRPC | **Private** | Internal gRPC |
| Tempo | 3200 | HTTP | **Public** | Query API |
| Tempo | 4317 | gRPC | **Public** | OTLP Receiver |
| Memcached | 11211 | TCP | **Public** | Cache Protocol |

## 📝 Configuration Files

- `configs/config.alloy`: Alloy OpenTelemetry collector configuration
- `configs/loki-config.yaml`: Loki log aggregation configuration
- `configs/tempo.yaml`: Tempo tracing backend configuration

## 📦️ Data Volumes

All persistent data is stored in the bind-mounted `volumes/` directory:

- `alloy_data/`: Alloy collector data
- `grafana_data/`: Grafana dashboards and settings
- `loki_data/`: Loki log storage
- `loki_data_var/`: Loki runtime data
- `loki_data_wal/`: Loki write-ahead log
- `tempo_data/`: Tempo trace storage

It is advisable to not use bind-mounted volumes in production environments. Instead, use Docker volumes or other storage solutions to ensure data persistence and integrity.

## ⚡️ Quick Start

1. **Start the stack**:

   ```bash
   docker-compose up -d
   ```

2. **Access Grafana**:
   - URL: [localhost:3000](http://localhost:3000)
   - Default credentials: admin/admin

3. **Send telemetry data**:
   - OTLP endpoint: [localhost:4319](http://localhost:4319) (gRPC)
   - Logs are automatically collected by default from `/var/log`
        - Configure this from `configs/config.alloy` to include your application's logs as well

4. **Check service status**:

   ```bash
   docker-compose ps
   ```

## ✈️ Data Flow

1. **Logs**: System logs from `/var/log` → Alloy → Loki
2. **Traces and Spans**: Applications → Alloy (OTLP) → Tempo
3. **Metrics**: Applications → Alloy (OTLP) → Tempo (metrics generator)
4. **Visualization**: Grafana queries Loki and Tempo for dashboard display

## 🛂 Retention Policies

- **Loki**: 168 hours (7 days) log retention
- **Tempo**: 24 hours trace retention

## 🔒️ Security Notes

- All services run without authentication (suitable for development/testing)
- Tempo and Loki use TLS disabled for internal communication
- System log access requires root privileges for Alloy container

## 🧑‍💻 Authors

- [ZephyrCodesStuff](https://github.com/ZephyrCodesStuff): System architect and developer
- [AuraDoesScience](https://github.com/AuraDoesScience): Documentation, research, and testing
