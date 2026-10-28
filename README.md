# 🚗 Gesture-Controlled Car — The Nonchalant

A 4-wheel-drive robotic car controlled by hand gestures using two ESP32 microcontrollers and ESP-NOW wireless protocol.

![Assembled Gesture Controlled Car](assets/PHOTO-2026-10-23-01-56-13.jpg)

---

## 📌 Overview

Tilt your hand to steer the car. An MPU-9250 motion sensor reads the hand tilt angle, an ESP32 sends commands wirelessly via ESP-NOW, and the receiving ESP32 drives four DC motors through an L298N motor driver. An ultrasonic sensor on the front provides automatic obstacle avoidance.

- **No Wi-Fi router needed**: Direct board-to-board ESP-NOW communication.
- **Fast response**: Low-latency control (~100 ms packet rate).
- **Custom chassis**: 3D modeled in Fusion 360 and 3D printed.
- **Build Journal**: Detailed step-by-step build notes in [JOURNAL.md](JOURNAL.md).

---

## 🎬 Demo

- 📹 **Video Demo**: [Watch the Driving Demo](assets/VIDEO-2026-10-23-02-05-10.mp4)
- 📷 **Exterior View**: [assets/PHOTO-2026-10-23-01-56-13.jpg](assets/PHOTO-2026-10-23-01-56-13.jpg)
- 📷 **Chassis & Electronics**: [assets/PHOTO-2026-10-23-02-08-37.jpg](assets/PHOTO-2026-10-23-02-08-37.jpg)

---

## 🖐️ Gesture Controls

The transmitter measures pitch and roll angles with a 15° deadzone threshold:

| Hand Gesture | Command | Car Action |
|---|---|---|
| **Tilt Forward** | `FORWARD` | Drives all 4 wheels forward |
| **Tilt Left** | `LEFT_CMD` | Stops left wheels, drives right wheels |
| **Tilt Right** | `RIGHT_CMD` | Stops right wheels, drives left wheels |
| **Tilt Backward** | `REVERSE_CMD` | Toggles drive direction (Forward / Reverse) |
| **Keep Flat** | `BRAKE_CMD` | Stops all motors |

---

## 🧩 System Architecture

```text
[ Hand Controller ]                     [ Robotic Car ]
   MPU-9250 IMU                         ESP32 Receiver
        │ (I2C)                                │
  ESP32 Transmitter ─── ESP-NOW Wireless ───►  ├── L298N Driver ──► 4x BO Motors
                                               └── HC-SR04 Sensor (Auto-Brake)
```

### Obstacle Avoidance
- An **HC-SR04** ultrasonic sensor continuously measures front distance.
- If an obstacle appears within **20 cm**, the car stops, turns right for 350 ms, and halts safely.

---

## 🛠️ Hardware Components

| Component | Qty | Function |
|---|:---:|---|
| **ESP32 Dev Board (ESP-WROOM-32)** | 2 | Controller (Tx) and Car Receiver (Rx) |
| **MPU-9250 IMU Sensor** | 1 | 9-axis gesture and tilt detection |
| **L298N Dual H-Bridge Driver** | 1 | High-current DC motor driver |
| **TT / BO Geared DC Motors (6V)** | 4 | 4WD drivetrain |
| **Rubber Wheels (65 mm)** | 4 | Drive wheels |
| **HC-SR04 Ultrasonic Sensor** | 1 | Distance detection (front bumper) |
| **7.4V 2S Li-ion Battery Pack** | 1 | Car power supply |
| **Custom 3D Printed Chassis** | 1 | Custom frame from Fusion 360 |
| **Jumper Wires & Power Switch** | — | Interconnects and power toggle |

---

## 🔌 Circuit & Wiring

### 1. Transmitter (Hand Controller)

![Controller Circuit Diagram](assets/circuit-diagram.png)

| ESP32 Pin | MPU-9250 Pin |
|---|---|
| `3V3` | `VCC` |
| `GND` | `GND` |
| `GPIO 15` | `SDA` |
| `GPIO 14` | `SCL` |

### 2. Receiver (Car Chassis)

| ESP32 Pin | Connected To | Description |
|---|---|---|
| `GPIO 26` | L298N `IN1` | Left motors direction 1 |
| `GPIO 25` | L298N `IN2` | Left motors direction 2 |
| `GPIO 17` | L298N `IN3` | Right motors direction 1 |
| `GPIO 16` | L298N `IN4` | Right motors direction 2 |
| `GPIO 32` | L298N `ENA` | Left speed (PWM Ch 0) |
| `GPIO 33` | L298N `ENB` | Right speed (PWM Ch 1) |
| `GPIO 18` | HC-SR04 `TRIG` | Ultrasonic trigger pulse |
| `GPIO 19` | HC-SR04 `ECHO` | Ultrasonic echo return *(use voltage divider to 3.3V)* |

> ⚠️ **Note:** Connect all grounds (`GND`) together (ESP32, L298N, and Battery).

---

## 📐 3D CAD Chassis

The chassis is custom-designed in Autodesk Fusion 360 to house the motors, battery pack, driver, and controller boards.

![3D CAD Model](assets/car-3d-model.svg)

CAD files are provided in the [`3D Cad/`](3D%20Cad/) folder:
- `The Nonchalant (1).f3d` — Editable Fusion 360 archive.
- `The Nonchalant.step` — Complete 3D STEP assembly.
- Individual `.step` files for custom chassis components.

![Electronics Inside Chassis](assets/PHOTO-2026-10-23-02-08-37.jpg)

---

## 🚀 Quick Setup

### 1. Requirements
- **Arduino IDE** (with ESP32 board support by Espressif).
- Library: **MPU9250_asukiaaa** (install via Arduino Library Manager).

### 2. Find Receiver MAC Address
Upload a simple sketch to the car's ESP32:
```cpp
#include <WiFi.h>
void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  Serial.print("Car MAC: ");
  Serial.println(WiFi.macAddress());
}
void loop() {}
```

### 3. Configure & Flash
1. In [`GestureController/GestureController.ino`](GestureController/GestureController.ino), update `carMAC` with your car ESP32 MAC address:
   ```cpp
   uint8_t carMAC[] = { 0xXX, 0xXX, 0xXX, 0xXX, 0xXX, 0xXX };
   ```
2. Upload `GestureController.ino` to the controller ESP32.
3. Upload [`GestureCarReceiver/GestureCarReceiver.ino`](GestureCarReceiver/GestureCarReceiver.ino) to the car ESP32.
4. Power both boards. Tilt your hand to drive!

---

## 📁 Repository Structure

```text
├── 3D Cad/                 # Fusion 360 (.f3d) and STEP CAD files
├── assets/                 # Photos, circuit schematics, CAD SVG & demo video
├── GestureCarReceiver/     # ESP32 receiver & motor control sketch
├── GestureController/      # ESP32 gesture transmitter sketch
├── hardware_headers/       # Header reference files
├── JOURNAL.md              # Engineering build journal & design steps
└── README.md               # Project documentation
```

---

## 👤 Author

- **Bhavesh Sharma** — [GitHub @bhavesh-210](https://github.com/bhavesh-210)
