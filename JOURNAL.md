# 📓 Project Build Journal — The Nonchalant

A step-by-step development journal of building a 4WD gesture-controlled robotic car from scratch.

![Completed Gesture Controlled Car](assets/PHOTO-2026-10-23-01-56-13.jpg)

---

## 📅 Timeline & Progress Overview

| Phase | Milestone | Status |
|:---:|---|:---:|
| **1** | Project Idea & Architecture | ✅ Completed |
| **2** | Gesture Detection & IMU Integration | ✅ Completed |
| **3** | Wireless Setup via ESP-NOW | ✅ Completed |
| **4** | 3D CAD Design & Chassis Modeling | ✅ Completed |
| **5** | Electronics Assembly & Motor Driving | ✅ Completed |
| **6** | Obstacle Avoidance Integration | ✅ Completed |
| **7** | Field Testing & Calibration | ✅ Completed |

---

## Phase 1: The Idea & Architecture

### Goal
Build a remote-controlled car steered naturally by tilting a hand, avoiding traditional joysticks or smartphone touchscreen apps.

### Key Decisions
1. **Microcontroller**: Used two **ESP32** boards because they have built-in Wi-Fi, high processing speed, and native ESP-NOW protocol support.
2. **Wireless Protocol**: Picked **ESP-NOW** instead of classic Bluetooth or Wi-Fi web server. ESP-NOW sends peer-to-peer packets in under 5 ms without router pairing or connection lag.
3. **Drive System**: 4-wheel drive (4 geared BO motors) with skid steering for tight turns.

---

## Phase 2: Gesture Detection (MPU-9250)

### Implementation
- Connected an **MPU-9250** 9-axis sensor to the transmitter ESP32 over I2C.
- Calculated pitch and roll angles from raw accelerometer readings using `atan2`:
  ```cpp
  float pitch = atan2(ax, sqrt(ay * ay + az * az)) * 180 / PI;
  float roll  = atan2(ay, sqrt(ax * ax + az * az)) * 180 / PI;
  ```
- Set a **15° threshold** deadzone to prevent accidental movement when holding the hand relaxed.

![Transmitter Circuit](assets/circuit-diagram.png)

### Pinout (Transmitter)
- `ESP32 3V3` ──► `MPU-9250 VCC`
- `ESP32 GND` ──► `MPU-9250 GND`
- `ESP32 GPIO 15` ──► `MPU-9250 SDA`
- `ESP32 GPIO 14` ──► `MPU-9250 SCL`

---

## Phase 3: Wireless Communication (ESP-NOW)

### Implementation
- Configured transmitter in Wi-Fi Station mode (`WIFI_STA`).
- Paired the controller directly to the car ESP32's unique MAC address.
- Created a lightweight packet payload containing just the command ID:
  ```cpp
  typedef struct struct_message {
    int comand_id;
  } struct_message;
  ```
- Transmitter dispatches updates every **100 ms**, giving instantaneous response without saturating the wireless channel.

---

## Phase 4: 3D CAD Modeling in Fusion 360

### Design Goals
- House 4 TT geared motors securely.
- Provide a flat interior deck for the L298N motor driver, ESP32, and 7.4V battery pack.
- Include a front mounting slot for the HC-SR04 ultrasonic sensor.

![3D CAD Model](assets/car-3d-model.svg)

### Deliverables
- Exported complete solid model `The Nonchalant.step` and native parametric file `The Nonchalant (1).f3d`.
- Exported individual body components in [`3D Cad/`](3D%20Cad/) for 3D printing.

---

## Phase 5: Hardware Assembly & Motor Driving

### Motor & Driver Setup
- Used an **L298N Dual H-Bridge** module powered by a 7.4V 2S Li-ion battery pack.
- Paired left-side motors together and right-side motors together.
- Used ESP32 hardware PWM channels (`ledcSetup`, 1 kHz, 8-bit) on enable pins `ENA` (GPIO 32) and `ENB` (GPIO 33) to maintain consistent motor speed (`speed = 180 / 255`).

![Inside Chassis Hardware](assets/PHOTO-2026-10-23-02-08-37.jpg)

### Challenges Solved
- **Common Ground**: Linked battery negative, ESP32 GND, and L298N GND together to avoid erratic signal floating.
- **Tidy Wire Routing**: Placed the driver centrally to keep motor wire runs short and balanced.

---

## Phase 6: Obstacle Avoidance

### Adding Safety
- Placed an **HC-SR04** ultrasonic sensor at the front bumper.
- Trigger pin on `GPIO 18`, Echo pin on `GPIO 19`.
- If an object is detected closer than **20 cm**:
  1. Immediately overrides forward motion and brakes.
  2. Pauses 200 ms.
  3. Executes a brief right turn (350 ms).
  4. Brakes again and waits for user gesture.

---

## Phase 7: Field Testing & Calibration

![Fully Assembled Car](assets/PHOTO-2026-10-23-01-56-13.jpg)

### Results
- Car moves forward smoothly when hand tilts forward.
- Responsive left and right skid steering when tilting hand sideways.
- Obstacle avoidance reliably stops the car before colliding with walls.
- Watch video demo: [assets/VIDEO-2026-10-23-02-05-10.mp4](assets/VIDEO-2026-10-23-02-05-10.mp4)

---

## Phase 8: Key Takeaways & Future Improvements

| What Worked Well | What Needs Improvement | Next Steps |
|---|---|---|
| ESP-NOW is rock solid with zero lag | Reverse command toggles repeatedly if hand is held back | Add gesture debounce / edge detection |
| 4WD gives good indoor traction | L298N drops ~1.5V to 2V across bipolar switches | Upgrade to MOSFET driver (e.g. TB6612FNG) |
| Ultrasonic sensor prevents head-on crashes | Fixed speed (180/255) | Implement proportional speed based on tilt angle |
| Chassis fits all components neatly | No failsafe if controller turns off | Add a receiver timeout watchdog timer |

---

## 🏁 Summary

**The Nonchalant** demonstrates an end-to-end integration of embedded systems, wireless communications, sensor fusion, and 3D mechanical design into a fun, responsive gesture-controlled robot.

