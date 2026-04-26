# Arduino GIGA R1 WiFi - LED Control System
(# vyucba-arduino-tutorial)
A professional IoT web interface for controlling the Arduino GIGA R1's built-in RGB LED via WiFi, featuring role-based authentication, real-time WebSocket updates, and activity logging.

![Dashboard Screenshot](images/screenshot_dashboard.png)

## 🌟 Features

- **WiFi Web Interface**: Access LED controls from any device on your network
- **Role-Based Authentication**: Admin users control LEDs, regular users view-only
- **Real-Time Updates**: WebSocket communication for instant LED state synchronization
- **Activity Logging**: Server-side history of last 10 LED state changes
- **Responsive Design**: Mobile-friendly interface with modern UI
- **Multi-Protocol Architecture**: HTTP, WebSocket, and optional TCP/IP for MATLAB integration

## 📋 Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Software Requirements](#software-requirements)
- [Quick Start](#quick-start)
- [System Architecture](#system-architecture)
- [Usage](#usage)
- [API Documentation](#api-documentation)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## 🔧 Hardware Requirements

- **Arduino GIGA R1 WiFi** (STM32H747XI dual-core MCU)
- USB-C cable for programming
- 2.4 GHz WiFi network (5 GHz not supported)

## 💻 Software Requirements

- **Arduino IDE** 2.0 or later
- **Arduino GIGA Board Package** (via Board Manager)
- **Libraries** (install via Library Manager):
  - `WiFi` (built-in)
  - `WebSocketsServer` by Markus Sattler
  - `ArduinoJson` by Benoit Blanchon

## 🚀 Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/YOUR_USERNAME/giga-led-control.git
cd giga-led-control
```

### 2. Configure WiFi Credentials

Edit `arduino/config.h`:

```cpp
const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASSWORD = "YOUR_PASSWORD";
```

### 3. Upload to Arduino GIGA

1. Open `arduino/giga_led_control.ino` in Arduino IDE
2. Select: **Tools → Board → Arduino GIGA R1 WiFi**
3. Select correct COM port
4. Click **Upload**

### 4. Access Web Interface

1. Open Serial Monitor (115200 baud)
2. Note the IP address (e.g., `192.168.0.147`)
3. Open browser and navigate to `http://192.168.0.147`
4. Login with credentials:
   - **Admin**: `admin` / `admin` (full control)
   - **User**: `user` / `user` (view-only)

## 🏗️ System Architecture

### Hardware Architecture
┌─────────────────────────────────────────────┐
│ Arduino GIGA R1 WiFi Board │
├─────────────────────────────────────────────┤
│ STM32H747XI Dual-Core MCU │
│ ├─ Cortex-M7 @ 480 MHz (Web Server) │
│ └─ Cortex-M4 @ 240 MHz (Available) │
│ │
│ Murata WiFi Module (2.4 GHz) │
│ RGB LED (Pins 86, 87, 88 - Active LOW) │
└─────────────────────────────────────────────┘

### Software Architecture
┌──────────────────────────────────────────┐
│ User's Web Browser │
│ (HTTP + WebSocket Client) │
└──────────────┬───────────────────────────┘
│ WiFi
┌──────────────▼───────────────────────────┐
│ Arduino GIGA R1 WiFi │
│ ┌────────────────────────────────────┐ │
│ │ HTTP Server (Port 80) │ │
│ │ - Login authentication │ │
│ │ - Serve HTML pages │ │
│ │ - LED control API │ │
│ └────────────────────────────────────┘ │
│ ┌────────────────────────────────────┐ │
│ │ WebSocket Server (Port 81) │ │
│ │ - Real-time LED state broadcast │ │
│ │ - Activity log updates │ │
│ └────────────────────────────────────┘ │
│ ┌────────────────────────────────────┐ │
│ │ Application Layer │ │
│ │ - LED Control (Pins 86/87/88) │ │
│ │ - Session Management │ │
│ │ - History Logging (10 entries) │ │
│ └────────────────────────────────────┘ │
└──────────────────────────────────────────┘

### File Structure

| File | Purpose |
|------|---------|
| `giga_led_control.ino` | Main sketch (setup & loop) |
| `config.h` | Configuration constants (WiFi, pins, ports) |
| `authentication.h` | User credentials & session management |
| `led_control.h` | RGB LED state management |
| `history_log.h` | Activity logging (circular buffer) |
| `websocket_handler.h` | WebSocket server & event handling |
| `html_pages.h` | HTML page generation (login & dashboard) |
| `http_server.h` | HTTP request routing & processing |

## 📖 Usage

### Default Credentials

| Username | Password | Role | Permissions |
|----------|----------|------|-------------|
| `admin` | `admin` | Administrator | Full LED control |
| `user` | `user` | Read-only | View-only access |

### LED Control

**Admin users** can control three LEDs:
- 🔴 **Red LED** (Pin 86)
- 🟢 **Green LED** (Pin 87)
- 🔵 **Blue LED** (Pin 88)

Each LED has ON/OFF buttons on the dashboard.

### Activity Log

The system maintains a **server-side log** of the last 10 LED state changes:
- Timestamp (seconds since boot)
- Username who made the change
- Action performed (e.g., "Red ON", "Green OFF")

Log is displayed in **reverse chronological order** (newest first).

## 🔌 API Documentation

See [docs/API.md](docs/API.md) for complete API reference.

### HTTP Endpoints

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| `GET` | `/` | Main dashboard or login page | Session cookie |
| `POST` | `/login` | Authenticate user | No |
| `GET` | `/logout` | End session | Session cookie |
| `GET` | `/led?color=X&state=Y` | Control LED | Admin only |

### WebSocket Messages

**Server → Client:**
- LED state updates: `{"red": 1, "green": 0, "blue": 1}`
- History updates: `[{"time":"123s","user":"admin","action":"Red ON"}]`

## 🐛 Troubleshooting

See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for detailed solutions.

### Common Issues

**Problem**: Arduino won't connect to WiFi  
**Solution**: Ensure you're using a 2.4 GHz network (not 5 GHz)

**Problem**: Can't access web page  
**Solution**: Check Serial Monitor for IP address; ensure PC and Arduino on same network

**Problem**: LEDs don't respond  
**Solution**: Verify pins 86/87/88 are defined correctly; LEDs are active-LOW

## 🛠️ Development

### Adding New Features

The modular architecture makes it easy to extend:

**Example: Add temperature sensor**
1. Create `sensor_handler.h`
2. Include in main `.ino` file
3. Add sensor data to WebSocket broadcasts
4. Update HTML to display temperature

### Code Style

- Use **header guards** (`#ifndef`, `#define`, `#endif`)
- Follow **Arduino naming conventions**
- Comment complex logic
- Keep files focused (single responsibility)

## 🤝 Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Arduino Community** for GIGA board support
- **Markus Sattler** for WebSocketsServer library
- **Benoit Blanchon** for ArduinoJson library

## 📧 Contact

**Your Name** - [@your_twitter](https://twitter.com/your_twitter)

Project Link: [https://github.com/YOUR_USERNAME/giga-led-control](https://github.com/YOUR_USERNAME/giga-led-control)

## 🗺️ Roadmap

- [ ] Add MQTT support for cloud integration
- [ ] Implement MATLAB TCP/IP server
- [ ] Add Python data logging bridge
- [ ] Support for external sensors (DHT22, etc.)
- [ ] mDNS hostname support (`giga-led.local`)
- [ ] OTA (Over-The-Air) firmware updates

---

**Built with ❤️ for Industrial IoT Education**
