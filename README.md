# 🚦 AI + IoT Smart Traffic Monitoring System

> A production-ready cloud-based smart traffic management platform using **AI**, **IoT**, **Computer Vision**, **PostgreSQL**, and a modern **Web Dashboard**.

![Tech Stack](https://img.shields.io/badge/AI-YOLOv8-blue)
![Backend](https://img.shields.io/badge/Backend-FastAPI-green)
![Frontend](https://img.shields.io/badge/Frontend-Next.js-black)
![DB](https://img.shields.io/badge/DB-PostgreSQL-blue)
![IoT](https://img.shields.io/badge/IoT-MQTT-orange)

---

## 📋 Features

| # | Feature | Status |
|---|---------|--------|
| 1 | Vehicle Density Detection (CCTV/Video/Stream) | ✅ |
| 2 | Number Plate Recognition (ANPR) | ✅ |
| 3 | Vehicle Type Detection (car, bike, truck, bus, ambulance) | ✅ |
| 4 | Auto Traffic Signal Duration Change | ✅ |
| 5 | Traffic Violation Detection | ✅ |
| 6 | Accident Detection | ✅ |
| 7 | Congestion Prediction | ✅ |
| 8 | Emergency Ambulance Verification | ✅ |
| 9 | Smart Dashboard Website | ✅ |
| 10 | IoT Traffic Light Control | ✅ |

### Input Sources
- ✅ Upload prerecorded video files (MP4, AVI)
- ✅ RTSP live camera streams
- ✅ Process via URL (including YouTube)

---

## 🏗️ Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Frontend   │◄──►│   Backend   │◄──►│  AI Engine  │
│  (Next.js)   │    │  (FastAPI)  │    │  (YOLOv8)   │
│  Port: 3000  │    │  Port: 8000 │    │  Port: 8001 │
└──────┬───────┘    └──────┬──────┘    └─────────────┘
       │                   │
       │            ┌──────┴──────┐
       │            │ PostgreSQL  │
       │            │  Port: 5432 │
       │            └──────┬──────┘
       │                   │
       │            ┌──────┴──────┐    ┌─────────────┐
       └────────────│    Nginx    │    │ MQTT Broker  │
                    │  Port: 80   │    │  Port: 1883  │
                    └─────────────┘    └──────┬───────┘
                                              │
                                       ┌──────┴──────┐
                                       │ IoT Devices │
                                       │ (ESP32/RPi) │
                                       └─────────────┘
```

---

## 📁 Project Structure

```
smart-traffic-system/
├── frontend/          # Next.js + Tailwind CSS Dashboard
├── backend/           # FastAPI + PostgreSQL Backend
│   ├── app/
│   │   ├── routes/    # API route handlers
│   │   ├── models.py  # SQLAlchemy ORM models
│   │   ├── schemas.py # Pydantic schemas
│   │   ├── config.py  # Settings management
│   │   └── ...
│   └── db/init.sql    # Database schema
├── ai-engine/         # YOLOv8 + OpenCV Pipeline
│   ├── detector.py    # Vehicle/plate/accident detection
│   ├── processor.py   # Video processing pipeline
│   └── api.py         # AI Engine API server
├── iot/               # IoT Signal Controllers
│   ├── signal_controller.py  # Raspberry Pi controller
│   └── esp32_signal/  # ESP32 Arduino firmware
├── devops/            # Docker, Nginx, CI/CD
├── docs/              # Documentation
└── docker-compose.yml # Full orchestration
```

---

## 🚀 Quick Start

### Prerequisites
- Python 3.11+
- Node.js 20+
- PostgreSQL 16+
- Docker & Docker Compose (for containerized deployment)

### 1. Clone & Setup

```bash
git clone <repo-url>
cd smart-traffic-system
```

### 2. Database Setup

```bash
# Start PostgreSQL (or use Docker)
docker run -d --name traffic-pg \
  -e POSTGRES_USER=traffic_admin \
  -e POSTGRES_PASSWORD=traffic_secure_2026 \
  -e POSTGRES_DB=traffic_data \
  -p 5432:5432 \
  postgres:16-alpine

# Run schema
psql -U traffic_admin -d traffic_data -f backend/db/init.sql
```

### 3. Start Backend

```bash
cd backend
pip install -r requirements.txt
cp .env.example .env  # Edit with your settings
uvicorn main:app --reload --port 8000
```

### 4. Start AI Engine

```bash
cd ai-engine
pip install -r requirements.txt
uvicorn api:app --reload --port 8001
```

### 5. Start Frontend

```bash
cd frontend
npm install
npm run dev
```

### 6. Docker Compose (All-in-One)

```bash
docker-compose up -d
```

---

## 🔌 API Routes

### Vehicles
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/vehicles/` | List all detected vehicles |
| GET | `/api/vehicles/search?plate_no=` | Search by plate number |
| GET | `/api/vehicles/types` | Vehicle type distribution |
| POST | `/api/vehicles/` | Log new vehicle detection |

### Signals
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/signals/` | List all signals |
| PUT | `/api/signals/{id}` | Update signal |
| POST | `/api/signals/update-timing` | Update timing by density |

### Processing
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/processing/upload` | Upload video file |
| POST | `/api/processing/rtsp` | Process RTSP stream |
| POST | `/api/processing/url` | Process video URL |
| GET | `/api/processing/{id}` | Get job status |

### Ambulance
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/ambulance/authorized` | List authorized ambulances |
| POST | `/api/ambulance/requests` | Create emergency request |
| POST | `/api/ambulance/requests/{id}/verify` | Verify request |

### Analytics
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/analytics/dashboard` | Dashboard statistics |
| GET | `/api/analytics/traffic-trends` | Traffic volume trends |
| GET | `/api/analytics/vehicle-distribution` | Vehicle type breakdown |

### WebSocket
| Endpoint | Description |
|----------|-------------|
| `ws://host/ws/live` | Real-time traffic updates |
| `ws://host/ws/alerts` | Alert notifications |
| `ws://host/ws/density` | Density updates |

---

## 🚦 Signal Logic

| Density Level | Vehicles | Green Duration |
|---------------|----------|----------------|
| **Low** | ≤ 10 | 20 seconds |
| **Medium** | 11-25 | 40 seconds |
| **High** | > 25 | 70 seconds |
| **Emergency** | Ambulance | 120 seconds (all route) |

**Emergency Logic:**
1. Ambulance detected → Check authorized list
2. If verified → All route signals → GREEN priority mode
3. If not verified → Pending for manual approval
4. On completion → Signals restore to auto mode

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js, Tailwind CSS, Recharts |
| Backend | FastAPI, SQLAlchemy, Pydantic |
| AI/CV | Python, YOLOv8, OpenCV, EasyOCR |
| Database | PostgreSQL 16 |
| Real-time | WebSocket |
| IoT | MQTT, ESP32, Raspberry Pi |
| Cloud | Docker, Nginx, GitHub Actions |

---

## 📦 Deployment

See [docs/deployment-guide.md](docs/deployment-guide.md) for detailed instructions.

### Quick Docker Deploy
```bash
docker-compose up -d --build
```

Access at: `http://localhost` (Nginx), `http://localhost:3000` (Direct Frontend)

---

## 📄 License

MIT License - See LICENSE file for details.
