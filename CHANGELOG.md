# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2026-05-22

### Added
- TP-Link Tapo C319 4MP camera support with dual-stream configuration
- License plate recognition (LPR) with YOLOv9 license plate model
- PaddleOCR models for text detection and recognition
- Semantic search capability (disabled by default)
- Face recognition support (disabled by default)
- Bird classification support (disabled by default)
- Timestamp overlay configuration (`timestamp_style` format)
- Unified `home_cameras.yaml` dashboard for all camera feeds
- Multi-camera environment variable structure (HIKVISION1, TAPO1 prefixes)

### Changed
- **BREAKING**: Upgraded Frigate from v0.16 to v0.17
- **BREAKING**: Restructured recording configuration with new keys:
  - `record.alerts.retain` for alert retention
  - `record.detections.retain` for detection retention  
  - `record.continuous` for 24/7 recording
  - `record.motion` for motion-based recording
- **BREAKING**: Environment variable naming scheme updated:
  - `FRIGATE_CAMERA_USERNAME` → `FRIGATE_HIKVISION1_CAMUSER`
  - `FRIGATE_CAMERA_PASSWORD` → `FRIGATE_HIKVISION1_CAMPASS`
  - `FRIGATE_CAMERA_IP` → `FRIGATE_HIKVISION1_CAMIP`
- Upgraded Home Assistant from `stable` to version `2026.5`
- Improved bicycle detection threshold from 0.90 to 0.85
- Consolidated all camera dashboards into single `home_cameras` dashboard
- Dashboard title changed from "Security Cameras" to "Home Cameras"
- OpenVINO thread limit reduced to 2 for better CPU efficiency

### Removed
- `security_dashboard.yaml` - replaced by unified dashboard
- `simple_cameras.yaml` - replaced by unified dashboard
- `working_cameras.yaml` - replaced by unified dashboard
- Continuous recording (set to 0 days retention)
- Tracked objects: `dog` and `cat` (disabled in current config)

### Fixed
- SQLite database locking issues with network storage paths
- VAAPI hardware acceleration configuration format
- Recording retention policy structure for Frigate v0.17 compatibility

### Performance
- Motion-based recording reduces storage usage compared to continuous recording
- Optimized dual-camera setup with efficient stream reuse via go2rtc
- VAAPI hardware acceleration for ffmpeg operations

### Security
- Multi-camera credential isolation with separate environment variables per camera

## [1.1.0] - 2026-02-09

### Added
- Aspect ratio filtering for object detection to prevent person/dog misclassification
  - Person: `min_ratio: 0.3`, `max_ratio: 3.5` (vertical orientation)
  - Dog/Cat: `max_ratio: 2.0` (horizontal orientation)
- Minimum area filters for all tracked objects (person: 5000px², car: 10000px², dog/cat: 3000px²)
- Stationary object optimization (detect every 50th frame for static objects)
- Hardware acceleration auto-detection with OpenVINO AUTO mode
- Motion detection enhancements (threshold: 30, contour_area: 50, lightning suppression)
- RTSP streaming optimization with `preset-rtsp-restream`
- Comprehensive release notes documentation

### Changed
- Increased person detection confidence (`min_score: 0.65 → 0.70`)
- Increased dog detection thresholds (`min_score: 0.70 → 0.75`, `threshold: 0.90 → 0.92`)
- Increased cat detection thresholds (`min_score: 0.65 → 0.70`, `threshold: 0.90 → 0.92`)
- Reduced bicycle detection threshold (`threshold: 0.90 → 0.85`) for better detection rate
- Reduced detection FPS from 5 to 3 for better CPU efficiency
- OpenVINO detector device from `CPU` to `AUTO` (enables GPU/NPU acceleration)
- Limited detector threads to 2 (optimized for Intel N150 4-core CPU)
- Disabled contrast improvement in motion detection to save CPU cycles
- Streamlined README.md documentation

### Removed
- Unsupported object classes: `face`, `license_plate`, `package` (not in COCO-80 model)
- Redundant documentation sections in README (Management Commands, duplicate FAQ)

### Fixed
- Person/dog misclassification issue through aspect ratio constraints
- OpenVINO detector performance warnings on Intel N150 CPU
- High CPU usage during continuous detection
- False motion triggers from environmental factors

### Performance
- 40-60% reduction in CPU usage on Intel N150 (4-core) processors
- 98% reduction in CPU usage for stationary object detection
- Detection inference time improved from ~100-150ms to ~50-100ms
- 60-80% reduction in false person detection events

## [1.0.0] - 2026-02-01

### Added
- Initial release with complete Home Assistant + Frigate NVR stack
- Docker Compose orchestration for all services
- OpenVINO detector with YOLOv9-T model
- MQTT broker for service communication
- Cloudflare Tunnel for secure remote access
- Hikvision DS-2CD2387G3-LI2UY 8MP camera configuration
- Dual-stream support (4K recording + 640x360 detection)
- 14-day retention policy for recordings, detections, and snapshots
- Custom Home Assistant security dashboards
- Automation blueprints (motion light, zone notifications)
- Frigate custom component integration
- HACS (Home Assistant Community Store) support
- Environment-based configuration with .env file

### Security
- MQTT anonymous access enabled (development mode)
- Cloudflare Tunnel encrypted connection

---

## Legend

- `Added` for new features
- `Changed` for changes in existing functionality
- `Deprecated` for soon-to-be removed features
- `Removed` for now removed features
- `Fixed` for any bug fixes
- `Security` for vulnerability fixes
- `Performance` for performance improvements

[2.0.0]: https://github.com/presidenri/home-aide/compare/v1.1.0...v2.0.0
[1.1.0]: https://github.com/presidenri/home-aide/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/presidenri/home-aide/releases/tag/v1.0.0
