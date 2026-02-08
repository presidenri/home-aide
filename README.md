# 🏠 Home Aide

<div align="center">

**AI-Powered Home Automation & Surveillance System**

[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Integrated-41BDF5?style=for-the-badge&logo=homeassistant&logoColor=white)](https://www.home-assistant.io/)
[![Frigate](https://img.shields.io/badge/Frigate%20NVR-Active-00B4A0?style=for-the-badge)](https://frigate.video/)
[![OpenVINO](https://img.shields.io/badge/OpenVINO-Accelerated-0071C5?style=for-the-badge&logo=intel&logoColor=white)](https://docs.openvino.ai/)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

*A complete **Home Assistant + Frigate NVR** setup using Docker Compose. Featuring AI-powered surveillance with OpenVINO acceleration, real-time object detection, intelligent recording, and seamless home automation in one integrated stack.*

</div>

---

## 📑 Table of Contents

- [✨ What's New](#-whats-new)
- [Features](#features)
- [🏗️ Architecture](#️-architecture)
- [🚀 Quick Start](#-quick-start)
- [⚙️ Configuration](#️-configuration)
- [🛠️ Technology Stack](#️-technology-stack)
- [📋 Management Commands](#-management-commands)
- [🚀 Hardware Acceleration](#-hardware-acceleration)
- [🔧 Troubleshooting](#-troubleshooting)
- [🔨 Development](#-development)
- [⚡ Performance & Optimization](#-performance--optimization)
- [🎯 Features & Capabilities](#-features--capabilities)
- [🔐 Security Considerations](#-security-considerations)
- [Contributing](#contributing)
- [❓ FAQ](#-faq)
- [📚 Additional Resources](#-additional-resources)

---

## ✨ What's New

- 🧠 **OpenVINO AI Acceleration**: Upgraded from CPU to Intel OpenVINO with YOLOv9-T model for faster, more efficient object detection
- ☁️ **Cloudflare Tunnel**: Secure remote access to Home Assistant without port forwarding
- 📊 **Custom Dashboards**: Pre-built security dashboards with live camera feeds and detection status
- 🎯 **Active Camera Configuration**: Hikvision DS-2CD2387G3-LI2UY (8MP/4K) fully configured with dual-stream support
- 🔧 **Environment-Based Config**: Camera credentials managed via .env for easier deployment

## Features

- 🏠 **Home Assistant**: Complete home automation platform with web UI and custom security dashboards
- 📹 **Frigate NVR**: AI-powered network video recorder with real-time object detection using YOLOv9-T
- 🧠 **OpenVINO Detector**: CPU-optimized AI inference for efficient object detection
- 📡 **MQTT Integration**: Seamless communication between services with automatic discovery
- 🔍 **Object Detection**: Person, vehicle (car, bicycle, motorcycle), and pet (cat, dog) detection with configurable zones
- 💾 **Smart Recording**: Automatic retention policies (14 days) with event-triggered clips and snapshots
- 🌐 **Remote Access**: Secure access via Cloudflare Tunnel without exposing ports
- 📱 **Mobile Ready**: Responsive web interfaces accessible from any device
- 🔧 **Hardware Acceleration**: Support for Intel OpenVINO (active), NVIDIA TensorRT, Google Coral TPU

## 🏗️ Architecture

```mermaid
graph TB
    subgraph "External Access"
        Internet[🌐 Internet]
        CloudFlare[☁️ Cloudflare Tunnel<br/>Secure Remote Access]
        Camera[📹 Hikvision Camera<br/>DS-2CD2387G3-LI2UY<br/>8MP/4K]
    end
    
    subgraph "Docker Network - frigate-network"
        subgraph "Home Automation"
            HA[🏠 Home Assistant<br/>:8123<br/>Automation Hub]
            HADash[📊 Dashboards<br/>- Security Dashboard<br/>- Camera Views<br/>- Detection Status]
        end
        
        subgraph "AI/NVR Services"
            Frigate[🔍 Frigate NVR<br/>:8971 Web, :8554 RTSP<br/>Object Detection]
            OpenVINO[🧠 OpenVINO Detector<br/>YOLOv9-T Model<br/>CPU Optimized]
            Detection[🎯 Detection Engine<br/>Person, Car, Bicycle<br/>Motorcycle, Pets]
        end
        
        subgraph "Communication"
            MQTT[📡 MQTT Broker<br/>Mosquitto<br/>:1883, :9001<br/>Message Bus]
        end
    end
    
    subgraph "Storage"
        LocalConfig[💾 Local Storage<br/>Config & DB<br/>./configs/frigate]
        NetworkStorage[🗄️ Network Storage<br/>/mnt/unas2/frigate<br/>Recordings & Clips]
        HAStorage[💾 HA Storage<br/>Database & Logs<br/>./storage/homeassistant]
    end
    
    Internet --> CloudFlare
    CloudFlare --> HA
    Camera -->|RTSP Stream<br/>Main: 4K<br/>Sub: 640x360| Frigate
    
    Frigate --> OpenVINO
    OpenVINO --> Detection
    Detection --> Frigate
    Frigate <-->|MQTT Messages<br/>Detection Events| MQTT
    HA <-->|MQTT Discovery<br/>State Updates| MQTT
    HA --> HADash
    
    Frigate --> LocalConfig
    Frigate --> NetworkStorage
    HA --> HAStorage
    
    style HA fill:#41BDF5
    style Frigate fill:#1A3A52
    style MQTT fill:#3C5280
    style OpenVINO fill:#0071C5
    style Detection fill:#00B4A0
    style CloudFlare fill:#F38020
    style Camera fill:#E31E24
```

### How It Works

1. **Camera Streaming**: Hikvision camera sends dual RTSP streams (4K for recording, 640x360 for detection)
2. **AI Detection**: OpenVINO with YOLOv9-T model processes video frames for object detection
3. **Event Communication**: Detection events published via MQTT to Home Assistant
4. **Smart Storage**: Recordings stored on network storage, config/database on local SSD for performance
5. **Remote Access**: Cloudflare Tunnel provides secure access without port forwarding
6. **Automation**: Home Assistant receives real-time detection events for automation triggers

## 🚀 Quick Start

### Prerequisites

- Docker and Docker Compose v2.0+
- At least 4GB RAM (8GB+ recommended for multiple cameras)
- Storage space for recordings (14-day retention, varies by camera count)
- (Optional) Cloudflare account for remote access

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd home-aide
   ```

2. **Configure environment**
   ```bash
   cp .env.example .env
   nano .env  # Edit timezone, passwords, and camera credentials
   ```
   
   Required environment variables:
   - `TZ`: Your timezone (e.g., America/New_York)
   - `FRIGATE_CAMERA_USERNAME`: Camera RTSP username
   - `FRIGATE_CAMERA_PASSWORD`: Camera RTSP password
   - `FRIGATE_CAMERA_IP`: Camera IP address
   - `CLOUDFLARE_TUNNEL_TOKEN`: (Optional) Cloudflare Tunnel token

3. **Start the services**
   ```bash
   docker compose up -d
   ```

4. **Access the interfaces**
   - Home Assistant: http://localhost:8123 (initial setup required)
   - Frigate NVR: http://localhost:8971 (check live detection and events)
   - MQTT WebSocket: http://localhost:9001
   - Remote Access: Your custom Cloudflare domain (if configured)

## ⚙️ Configuration

### Current Setup

The system is currently configured with:
- **Camera**: Hikvison DS-2CD2387G3-LI2UY (8MP/4K)
  - Main stream: 4K UHD for high-quality recording
  - Sub stream: 640x360 for efficient AI detection
  - Detection zone: Main area covering front view
  - Objects tracked: Person, car, bicycle, motorcycle
- **AI Model**: YOLOv9-T optimized for Intel CPUs via OpenVINO
- **Storage**: Network storage mounted at `/mnt/unas2/frigate` for recordings
- **Retention**: 14 days for recordings, detections, and snapshots

### Adding Additional Cameras

Edit [configs/frigate/config.yml](configs/frigate/config.yml) to add more cameras:

```yaml
cameras:
  front_door:
    enabled: true
    ffmpeg:
      inputs:
        # High res stream for recording
        - path: rtsp://{FRIGATE_CAMERA_USERNAME}:{FRIGATE_CAMERA_PASSWORD}@{FRIGATE_CAMERA_IP}:554/main_stream
          roles: ["record"]
        # Low res stream for detection
        - path: rtsp://{FRIGATE_CAMERA_USERNAME}:{FRIGATE_CAMERA_PASSWORD}@{FRIGATE_CAMERA_IP}:554/sub_stream
          roles: ["detect"]
    detect:
      width: 640
      height: 360
      fps: 5
    zones:
      entry_area:
        coordinates: 100,100,600,100,600,300,100,300
        objects:
          - person
          - car
```

**Tip**: Use the Frigate web UI at http://localhost:8971 to visually define detection zones by clicking on the camera feed.

### Home Assistant Dashboards

Pre-configured dashboards available in [configs/homeassistant/dashboards/](configs/homeassistant/dashboards/):
- **security_dashboard.yaml**: Main security overview with live feeds and detection status  
- **simple_cameras.yaml**: Basic camera grid view
- **working_cameras.yaml**: Troubleshooting dashboard

To add a dashboard:
1. Copy dashboard YAML from the `dashboards/` folder
2. In Home Assistant, go to Settings → Dashboards → Add Dashboard
3. Choose "Show in sidebar" and paste the YAML content

### Automations

Example automations are provided in [configs/homeassistant/automations.yaml](configs/homeassistant/automations.yaml). Blueprint templates are available in [configs/homeassistant/blueprints/](configs/homeassistant/blueprints/) for:
- **Motion-activated lights**: Turn on lights when person detected
- **Person detection notifications**: Get alerts on phone when people detected 
- **Zone departure alerts**: Know when someone leaves a specific area
- **Confirmable notifications**: Interactive alerts with action buttons

### Cloudflare Tunnel Setup

For secure remote access without port forwarding:

1. Create a Cloudflare Tunnel at https://one.dash.cloudflare.com/
2. Configure tunnel to forward to `homeassistant:8123`
3. Copy tunnel token to `.env` file:
   ```bash
   CLOUDFLARE_TUNNEL_TOKEN=your-token-here
   ```
4. Restart services: `docker compose restart cloudflared`
5. Access Home Assistant via your custom domain

### MQTT Configuration

The MQTT broker is pre-configured for anonymous access. For production use, edit [configs/mosquitto/mosquitto.conf](configs/mosquitto/mosquitto.conf) to add authentication:

```conf
allow_anonymous false
password_file /mosquitto/config/passwd
```

Then create password file:
```bash
docker compose exec mqtt mosquitto_passwd -c /mosquitto/config/passwd username
```

## 🛠️ Technology Stack

### Core Services
- **Home Assistant**: Main automation hub (Port 8123)
  - Version: Latest stable from ghcr.io/home-assistant
  - Integrations: Frigate, MQTT, custom dashboards
- **Frigate NVR**: AI-powered video recorder (Ports 8971 web, 8554 RTSP)
  - Version: Latest stable from ghcr.io/blakeblackshear/frigate
  - Detector: OpenVINO with YOLOv9-T model
  - Detection objects: Person, car, bicycle, motorcycle, cat, dog
- **MQTT Broker**: Mosquitto message broker (Ports 1883, 9001)
  - Version: Latest eclipse-mosquitto
  - Features: WebSocket support, persistent data
- **Cloudflare Tunnel**: Secure remote access gateway
  - Version: Latest cloudflare/cloudflared
  - No port forwarding required

### AI & Detection
- **YOLOv9-T Model**: Lightweight YOLO variant optimized for real-time detection
- **OpenVINO Runtime**: Intel's inference acceleration toolkit for CPU optimization
- **Detection Parameters**:
  - Person: 5,000-100,000 px² area, 70% confidence threshold
  - Car: 15,000-50,000 px² area, 70% confidence threshold
  - Configurable zones and object filters

### Storage Strategy
```
Local SSD (./configs/):
├── Frigate config and SQLite database (fast access)
├── Home Assistant config and database
└── MQTT configuration

Network Storage (/mnt/unas2/frigate/):
├── 14-day recording retention
├── Event clips and snapshots
└── Automated cleanup via retention policies

Temporary (tmpfs in memory):
└── 1GB cache for Frigate processing (reduces disk wear)
```

### Network Architecture
All services communicate via a dedicated Docker network (`frigate-network`) with automatic service discovery:
- Container-to-container: Use service names (e.g., `mqtt:1883`, `homeassistant:8123`)
- Host access: Use localhost with mapped ports
- External access: Via Cloudflare Tunnel (secure HTTPS)

## 📋 Management Commands

```bash
# View real-time logs
docker compose logs -f frigate        # AI detection and recording
docker compose logs -f homeassistant  # Automations and integrations
docker compose logs -f mqtt           # Message broker
docker compose logs -f cloudflared    # Remote access tunnel

# Restart after config changes
docker compose restart frigate        # After camera or detection changes
docker compose restart homeassistant  # After automation or integration changes

# Update services to latest versions
docker compose pull
docker compose up -d

# Stop all services (preserves data)
docker compose down

# View resource usage
docker stats

# Check service status
docker compose ps
```

## 🚀 Hardware Acceleration

### Current Setup: OpenVINO (Active)
The system currently uses Intel OpenVINO for CPU-accelerated AI inference, providing efficient object detection without dedicated GPU hardware.

### Upgrade Options

For even better performance with multiple cameras, you can enable GPU acceleration:

#### Intel GPU (Integrated or Arc)
Uncomment in [docker-compose.yml](docker-compose.yml):
```yaml
devices:
  - /dev/dri:/dev/dri
```

Then update [configs/frigate/config.yml](configs/frigate/config.yml):
```yaml
detectors:
  ov:
    type: openvino
    device: GPU
```

#### NVIDIA GPU (TensorRT)
Uncomment in [docker-compose.yml](docker-compose.yml):
```yaml
runtime: nvidia
environment:
  - NVIDIA_VISIBLE_DEVICES=all
```

Update detector configuration:
```yaml
detectors:
  tensorrt:
    type: tensorrt
    device: 0
```

#### Google Coral TPU
Uncomment in [docker-compose.yml](docker-compose.yml):
```yaml
devices:
  - /dev/apex_0:/dev/apex_0
privileged: true
```

Update detector configuration:
```yaml
detectors:
  coral:
    type: edgetpu
    device: usb
```

**Note**: GPU/TPU acceleration can process 5-10x more FPS compared to CPU, enabling support for many more cameras.

## 🔧 Troubleshooting

### Camera Connection Issues

**Test camera stream before adding to config:**
```bash
# Test RTSP stream
ffprobe -rtsp_transport tcp "rtsp://user:pass@camera_ip:554/stream"

# For Hikvision cameras
ffprobe -rtsp_transport tcp "rtsp://user:pass@camera_ip:554/Streaming/Channels/101"  # Main
ffprobe -rtsp_transport tcp "rtsp://user:pass@camera_ip:554/Streaming/Channels/102"  # Sub
```

**Check Frigate logs for camera errors:**
```bash
docker compose logs frigate | grep -i error
docker compose logs frigate | grep -i "camera_name"
```

**Common fixes:**
- Verify camera credentials in `.env` file
- Check camera IP address is reachable: `ping camera_ip`
- Some cameras require `rtsp_transport: tcp` in ffmpeg config
- Ensure camera supports RTSP (not all WiFi cameras do)

### Detection Not Working

**Verify detector is running:**
```bash
# Check for OpenVINO initialization
docker compose logs frigate | grep -i openvino

# Check detection stats in Frigate UI
# Navigate to http://localhost:8971 → System → Stats
```

**Adjust detection sensitivity:**
Edit [configs/frigate/config.yml](configs/frigate/config.yml):
```yaml
objects:
  filters:
    person:
      min_area: 5000      # Decrease to detect smaller objects
      threshold: 0.5      # Lower to be more sensitive (may increase false positives)
```

**Check detection zones:**
- Use Frigate web UI → Camera → Zone editor to visually adjust zones
- Ensure zone coordinates cover the area you want to monitor

### Home Assistant Integration Issues

**Frigate not appearing in HA:**
1. Check MQTT connection:
   ```bash
   docker compose logs mqtt
   docker compose logs homeassistant | grep mqtt
   ```
2. Verify MQTT config in HA: Settings → Devices & Services → MQTT
3. Check Frigate is publishing: `docker compose logs frigate | grep mqtt`

**Entities not updating:**
- Restart Home Assistant: `docker compose restart homeassistant`
- Check entity state in Developer Tools → States
- Verify MQTT broker is running: `docker compose ps mqtt`

### Storage Issues

**Check available space:**
```bash
df -h storage/                    # Local storage
df -h /mnt/unas2/frigate/        # Network storage
```

**Recordings filling up storage:**
- Adjust retention in [configs/frigate/config.yml](configs/frigate/config.yml):
  ```yaml
  record:
    retain:
      days: 7  # Reduce from 14 to 7 days
  ```
- Restart Frigate: `docker compose restart frigate`
- Manual cleanup: `rm -rf storage/recordings/old_date`

### Performance Issues

**High CPU usage:**
```bash
# Monitor container resources
docker stats

# Check detection performance in Frigate UI
# Look for: inference speed, detection FPS
```

**Solutions:**
- Reduce detection FPS: Set `fps: 5` in camera config (already optimized)
- Decrease detection resolution (already using 640x360 sub stream)
- Add motion masks to ignore high-activity areas (trees, flags)
- Consider GPU acceleration (see Hardware Acceleration section)

### Cloudflare Tunnel Issues

**Tunnel not connecting:**
```bash
docker compose logs cloudflared

# Common issues:
# - Invalid CLOUDFLARE_TUNNEL_TOKEN in .env
# - Tunnel not configured in Cloudflare dashboard
# - Tunnel route pointing to wrong service
```

**Fix steps:**
1. Verify token in `.env` file
2. Check tunnel status at https://one.dash.cloudflare.com/
3. Ensure Public Hostname points to `homeassistant:8123`
4. Restart tunnel: `docker compose restart cloudflared`

### Container Won't Start

```bash
# Check container status
docker compose ps

# View startup errors
docker compose logs <service_name>

# Common fixes:
docker compose down              # Stop all
docker compose pull              # Update images
docker compose up -d             # Start fresh
```

## 🔨 Development

### Project Structure
```
home-aide/
├── docker-compose.yml              # Service orchestration
├── .env                            # Environment configuration (create from .env.example)
├── configs/                        # All service configurations
│   ├── frigate/
│   │   ├── config.yml             # Main Frigate config (cameras, detectors, objects)
│   │   ├── backup_config.yaml     # Config backup
│   │   └── model_cache/           # AI model files (YOLOv9-T)
│   ├── homeassistant/
│   │   ├── configuration.yaml     # Main HA config
│   │   ├── automations.yaml       # Automation rules
│   │   ├── blueprints/            # Reusable automation templates
│   │   ├── dashboards/            # Pre-built dashboard configs
│   │   └── custom_components/     # Frigate & HACS integrations
│   └── mosquitto/
│       └── mosquitto.conf         # MQTT broker configuration
├── storage/                        # Persistent data (gitignored)
│   ├── homeassistant/             # HA database and logs
│   ├── mosquitto/                 # MQTT data and logs
│   ├── clips/                     # Event-triggered video clips
│   └── recordings/                # Continuous recordings (on network storage)
└── README.md                       # This file
```

### Configuration Workflow

1. **Edit Configuration Files**
   ```bash
   # Frigate: Camera and detection settings
   nano configs/frigate/config.yml
   
   # Home Assistant: Automations and integrations
   nano configs/homeassistant/automations.yaml
   
   # MQTT: Broker settings
   nano configs/mosquitto/mosquitto.conf
   ```

2. **Validate Configuration**
   - **Frigate**: Check web UI at http://localhost:8971 for config validation
   - **Home Assistant**: Use Configuration → Server Controls → Check Configuration
   - **MQTT**: Test with `mosquitto_pub` and `mosquitto_sub` commands

3. **Apply Changes**
   ```bash
   # Restart specific service
   docker compose restart frigate
   docker compose restart homeassistant
   
   # Or restart all
   docker compose restart
   ```

4. **Monitor Logs**
   ```bash
   docker compose logs -f frigate
   # Look for "INFO" messages confirming changes
   ```

### Adding Custom Automations

Create in [configs/homeassistant/automations.yaml](configs/homeassistant/automations.yaml):
```yaml
- id: '1234567890'
  alias: Person Detected at Night
  trigger:
    - platform: state
      entity_id: binary_sensor.hikvision_outdoor_person_occupancy
      to: 'on'
  condition:
    - condition: sun
      after: sunset
  action:
    - service: light.turn_on
      entity_id: light.outdoor_lights
    - service: notify.mobile_app
      data:
        message: Person detected outside
        title: Security Alert
```

### Using Blueprints

1. Find blueprint in [configs/homeassistant/blueprints/automation/](configs/homeassistant/blueprints/automation/)
2. In Home Assistant: Settings → Automations → Create Automation → Use Blueprint
3. Select blueprint and configure parameters

### Environment Variables

Key variables in `.env`:
```bash
# Timezone
TZ=America/New_York

# Camera credentials (used in Frigate config)
FRIGATE_CAMERA_USERNAME=admin
FRIGATE_CAMERA_PASSWORD=your-password
FRIGATE_CAMERA_IP=192.168.1.100

# RTSP password (for Frigate's RTSP restream)
FRIGATE_RTSP_PASSWORD=your-rtsp-password

# Cloudflare Tunnel (optional)
CLOUDFLARE_TUNNEL_TOKEN=your-tunnel-token
```

### Testing Changes

**Test camera streams:**
```bash
# Using ffprobe
ffprobe -rtsp_transport tcp "rtsp://user:pass@ip:554/stream"

# Using ffplay (visual test)
ffplay -rtsp_transport tcp "rtsp://user:pass@ip:554/stream"

# Using Frigate's debug endpoint
curl http://localhost:8971/api/config
```

**Test MQTT:**
```bash
# Subscribe to Frigate events
docker compose exec mqtt mosquitto_sub -t "frigate/#" -v

# Publish test message
docker compose exec mqtt mosquitto_pub -t "frigate/test" -m "hello"
```

**Test Home Assistant API:**
```bash
# Get state of camera entity
curl http://localhost:8123/api/states/camera.hikvision_outdoor

# Trigger automation (requires long-lived token)
curl -H "Authorization: Bearer YOUR_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"entity_id": "automation.person_detected"}' \
     http://localhost:8123/api/services/automation/trigger
```

### Backup & Restore

**Backup configuration:**
```bash
# Create timestamped backup
tar -czf backup-$(date +%Y%m%d).tar.gz configs/ .env

# Backup to remote location
rsync -av configs/ user@backup-server:/backups/home-aide/
```

**Restore configuration:**
```bash
# Extract backup
tar -xzf backup-20260208.tar.gz

# Restart services
docker compose restart
```

### Debugging Tips

- **Enable debug logging in Frigate**: Add to config.yml:
  ```yaml
  logger:
    default: info
    logs:
      frigate.record: debug
  ```
- **Home Assistant logs**: Available at http://localhost:8123/config/logs
- **Live camera debug**: Frigate UI → Camera → Debug
- **MQTT traffic**: Use MQTT Explorer or `mosquitto_sub -t "#" -v`

## ⚡ Performance & Optimization

### Current Performance Metrics

The system is optimized for efficient operation:
- **Detection Inference**: ~50-100ms per frame with OpenVINO on CPU
- **Detection FPS**: 5 FPS (configurable, balanced for CPU usage)
- **Recording Resolution**: 4K UHD (3840x2160) main stream
- **Detection Resolution**: 640x360 sub stream (reduces processing load)
- **Memory Usage**: ~1GB tmpfs cache for optimal performance
- **Storage**: Dual-tier (SSD for config/DB, network storage for recordings)

### Optimization Tips

**Reduce CPU usage:**
```yaml
# In configs/frigate/config.yml
cameras:
  your_camera:
    detect:
      fps: 5          # Lower FPS = less CPU (5 is already optimal)
    motion:
      mask:           # Ignore high-motion areas
        - 0,0,100,100,100,0  # Trees, flags, etc.
```

**Improve detection accuracy:**
```yaml
objects:
  filters:
    person:
      min_area: 5000      # Adjust based on camera distance
      max_area: 100000    # Prevent detecting entire screen
      threshold: 0.7      # Higher = fewer false positives
```

**Optimize storage:**
```yaml
record:
  retain:
    days: 7             # Reduce from 14 if storage limited
    mode: motion        # Only keep motion segments (not 24/7)
```

**Multi-camera considerations:**
- Each camera adds ~10-20% CPU usage (detect stream)
- Use sub-streams for all cameras (640x360 or similar)
- Consider GPU acceleration for 4+ cameras
- Limit detection zones to areas of interest

### Resource Requirements by Camera Count

| Cameras | CPU Usage | RAM Usage | GPU Recommended |
|---------|-----------|-----------|-----------------|
| 1-2     | 20-40%    | 2-4 GB    | No              |
| 3-4     | 40-60%    | 4-6 GB    | Optional        |
| 5-8     | 60-80%    | 6-8 GB    | Yes             |
| 9+      | 80%+      | 8+ GB     | Required        |

*Based on Intel i5/i7 CPU with OpenVINO, 640x360 detection streams at 5 FPS*

## 🎯 Features & Capabilities

### Object Detection
- **Supported Objects**: Person, car, bicycle, motorcycle, bus, cat, dog
- **Detection Zones**: Define areas for targeted monitoring
- **Motion Masks**: Ignore areas with false triggers (trees, flags)
- **Confidence Thresholds**: Configurable per object type
- **Size Filters**: Min/max area to reduce false positives

### Recording Features
- **24/7 Recording**: Continuous recording with motion-based retention
- **Event Clips**: Automatic clips for detected events
- **Snapshots**: Still images with bounding boxes
- **Retention Policies**: 14-day default (configurable)
- **Export**: Download clips and snapshots via web UI

### Home Assistant Integration
- **Camera Entities**: Live feeds in HA dashboards
- **Binary Sensors**: Person/object occupancy sensors
- **Switches**: Enable/disable detection per camera
- **Events**: Real-time detection events for automations
- **Services**: Trigger recordings, snapshots, clips
- **Statistics**: Detection counts, camera stats

### Web Interface Features
- **Live View**: Real-time streams from all cameras
- **Timeline**: Visual timeline of events and recordings
- **Events Browser**: Search and filter detection events
- **System Stats**: CPU, GPU, detector performance
- **Configuration**: Live config validation and reload
- **Debug View**: Frame-by-frame detection visualization

## 🔐 Security Considerations

### Production Deployment Checklist

- [ ] Enable MQTT authentication (edit `mosquitto.conf`)
- [ ] Use strong passwords in `.env` file
- [ ] Secure Cloudflare Tunnel with Access policies
- [ ] Use HTTPS for external access (handled by Cloudflare)
- [ ] Restrict Home Assistant to trusted networks (if not using tunnel)
- [ ] Enable Home Assistant authentication
- [ ] Regularly update Docker images: `docker compose pull`
- [ ] Backup configuration files regularly
- [ ] Monitor logs for unauthorized access attempts

### Network Security

**MQTT Authentication:**
```bash
# Create password file
docker compose exec mqtt mosquitto_passwd -c /mosquitto/config/passwd admin
docker compose restart mqtt
```

**Home Assistant Access:**
- Use Cloudflare Tunnel for remote access (encrypted, no open ports)
- Enable two-factor authentication in HA
- Use long-lived access tokens for API access only

**Frigate Security:**
- Web UI has basic authentication (set admin password on first login)
- RTSP restream password set via `FRIGATE_RTSP_PASSWORD` env var
- Not directly exposed to internet (access via HA or Cloudflare Tunnel)

## Contributing

Contributions are welcome! Here's how you can help:

### Ways to Contribute
- 🐛 Report bugs and issues
- 💡 Suggest new features or improvements
- 📝 Improve documentation
- 🔧 Submit bug fixes or enhancements
- 🎨 Create new dashboard templates
- 🤖 Add automation blueprints

### Development Process
1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Make your changes and test thoroughly
4. Commit with clear messages: `git commit -m 'feat: add amazing feature'`
5. Push to your fork: `git push origin feature/amazing-feature`
6. Submit a pull request with detailed description

### Commit Message Convention
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `refactor:` Code refactoring
- `chore:` Maintenance tasks

### Testing Guidelines
- Test all configuration changes locally
- Verify services start successfully: `docker compose up -d`
- Check logs for errors: `docker compose logs`
- Validate camera streams and detection
- Test automations in Home Assistant

## ❓ FAQ

### General Questions

**Q: How much storage do I need for recordings?**  
A: Depends on camera count, resolution, and retention. Example:
- 1 camera, 4K recording, 14-day retention: ~500GB-1TB
- 1 camera, 1080p recording, 7-day retention: ~100-200GB
- Calculate: [Frigate storage calculator](https://docs.frigate.video/configuration/record#calculating-required-shm-size)

**Q: Can I use WiFi cameras?**  
A: Yes, if they support RTSP streams. However, wired cameras are recommended for reliability. WiFi cameras may experience:
- Stream drops during interference
- Higher latency
- Bandwidth congestion with multiple cameras

**Q: Does this work on Raspberry Pi?**  
A: Yes, but performance varies:
- **Pi 4 (4GB+)**: 1-2 cameras with CPU detector
- **Pi 5**: 2-3 cameras with CPU detector
- **With Coral TPU**: 4-6 cameras (highly recommended for Pi)
- Consider using 1080p or lower resolution for detection

**Q: Can I access this remotely?**  
A: Yes, via Cloudflare Tunnel (included). Provides secure HTTPS access without port forwarding or dynamic DNS. No additional costs if you have a free Cloudflare account.

### Camera Questions

**Q: My camera doesn't show up in Frigate**  
A: Check:
1. Camera is reachable: `ping camera_ip`
2. RTSP stream works: `ffprobe rtsp://user:pass@ip:554/stream`
3. Credentials are correct in `.env` file
4. Camera is enabled in config: `enabled: true`
5. Check Frigate logs: `docker compose logs frigate | grep camera_name`

**Q: How do I find my camera's RTSP URL?**  
A: Common formats by brand:
- **Hikvision**: `rtsp://ip:554/Streaming/Channels/101` (main), `102` (sub)
- **Dahua**: `rtsp://ip:554/cam/realmonitor?channel=1&subtype=0`
- **Reolink**: `rtsp://ip:554/h264Preview_01_main` (main), `_sub` (sub)
- **Wyze (with RTSP firmware)**: `rtsp://ip:8554/unicast`
- **Amcrest**: `rtsp://ip:554/cam/realmonitor?channel=1&subtype=0`
- Check camera manufacturer documentation or use ONVIF Device Manager

**Q: Detection is too sensitive / not sensitive enough**  
A: Adjust in [configs/frigate/config.yml](configs/frigate/config.yml):
```yaml
objects:
  filters:
    person:
      threshold: 0.8      # Higher = less sensitive (fewer false positives)
      min_area: 10000     # Higher = ignore smaller objects
```

### Performance Questions

**Q: High CPU usage, how to reduce?**  
A:
1. Lower detection FPS: `fps: 5` → `fps: 3`
2. Use sub-streams for detection (already configured)
3. Add motion masks for busy areas
4. Enable hardware acceleration (Intel GPU, Coral TPU)
5. Reduce number of tracked objects

**Q: Can I use a GPU to speed things up?**  
A: Yes! See [Hardware Acceleration](#-hardware-acceleration) section. Options:
- Intel GPU (OpenVINO): 3-5x faster
- NVIDIA GPU (TensorRT): 5-10x faster
- Google Coral TPU: 10-15x faster, lowest CPU usage

**Q: Recordings are taking up too much space**  
A: Reduce retention period:
```yaml
record:
  retain:
    days: 7    # Reduce from 14
```
Or switch to motion-only mode (already configured).

### Integration Questions

**Q: Frigate not showing in Home Assistant**  
A: Ensure:
1. MQTT broker is running: `docker compose ps mqtt`
2. MQTT configured in HA: Settings → Integrations → MQTT
3. Frigate connected to MQTT: `docker compose logs frigate | grep mqtt`
4. Wait 1-2 minutes for MQTT discovery
5. Restart HA if needed: `docker compose restart homeassistant`

**Q: How do I create automations?**  
A: Three ways:
1. Use pre-built blueprints in `configs/homeassistant/blueprints/`
2. Create via HA UI: Settings → Automations → Create Automation
3. Edit `configs/homeassistant/automations.yaml` directly

**Q: Can I view cameras on my phone?**  
A: Yes!
1. Install Home Assistant mobile app
2. Configure Cloudflare Tunnel for external access
3. Access dashboards in the app
4. Enable notifications for detection events

### Troubleshooting Questions

**Q: "Error: Unable to connect to MQTT"**  
A:
```bash
# Check MQTT is running
docker compose ps mqtt

# Restart MQTT
docker compose restart mqtt

# Check MQTT logs
docker compose logs mqtt
```

**Q: Camera shows "No Signal" or blank screen**  
A:
1. Test stream: `ffprobe rtsp://user:pass@ip:554/stream`
2. Check camera is online: `ping camera_ip`
3. Verify credentials in `.env`
4. Try `rtsp_transport: tcp` in camera config
5. Check network firewall rules

**Q: Cloudflare Tunnel not working**  
A:
1. Verify `CLOUDFLARE_TUNNEL_TOKEN` in `.env`
2. Check tunnel status: `docker compose logs cloudflared`
3. Verify tunnel configuration at https://one.dash.cloudflare.com/
4. Ensure tunnel points to `homeassistant:8123`
5. Check Public Hostname is active

## 📚 Additional Resources

### Official Documentation
- [Home Assistant Docs](https://www.home-assistant.io/docs/) - Complete HA documentation
- [Frigate Documentation](https://docs.frigate.video/) - Comprehensive Frigate guide
- [Mosquitto MQTT](https://mosquitto.org/documentation/) - MQTT broker docs
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) - Tunnel setup guide

### Useful Tools
- [ONVIF Device Manager](https://sourceforge.net/projects/onvifdm/) - Discover camera RTSP URLs
- [MQTT Explorer](http://mqtt-explorer.com/) - Visualize MQTT messages
- [VLC Player](https://www.videolan.org/) - Test RTSP streams
- [Frigate Storage Calculator](https://docs.frigate.video/configuration/record#calculating-required-shm-size) - Estimate storage needs

### Community
- [Home Assistant Community](https://community.home-assistant.io/) - Community forum
- [Frigate Discussions](https://github.com/blakeblackshear/frigate/discussions) - Frigate community
- [r/homeassistant](https://www.reddit.com/r/homeassistant/) - Reddit community
- [r/frigate](https://www.reddit.com/r/frigate/) - Frigate subreddit

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Home Assistant](https://www.home-assistant.io/) - Amazing open-source home automation
- [Frigate](https://frigate.video/) - Brilliant AI-powered NVR
- [Eclipse Mosquitto](https://mosquitto.org/) - Reliable MQTT broker
- [OpenVINO](https://docs.openvino.ai/) - Intel's AI inference toolkit
- [Cloudflare](https://www.cloudflare.com/) - Secure tunnel infrastructure

---

**Made with ❤️ for smart home enthusiasts**

For issues and questions, please [open an issue](../../issues) on GitHub.