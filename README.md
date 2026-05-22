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
    Internet[🌐 Internet]
    CloudFlare[☁️ Cloudflare Tunnel]
    Camera[📹 Hikvision Camera 8MP/4K]
    
    HA[🏠 Home Assistant :8123]
    HADash[📊 Security Dashboards]
    
    Frigate[🔍 Frigate NVR :8971]
    OpenVINO[🧠 OpenVINO YOLOv9-T]
    Detection[🎯 Object Detection]
    
    MQTT[📡 MQTT Broker :1883]
    
    LocalConfig[💾 Local Config/DB]
    NetworkStorage[🗄️ Network Storage]
    HAStorage[💾 HA Storage]
    
    Internet --> CloudFlare
    CloudFlare --> HA
    Camera -->|RTSP 4K + Sub| Frigate
    
    Frigate --> OpenVINO
    OpenVINO --> Detection
    Detection --> Frigate
    
    Frigate <-->|Events| MQTT
    HA <-->|Discovery| MQTT
    HA --> HADash
    
    Frigate --> LocalConfig
    Frigate --> NetworkStorage
    HA --> HAStorage
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
  - Objects tracked: Person, car, bicycle, motorcycle
- **AI Model**: YOLOv9-T optimized for Intel CPUs via OpenVINO
- **Storage**: Network storage mounted at `/mnt/unas2/frigate` for recordings
- **Retention**: 14 days for recordings, detections, and snapshots
- **Hardware Acceleration**: Intel GPU with OpenVINO (active)

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
```

**Tip**: Use the Frigate web UI at http://localhost:8971 to visually define detection zones by clicking on the camera feed.

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

## 🚀 Hardware Acceleration

### Current Setup: OpenVINO (Active)
The system currently uses Intel OpenVINO for CPU-accelerated AI inference, providing efficient object detection without dedicated GPU hardware.

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
      entity_id: binary_sensor.home_garage_main_person_occupancy
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

### Resource Requirements by Camera Count

| Cameras | CPU Usage | RAM Usage | GPU Recommended |
|---------|-----------|-----------|-----------------|
| 1-2     | 20-40%    | 2-4 GB    | No              |
| 3-4     | 40-60%    | 4-6 GB    | Optional        |
| 5-8     | 60-80%    | 6-8 GB    | Yes             |
| 9+      | 80%+      | 8+ GB     | Required        |

*Based on Intel i5/i7 CPU with OpenVINO, 640x360 detection streams at 5 FPS*

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