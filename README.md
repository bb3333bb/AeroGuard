# AeroGuard: Autonomous Drone Surveillance & Patrol Platform

![Go Version](https://img.shields.io/badge/Go-1.22+-00ADD8?style=flat&logo=go)
![Architecture](https://img.shields.io/badge/Architecture-Microservices-blue)
![Database](https://img.shields.io/badge/PostGIS-Spatial_Data-336791)

**AeroGuard** is a high-performance, specialized backend platform designed for island surveillance and automated drone patrols. It leverages **Golang** for both high-throughput telemetry ingestion and complex mission scheduling, backed by **PostGIS** for geospatial intelligence.

## 🚀 Key Features

### 1. Real-time Telemetry Processing (Edge Layer)
- **High-speed Ingestion:** Handles 10k+ coordinate packets/sec via MQTT/UDP using Golang's concurrency primitives.
- **Dynamic Geofencing:** Real-time checking of drone positions against "No-Fly Zones" or "Critical Areas" using Ray Casting algorithms (or PostGIS queries).
- **Time-series Storage:** Optimized flight path storage using **TimescaleDB**.

### 2. Intelligent Mission Planning (Core Layer)
- **Asset Management:** Tracks drones, batteries, and charging stations.
- **Advanced Scheduling:** - Supports complex patterns: *Daily patrol at 08:00*, *Every 3 days*, *Start at Sunset*.
  - Auto-assigns drones based on battery level and proximity.
- **Spatial Management:** Define patrol routes and surveillance polygons directly on the map.

### 3. Anomaly Detection & Analytics
- Analysis of flight deviations.
- Change detection alerts (integrated with Computer Vision module).

---

## 🛠 Tech Stack

- **Core Language:** Golang 1.22+
- **Communication:** REST API (Gin), gRPC, MQTT (Eclipse Paho).
- **Databases:**
  - **PostgreSQL + PostGIS:** For spatial data (Zones, Routes).
  - **TimescaleDB:** For telemetry (Sensor data, Paths).
  - **Redis:** For caching and Pub/Sub (Real-time dashboard updates).
- **Task Queue:** `hibiken/asynq` (Redis backed) for mission dispatching.
- **Infrastructure:** Docker, Docker Compose.

---

## 🧩 Architecture Overview

```mermaid
graph TD
    Drone[Drone Fleet] -->|MQTT/MAVLink| Gateway[Telemetry Gateway (Go)]
    Gateway -->|Store Path| TimescaleDB[(TimescaleDB)]
    Gateway -->|Geofence Alert| RedisPub[Redis Pub/Sub]
    
    User[Operator] -->|REST API| MissionCtrl[Mission Control (Go)]
    MissionCtrl -->|CRUD| PostGIS[(PostGIS)]
    MissionCtrl -->|Schedule| Asynq[Task Queue]
    
    Asynq -->|Trigger Mission| Gateway
    Gateway -->|Command| Drone
```

## 🗂 Project Structure

- `cmd/telemetry-gateway`: Service handling raw data streams.
- `cmd/mission-control`: Service handling business logic, scheduling, and API.
- `cmd/drone-simulator`: CLI tool to simulate 100+ drones flying around an island.
- `pkg/geo`: Shared geospatial calculation library.

## ⚡ Getting Started

### Prerequisites
- Go 1.22+
- Docker & Docker Compose

### 1. Start Infrastructure
Spin up Postgres (PostGIS), Redis, and MQTT Broker.
```bash
make up
# Or: docker-compose up -d
```

### 2. Run Migrations
Initialize database schemas and spatial extensions.
```bash
make migrate
```

### 3. Start Services
```bash
# Terminal 1: Start Mission Control API
go run cmd/mission-control/main.go

# Terminal 2: Start Telemetry Gateway
go run cmd/telemetry-gateway/main.go
```

### 4. Run Simulation
Launch a fake drone flying over the "Island" area.
```bash
go run cmd/drone-simulator/main.go --drone-id=d01 --pattern=circle
```

## 🛡️ Senior Implementation Highlights
* **Hexagonal Architecture:** Decoupled business logic from database/transport layers.
* **Concurrency Patterns:** Worker pools implemented in `telemetry-gateway` for backpressure handling.
* **Geospatial Indexing:** Heavy usage of GiST indexes in PostGIS for millisecond-level spatial queries.

## 📜 License
MIT
