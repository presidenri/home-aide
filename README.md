# Home Aide

A complete **Home Assistant + Frigate NVR** setup using Docker Compose. This project provides a full home automation hub with AI-powered surveillance, real-time object detection, person/vehicle tracking, and intelligent recording capabilities all integrated in one stack.

## Features

- 🏠 **Home Assistant**: Complete home automation platform with web UI
- 📹 **Frigate NVR**: AI-powered network video recorder with real-time object detection
- 📡 **MQTT Integration**: Seamless communication between services
- 🔍 **Object Detection**: Person, vehicle, and pet detection with configurable zones
- 💾 **Smart Recording**: Automatic retention policies and event-triggered clips
- 📱 **Mobile Ready**: Web interfaces accessible from any device
- 🔧 **Hardware Acceleration**: Support for Intel OpenVINO, NVIDIA TensorRT, Google Coral TPU

## Quick Start

### Prerequisites

- Docker and Docker Compose
- At least 4GB RAM (8GB+ recommended for multiple cameras)
- Storage space for recordings (varies by camera count and retention)

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd home-aide
   ```

2. **Configure environment**
   ```bash
   cp .env.example .env
   nano .env  # Edit timezone and passwords
   ```

3. **Start the services**
   ```bash
   docker compose up -d
   ```

4. **Access the interfaces**
   - Home Assistant: http://localhost:8123
   - Frigate NVR: http://localhost:8971
   - MQTT WebSocket: http://localhost:9001

## Configuration

### Adding Cameras

Edit `configs/frigate/config.yml` to add your cameras:

```yaml
cameras:
  front_door:
    ffmpeg:
      inputs:
        - path: rtsp://user:pass@camera_ip:554/stream
          roles: ["detect", "record"]
    zones:
      entry_area:
        coordinates: 100,100,200,100,200,200,100,200
```

### Home Assistant Automations

Example automations are provided in `configs/homeassistant/automations.yaml`. Blueprint templates are available in `configs/homeassistant/blueprints/` for:
- Motion-activated lights
- Person detection notifications  
- Zone departure alerts

### MQTT Configuration

The MQTT broker is pre-configured for anonymous access. For production use, edit `configs/mosquitto/mosquitto.conf` to add authentication.

## Architecture

### Services
- **homeassistant**: Main automation hub (Port 8123)
- **frigate**: AI-powered NVR (Ports 8971, 8554)
- **mqtt**: Message broker (Ports 1883, 9001)

### Storage Structure
```
storage/
├── homeassistant/     # HA database and config
├── frigate/           # Video recordings and clips
└── mosquitto/         # MQTT data and logs
```

### Network
All services communicate via a dedicated Docker network (`frigate-network`) with automatic service discovery.

## Management Commands

```bash
# View service logs
docker compose logs -f frigate
docker compose logs -f homeassistant
docker compose logs -f mqtt

# Restart after config changes
docker compose restart frigate

# Update services
docker compose pull
docker compose up -d

# Stop all services
docker compose down
```

## Hardware Acceleration

For multiple cameras, enable hardware acceleration by uncommenting the relevant sections in `docker-compose.yml`:

```yaml
# Intel GPU
devices:
  - /dev/dri:/dev/dri

# Google Coral TPU  
devices:
  - /dev/apex_0:/dev/apex_0
```

Update `config/config.yml` detector configuration accordingly.

## Troubleshooting

### Camera Connection Issues
```bash
# Test camera stream
ffprobe rtsp://user:pass@camera_ip:554/stream

# Check Frigate logs
docker compose logs frigate | grep -i error
```

### Storage Issues
- Recordings are auto-managed by retention policies
- Check available space: `df -h storage/`
- Adjust retention in `configs/frigate/config.yml`

### Performance Issues
- Monitor resource usage: `docker stats`
- Consider hardware acceleration for multiple cameras
- Adjust detection zones to reduce processing load

## Development

### File Structure
- `configs/frigate/config.yml`: Main Frigate configuration
- `configs/homeassistant/`: Home Assistant configuration files
- `configs/mosquitto/mosquitto.conf`: MQTT broker settings
- `docker-compose.yml`: Service orchestration

### Testing Changes
1. Modify configuration files
2. Restart affected services: `docker compose restart <service>`
3. Check logs for errors
4. Access web interfaces for validation

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For issues and questions:
- Check the troubleshooting section above
- Review service logs for error messages
- Consult official documentation:
  - [Home Assistant](https://www.home-assistant.io/docs/)
  - [Frigate](https://docs.frigate.video/)
  - [Mosquitto MQTT](https://mosquitto.org/documentation/)