# Bilateral Exoskeleton Gait Rehabilitation System

> An ESP32-based exoskeleton rehabilitation controller with real-time WebSocket communication and a web-based control panel for bilateral gait pattern playback.

---

## 📖 Overview

This project implements a **bilateral lower-limb exoskeleton control system** designed for gait rehabilitation. The ESP32 microcontroller drives four servo motors (left/right hip and knee joints) using pre-recorded 100-sample gait arrays derived from clinical gait data. A built-in Wi-Fi Access Point hosts a responsive web dashboard for real-time monitoring and manual control — no internet connection required.

---

## ✨ Features

- ✅ Real-time bilateral gait pattern playback from 100-sample arrays
- ✅ Web-based control interface via Wi-Fi Access Point
- ✅ WebSocket communication for low-latency updates (~100ms refresh)
- ✅ Direct servo control at 50Hz (20ms update interval)
- ✅ Variable playback speed (0.5×–2.0×)
- ✅ Manual joint override with live position feedback
- ✅ Gait phase detection (Heel Strike → Foot Flat → Midstance → Heel Off → Toe Off)
- ✅ Completed cycle counter
- ✅ Buzzer feedback on start, pause, stop, and cycle completion
- ✅ Status LED with state-based blink patterns
- ✅ System diagnostics (load, uptime, connected clients)
- ✅ Error handling and emergency stop

---

## 🔧 Hardware Requirements

| Component | Specification |
|-----------|--------------|
| Microcontroller | ESP32 (any variant with sufficient GPIO) |
| Servo Motors (×4) | Standard PWM servos, 500–2500 µs pulse range |
| Buzzer | Passive buzzer on GPIO 26 |
| Status LED | GPIO 2 (built-in LED usable) |
| Power Supply | Adequate for 4 simultaneous servos |

### Pin Assignments

| Joint | GPIO Pin |
|-------|----------|
| Left Hip | GPIO 18 |
| Left Knee | GPIO 19 |
| Right Hip | GPIO 32 |
| Right Knee | GPIO 33 |
| Buzzer | GPIO 26 |
| Status LED | GPIO 2 |

---

## 📦 Dependencies

Install these libraries via the **Arduino Library Manager** or **PlatformIO**:

| Library | Purpose |
|---------|---------|
| `ESP32Servo` | Servo motor control with PWM allocation |
| `ESPAsyncWebServer` | Async HTTP server |
| `AsyncTCP` | Async TCP base for ESPAsyncWebServer |
| `ArduinoJson` | JSON parsing for WebSocket commands |
| `WiFi` (built-in) | Wi-Fi Access Point |
| `Wire` (built-in) | I2C (reserved for future sensors) |

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/exoskeleton-rehab-control.git
cd exoskeleton-rehab-control
```

### 2. Configure Wi-Fi Credentials

Open the sketch and update the AP credentials at the top of the file:

```cpp
const char* ssid     = "ExoskeletonAP";   // Change to your preferred SSID
const char* password = "rehab2025";        // Change to a secure password
```

### 3. Upload to ESP32

1. Open the `.ino` file in Arduino IDE (2.x recommended)
2. Select your ESP32 board under **Tools → Board**
3. Select the correct COM port
4. Click **Upload**

### 4. Connect and Control

1. On your device, connect to the Wi-Fi network **`ExoskeletonAP`**
2. Open a browser and navigate to **`http://192.168.4.1`**
3. The control dashboard will load automatically

---

## 🖥️ Web Interface

The dashboard is served directly from ESP32 flash memory (no external files needed).

### Control Panel

| Button | Action |
|--------|--------|
| ▶ Start | Begin gait pattern playback |
| ⏸ Pause | Freeze at current gait index |
| ⏹ Stop | Stop and return all joints to 90° |

### Speed Control
Adjust playback speed from **0.5× (slow)** to **2.0× (fast)** using the slider or ± buttons.

### Manual Override
Individual sliders for each of the four joints (0°–180°) allow direct positioning when the system is stopped or paused.

### Diagnostics Panel
Displays system load, session uptime, completed gait cycles, and connected WebSocket clients.

---

## 🦵 Gait Pattern Data

The gait arrays contain **100 samples** representing one complete normalized gait cycle for both legs in a **contralateral (alternating) pattern**:

| Array | Joint | Description |
|-------|-------|-------------|
| `hipLeft[100]` | Left Hip | ~71°–109° swing/stance arc |
| `kneeLeft[100]` | Left Knee | ~72°–107° flexion/extension |
| `hipRight[100]` | Right Hip | Phase-shifted left hip (50 samples offset) |
| `kneeRight[100]` | Right Knee | Phase-shifted left knee (50 samples offset) |

The right leg arrays are offset by 50 samples from the left leg, producing natural bilateral gait symmetry.

### Gait Phases

| Phase | Name | Sample Range |
|-------|------|-------------|
| 0 | Heel Strike | 0–19 |
| 1 | Foot Flat | 20–39 |
| 2 | Midstance | 40–59 |
| 3 | Heel Off | 60–79 |
| 4 | Toe Off | 80–99 |

---

## ⚙️ System Architecture

```
ESP32
 ├── Wi-Fi AP (192.168.4.1)
 │    ├── HTTP Server (port 80) → Serves dashboard HTML
 │    └── WebSocket (/ws) → Bidirectional real-time data
 │
 ├── Gait Engine (50Hz loop)
 │    └── Advances gait index → Updates servo target angles
 │
 ├── Servo Controller (50Hz)
 │    ├── Left Hip  (GPIO 18)
 │    ├── Left Knee (GPIO 19)
 │    ├── Right Hip (GPIO 32)
 │    └── Right Knee (GPIO 33)
 │
 └── Feedback
      ├── Buzzer (GPIO 26)
      └── Status LED (GPIO 2)
```

### WebSocket Message Format

**Outbound (ESP32 → Browser) — JSON, every 100ms:**
```json
{
  "state": "running",
  "leftHipAngle": 95.3,
  "leftKneeAngle": 82.1,
  "rightHipAngle": 105.2,
  "rightKneeAngle": 81.0,
  "gaitPhase": 2,
  "gaitProgress": 47,
  "speed": 1.0,
  "cycleCount": 3,
  "systemLoad": 55.0,
  "clientCount": 1
}
```

**Inbound (Browser → ESP32) — supported commands:**
```json
{ "command": "start" }
{ "command": "pause" }
{ "command": "stop" }
{ "command": "setSpeed", "value": 1.5 }
{ "command": "setAngle", "joint": "leftHip", "value": 95.0 }
{ "command": "getStatus" }
```

---

## 🔒 Safety Features

- All servo angles are hard-clamped to **0°–180°** before writing
- Emergency stop returns all joints to **90° (neutral/home)**
- `ERROR_STATE` halts gait playback and triggers buzzer alarm
- WebSocket clients are cleaned up automatically to prevent memory leaks

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|---------|
| Can't connect to Wi-Fi AP | Verify SSID/password; check Serial Monitor for AP IP |
| Dashboard not loading | Ensure you're browsing `http://192.168.4.1` (not HTTPS) |
| Servos not moving | Check GPIO connections and power supply adequacy |
| WebSocket disconnecting | Check for Serial Monitor interference; reduce `delay(5)` if needed |
| Angles out of range | Verify servo pulse range matches `SERVO_MIN_PULSE`/`SERVO_MAX_PULSE` |

---

## 📁 File Structure

```
exoskeleton-rehab-control/
├── exoskeleton_control.ino   # Main Arduino sketch
└── README.md                 # This file
```

---

## 👥 Proponents

**Capstone Project — 2026**

- Caparro, D.
- David, L.
- Echavez, M.
- Monterde, L.
- Tañala, A.

---

## 📄 License

This project is intended for academic and rehabilitation research purposes. Please consult your institution's policies before clinical deployment.

---

*ESP32-Based Gait Rehabilitation Device · v1.0 · © 2026*
