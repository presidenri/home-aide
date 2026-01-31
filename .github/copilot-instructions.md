# Copilot Instructions for home-aide

## Project Overview
This is a complete **Home Assistant + Frigate NVR** setup using Docker Compose. It provides a full home automation hub with AI-powered surveillance, real-time object detection, person/vehicle tracking, and intelligent recording capabilities all integrated in one stack.

## Architecture & Key Components

### Docker Services
- **homeassistant**: Main home automation hub
  - Image: `ghcr.io/home-assistant/home-assistant:stable`
  - Web UI: Port `8123`
  - Manages automations, integrations, and device control
- **frigate**: AI-powered NVR service
  - Image: `ghcr.io/blakeblackshear/frigate:stable`
  - Web UI: Port `8971`
  - RTSP streams: Port `8554`
  - Uses tmpfs cache (1GB) to reduce storage wear
- **mqtt**: Mosquitto MQTT broker for inter-service communication
  - Port `1883` (MQTT), `9001` (WebSocket)
  - Enables real-time communication between services

### Directory Structure
- `config/`: Configuration files for all services
  - `config.yml`: Main Frigate configuration
  - `homeassistant/`: Home Assistant configuration files
  - `mosquitto/`: MQTT broker configuration
- `storage/`: Persistent data for all services
  - `frigate/`: Video recordings, snapshots, clips
  - `homeassistant/`: HA database and user data
  - `mosquitto/`: MQTT data and logs
- `docker-compose.yml`: Complete stack orchestration
- `.env.example`: Environment variables template

## Development Workflows

### Starting/Managing Services
```bash
# Copy environment template
cp .env.example .env

# Edit environment variables
nano .env

# Start all services
docker compose up -d

# View logs
docker compose logs -f frigate
docker compose logs -f mqtt
docker compose logs -f homeassistant

# Restart after config changes
docker compose restart frigate

# Stop services
docker compose down
```

### Configuration Management
- Frigate config goes in `config/config.yml` (create if missing)
- Config changes require container restart: `docker compose restart frigate`
- Access web interface at `http://localhost:8971` for configuration validation

## Project-Specific Patterns

### Configuration Structure
Frigate uses YAML configuration with these key sections:
- `cameras`: Define camera streams and detection zones
- `objects`: Configure object detection classes
- `record`: Set recording retention policies
- `snapshots`: Configure snapshot settings
- `detectors`: Configure AI detection models (CPU/OpenVINO/TensorRT)

### Storage Management
- Recordings are automatically managed by Frigate's retention policies
- `storage/` directory will contain subdirectories like:
  - `recordings/`: Continuous recordings
  - `clips/`: Event-triggered clips
  - `snapshots/`: Still images from detections

### Network Configuration
- RTSP streams accessible on port `8554`
- Web UI on port `8971` (customizable via environment)
- Camera streams typically use RTSP URLs in config

## Integration Points

### Camera Integration
- Supports RTSP, HTTP, and local USB cameras
- Common camera brands: Reolink, Dahua, Hikvision, Wyze
- Use `ffmpeg` probe commands to test camera streams before adding to config

### Home Assistant Integration
- Frigate provides native Home Assistant integration
- Publishes MQTT topics for events and statistics
- Provides camera entities, sensors, and binary sensors

### Hardware Acceleration
- Configure GPU acceleration via `detectors` section
- Supports Intel OpenVINO, NVIDIA TensorRT, Google Coral TPU
- CPU-only detection is default but resource-intensive

## Common Configuration Examples

### Basic Camera Setup
```yaml
cameras:
  front_door:
    ffmpeg:
      inputs:
        - path: rtsp://user:pass@camera_ip:554/stream
          roles: ["detect", "record"]
```

### Detection Zone
```yaml
cameras:
  front_door:
    zones:
      entry_area:
        coordinates: 100,100,200,100,200,200,100,200
```

When working with this codebase:
1. Always validate camera streams with `ffprobe` before adding to config
2. Use the web UI for real-time config validation
3. Monitor container logs when debugging detection issues
4. Consider hardware acceleration for multiple camera setups