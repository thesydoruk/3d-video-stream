# MediaMTX + SnapFeeder Auto Installer

This project provides a complete, self-contained RTSP + JPEG snapshot system using:

- **MediaMTX** for RTSP and WebRTC streaming with FFmpeg backend
- **SnapFeeder**, a Flask server that serves JPEG snapshots on-demand
- A Python-based configuration generator that automatically detects connected cameras and sets up MediaMTX accordingly

---

## 📁 Project Structure

```
project/
├── install.sh                # Full setup script
├── uninstall.sh              # Cleanup script
├── mediamtx/                 # Holds downloaded MediaMTX binary and mediamtx.yml
│   ├── mediamtx
│   └── mediamtx.yml
├── scripts/                  # All project Python logic
│   ├── generate_mediamtx_config.py
│   └── snapfeeder.py
├── templates/                # Template .service files with placeholders
│   ├── mediamtx.service.template
│   └── snapfeeder.service.template
└── services/                 # Populated during install with rendered .service files
```

---

## ✅ Features

- Detects all `/dev/video*` USB cameras
- Chooses best format:
  - Prefers `mjpeg`, falls back to others
  - Picks `1280x720` if supported, else highest
  - Uses max fps for selected resolution
- Leverages **VAAPI** hardware acceleration if available
- Configures `mediamtx.yml` with:
  - Enabled: `rtsp`, `webrtc`
  - Disabled: `rtmp`, `hls`, `api`, `metrics`, `pprof`, `playback`, `srt`
  - Adds Google STUN server for WebRTC
- Snapshot server:
  - Reads mediamtx config
  - Spawns `ffmpeg` for each RTSP camera
  - Decodes using PyAV
  - JPEG-encodes via TurboJPEG only when requested
  - Dynamic endpoints: `/cam0.jpg`, `/cam1.jpg`, etc.

---

## 🚀 Installation

From the project root:

```bash
chmod +x install.sh
./install.sh
```

This will:

- Install dependencies via APT and pip (if needed)
- Download MediaMTX into `mediamtx/`
- Generate `mediamtx.yml` using `scripts/generate_mediamtx_config.py`
- Create `.service` files from `templates/` and write to `services/`
- Create systemd symlinks into `/etc/systemd/system/`
- Enable and start `mediamtx` and `snapfeeder` services
- Print available camera URLs

---

## 🔍 Snapshot Access

After installation, each camera is available via:

```
🎥 cam0:
   📡 RTSP:     rtsp://<ip>:8554/cam0
   🌐 WebRTC:   http://<ip>:8889/cam0/
   🖼️ Snapshot: http://<ip>:5050/cam0.jpg
```

---

## 🧹 Uninstallation

```bash
chmod +x uninstall.sh
./uninstall.sh
```

This will:

- Stop and disable both services
- Remove symlinks from `/etc/systemd/system/`
- Delete the `services/` and `mediamtx/` directories

---

## 🔧 Development Notes

- `generate_mediamtx_config.py` and `snapfeeder.py` use hardcoded paths relative to the project root
- No environment variables required
- All Python scripts are isolated in `scripts/`

---

## 📜 License

MIT License  
(c) 2025 Valerii Sydoruk
