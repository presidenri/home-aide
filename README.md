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
- [🚀 Hardware Acceleration](#-hardware-acceleration)
- [🔧 Troubleshooting](#-troubleshooting)
- [🔨 Development](#-development)
- [🔐 Security Considerations](#-security-considerations)
- [Contributing](#contributing)
- [📚 Additional Resources](#-additional-resources)

---

## ✨ What's New

- ⬆️ **Frigate 0.18.2 & Home Assistant 2026.10.4**: Latest releases, including Frigate's 0.18.1/0.18.2 security fixes
- 📹 **Two Cameras**: Hikvision DS-2CD2387G3-LI2UY (8MP) and TP-Link Tapo C319 (4MP), both dual-stream
- 🔁 **go2rtc Restreaming**: Each camera is pulled once by go2rtc and shared with Frigate and live view
- 🔢 **License Plate Recognition**: Frigate LPR enabled (models downloaded automatically on first start)
- 🔧 **Per-Camera Credentials**: Each camera has its own `.env` variables (`FRIGATE_HIKVISION1_*`, `FRIGATE_TAPO1_*`)

## Features

- 🏠 **Home Assistant**: Complete home automation platform with web UI and custom security dashboards
- 📹 **Frigate NVR**: AI-powered network video recorder with real-time object detection using YOLOv9-T
- 🧠 **OpenVINO Detector**: CPU-optimized AI inference for efficient object detection
- 📡 **MQTT Integration**: Seamless communication between services with automatic discovery
- 🔍 **Object Detection**: Person, car, bicycle and motorcycle detection (cat/dog available but disabled)
- 🔢 **License Plate Recognition**: Plate detection and OCR on detected cars
- 💾 **Smart Recording**: Automatic retention policies (14 days) with event-triggered clips and snapshots
- 🌐 **Remote Access**: Secure access via Cloudflare Tunnel without exposing ports
- 📱 **Mobile Ready**: Responsive web interfaces accessible from any device
- 🔧 **Hardware Acceleration**: Support for Intel OpenVINO (active), NVIDIA TensorRT, Google Coral TPU

## 🏗️ Architecture

```mermaid
graph TB
    Internet[🌐 Internet]
    CloudFlare[☁️ Cloudflare Tunnel]
    Camera[📹 Hikvision 8MP]
    Camera2[📹 Tapo C319 4MP]
    
    HA[🏠 Home Assistant :8123]
    HADash[📊 Security Dashboards]
    
    Go2rtc[🔁 go2rtc restream :8554]
    Frigate[🔍 Frigate NVR :8971]
    OpenVINO[🧠 OpenVINO YOLOv9-T]
    Detection[🎯 Object Detection]
    
    MQTT[📡 MQTT Broker :1883]
    
    LocalConfig[💾 Local Config/DB]
    NetworkStorage[🗄️ Network Storage]
    HAStorage[💾 HA Storage]
    
    Internet --> CloudFlare
    CloudFlare --> HA
    Camera -->|RTSP main + sub| Go2rtc
    Camera2 -->|RTSP main + sub| Go2rtc
    Go2rtc --> Frigate
    
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

1. **Camera Streaming**: Each camera sends two RTSP streams (high-res for recording, 640x360 for detection). go2rtc (built into Frigate) connects to each camera once and restreams it locally at `rtsp://127.0.0.1:8554/<stream>`.
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
   - `FRIGATE_HIKVISION1_CAMUSER` / `_CAMPASS` / `_CAMIP`: Hikvision camera credentials and IP
   - `FRIGATE_TAPO1_CAMUSER` / `_CAMPASS` / `_CAMIP`: Tapo C319 camera account and IP
   - `CLOUDFLARE_TUNNEL_TOKEN`: (Optional) Cloudflare Tunnel token

   See [Environment Variables](#environment-variables) for the full list.

3. **Provide the detection model**

   Frigate is configured to load `configs/frigate/model_cache/yolov9-t-converted.onnx` (640x640, NCHW, float). It is tracked in this repository, so a normal clone already has it. To build your own, follow the YOLOv9 export steps in the [Frigate object detector docs](https://docs.frigate.video/configuration/object_detectors/).

   The license plate and OCR models in `model_cache/` are **not** tracked; Frigate downloads them on first start when `lpr` is enabled.

4. **Start the services**
   ```bash
   docker compose up -d
   ```

5. **Access the interfaces**
   - Home Assistant: http://localhost:8123 (initial setup required)
   - Frigate NVR: http://localhost:8971 (check live detection and events)
   - MQTT WebSocket: http://localhost:9001
   - Remote Access: Your custom Cloudflare domain (if configured)

## ⚙️ Configuration

### Current Setup

The system is currently configured with:
- **Cameras**:
  | Frigate camera | Model | Record stream | Detect stream |
  |---|---|---|---|
  | `home_garage_main` | Hikvision DS-2CD2387G3-LI2UY (8MP) | `Streaming/Channels/101` | `Streaming/Channels/102` (640x360) |
  | `c319_garage_main` | TP-Link Tapo C319 (4MP) | `stream1` | `stream2` (640x360) |
- **Objects tracked**: Person, car, bicycle, motorcycle
- **License plate recognition**: Enabled
- **AI Model**: YOLOv9-T via OpenVINO (`device: AUTO`, 2 threads)
- **Video decoding**: VAAPI through the WSL2 `d3d12` driver
- **Storage**: Network storage mounted at `/mnt/unas2/frigate` for recordings
- **Retention**: 14 days for alerts, detections, motion recordings and snapshots; no continuous recording

### Adding Additional Cameras

Cameras are added in two places in [configs/frigate/config.yml](configs/frigate/config.yml): a go2rtc stream that connects to the camera, and a Frigate camera that reads from go2rtc.

1. Add credentials to `.env` using a new prefix (variables must start with `FRIGATE_` to be usable in the config):
   ```bash
   FRIGATE_FRONTDOOR1_CAMUSER=admin
   FRIGATE_FRONTDOOR1_CAMPASS=your-camera-password
   FRIGATE_FRONTDOOR1_CAMIP=192.168.1.101
   ```
2. Pass them to the container in the `frigate.environment` list in [docker-compose.yml](docker-compose.yml).
3. Add the streams and the camera:
   ```yaml
   go2rtc:
     streams:
       front_door_main:
         - rtsp://{FRIGATE_FRONTDOOR1_CAMUSER}:{FRIGATE_FRONTDOOR1_CAMPASS}@{FRIGATE_FRONTDOOR1_CAMIP}:554/main_stream
       front_door_sub:
         - rtsp://{FRIGATE_FRONTDOOR1_CAMUSER}:{FRIGATE_FRONTDOOR1_CAMPASS}@{FRIGATE_FRONTDOOR1_CAMIP}:554/sub_stream

   cameras:
     front_door_main:
       enabled: true
       ffmpeg:
         inputs:
           - path: rtsp://127.0.0.1:8554/front_door_main
             input_args: preset-rtsp-restream
             roles: [record]
           - path: rtsp://127.0.0.1:8554/front_door_sub
             input_args: preset-rtsp-restream
             roles: [detect]
       detect:
         width: 640
         height: 360
         fps: 5
   ```
4. Recreate the container so the new variables are picked up: `docker compose up -d frigate`

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

The MQTT broker requires a username and password ([mosquitto.conf](configs/mosquitto/mosquitto.conf) sets `allow_anonymous false`). Users live in `configs/mosquitto/passwd`, which is gitignored and has to be created on each install:

```bash
set -a; . ./.env; set +a
docker compose exec -T mqtt sh -c 'umask 077
  mosquitto_passwd -c -b /mosquitto/config/passwd "$1" "$2"
  mosquitto_passwd -b /mosquitto/config/passwd "$3" "$4"
  chown 1000:1000 /mosquitto/config/passwd' _ \
  "$FRIGATE_MQTT_USER" "$FRIGATE_MQTT_PASSWORD" "$HA_MQTT_USER" "$HA_MQTT_PASSWORD"
docker compose restart mqtt
```

The broker runs as UID 1000 in this setup (`docker logs mosquitto` shows `running mosquitto as user id`), so `passwd` must be owned by that UID; if Mosquitto keeps restarting with `Unable to open pwfile`, the ownership is wrong.

- **Frigate** reads `FRIGATE_MQTT_USER` / `FRIGATE_MQTT_PASSWORD` from the environment (see the `mqtt:` section of [config.yml](configs/frigate/config.yml)).
- **Home Assistant** stores its MQTT credentials in the UI, not in YAML: Settings → Devices & services → MQTT → ⋮ → Reconfigure, then enter `HA_MQTT_USER` / `HA_MQTT_PASSWORD`.

### RTSP Restream Authentication

go2rtc's restream on port `8554` requires `FRIGATE_RTSP_USER` / `FRIGATE_RTSP_PASSWORD` for every client except localhost, so Frigate's own `rtsp://127.0.0.1:8554/...` inputs need no credentials.

Home Assistant connects from another container, so the Frigate integration needs the credentials too: Settings → Devices & services → Frigate → Configure → **RTSP URL template**:

```
rtsp://<FRIGATE_RTSP_USER>:<FRIGATE_RTSP_PASSWORD>@frigate:8554/{{ name }}
```

Without it, camera streams in Home Assistant and the companion app fail to open (snapshots keep working, they use Frigate's API).

## 🛠️ Technology Stack

### Core Services
- **Home Assistant**: Main automation hub (Port 8123)
  - Version: `2026.10.4` (pinned in `docker-compose.yml`)
  - Integrations: Frigate (HACS, 5.15.6), MQTT, custom dashboards
- **Frigate NVR**: AI-powered video recorder (Ports 8971 web, 8554 RTSP)
  - Version: `0.18.2` (pinned in `docker-compose.yml`)
  - Detector: OpenVINO with YOLOv9-T model
  - Detection objects: Person, car, bicycle, motorcycle
- **MQTT Broker**: Mosquitto message broker (Ports 1883, 9001)
  - Version: `2.1.2-alpine` (pinned in `docker-compose.yml`)
  - Features: WebSocket support, persistent data
- **Cloudflare Tunnel**: Secure remote access gateway
  - Version: `2026.10.7` (pinned in `docker-compose.yml`)
  - No port forwarding required

### AI & Detection
- **YOLOv9-T Model**: Lightweight YOLO variant optimized for real-time detection
- **OpenVINO Runtime**: Intel's inference acceleration toolkit for CPU optimization
- **Detection Parameters** (all tracked objects):
  - `min_score: 0.65` (minimum score for a single detection)
  - `threshold: 0.75` (score needed to count as a real object; `0.85` for bicycle)
  - Zones are not configured yet; add them in the Frigate UI

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

### Current Setup: OpenVINO + VAAPI on WSL2 (Active)
- **Object detection**: OpenVINO with `device: AUTO`, which uses the GPU when available and falls back to the CPU.
- **Video decoding**: FFmpeg uses VAAPI (`-hwaccel vaapi`). On WSL2 this goes through Mesa's `d3d12` driver to the Windows GPU, which is why `docker-compose.yml` passes `/dev/dxg`, mounts `/usr/lib/wsl` and sets `LIBVA_DRIVER_NAME=d3d12`.

On a native Linux host with an Intel GPU, the WSL-specific devices, mounts and environment variables can be removed; `/dev/dri` is enough.

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
│   │   ├── backup_config.yaml     # Previous config, written by Frigate (gitignored)
│   │   └── model_cache/           # YOLOv9-T model (tracked) + auto-downloaded LPR/OCR models (ignored)
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
│   └── mosquitto/                 # MQTT data and logs
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

[configs/homeassistant/automations.yaml](configs/homeassistant/automations.yaml) ships with only a commented-out example, so no automations are active by default. Create your own there or in the Home Assistant UI, for example:
```yaml
- id: '1234567890'
  alias: Person Detected at Night
  triggers:
    - trigger: state
      entity_id: binary_sensor.home_garage_main_person_occupancy
      to: 'on'
  conditions:
    - condition: sun
      after: sunset
  actions:
    - action: light.turn_on
      target:
        entity_id: light.outdoor_lights
    - action: notify.mobile_app_your_phone
      data:
        message: Person detected outside
        title: Security Alert
```

### Environment Variables

Key variables in `.env`:
```bash
# Timezone
TZ=America/New_York

# Hikvision DS-2CD2387G3-LI2UY (home_garage_main)
FRIGATE_HIKVISION1_CAMUSER=admin
FRIGATE_HIKVISION1_CAMPASS=your-camera-password
FRIGATE_HIKVISION1_CAMIP=192.168.1.100

# TP-Link Tapo C319 (c319_garage_main) - camera account from the Tapo app
FRIGATE_TAPO1_CAMUSER=your-tapo-camera-account
FRIGATE_TAPO1_CAMPASS=your-tapo-camera-password
FRIGATE_TAPO1_CAMIP=192.168.1.101

# go2rtc restream credentials (port 8554)
FRIGATE_RTSP_USER=frigate-rtsp
FRIGATE_RTSP_PASSWORD=your-rtsp-password

# MQTT users - Frigate reads its pair, Home Assistant's pair is entered in the HA UI
FRIGATE_MQTT_USER=frigate
FRIGATE_MQTT_PASSWORD=your-frigate-mqtt-password
HA_MQTT_USER=homeassistant
HA_MQTT_PASSWORD=your-ha-mqtt-password

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

- [ ] Create the MQTT users in `configs/mosquitto/passwd` (see [MQTT Configuration](#mqtt-configuration))
- [ ] Use strong passwords in `.env` file
- [ ] Secure Cloudflare Tunnel with Access policies
- [ ] Use HTTPS for external access (handled by Cloudflare)
- [ ] Restrict Home Assistant to trusted networks (if not using tunnel)
- [ ] Enable Home Assistant authentication
- [ ] Regularly update Docker images: bump the pinned tags in `docker-compose.yml`, then `docker compose pull && docker compose up -d`
- [ ] Backup configuration files regularly
- [ ] Monitor logs for unauthorized access attempts

### Network Security

**Home Assistant Access:**
- Use Cloudflare Tunnel for remote access (encrypted, no open ports)
- Enable two-factor authentication in HA
- Use long-lived access tokens for API access only

**Frigate Security:**
- Web UI has basic authentication (set admin password on first login)
- The go2rtc restream on port `8554` requires `FRIGATE_RTSP_USER` / `FRIGATE_RTSP_PASSWORD` for non-localhost clients
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

### Secret Check (pre-commit hook)
The repository ships a pre-commit hook that blocks committing `.env`, `passwd`, databases, and literal credentials in `configs/**/*.yml|yaml` (credentials must use `{FRIGATE_*}` env vars or `!secret`). Enable it once per clone:
```bash
git config core.hooksPath .githooks
```
Frigate's UI config editor may write resolved values back into `config.yml`, so check `git diff` before committing config changes.

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