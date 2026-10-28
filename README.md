# ESP32 Gesture-Controlled Car

This is my ESP32 based gesture-controlled robotic car project made using two ESP32 microcontrollers, an MPU-9250 IMU sensor, an L298N motor driver, four BO geared DC motors, and an HC-SR04 ultrasonic distance sensor.

The car uses a 4-wheel drive system with skid steering. One ESP32 is worn on the hand and reads the tilt angles (pitch and roll) using the MPU-9250 sensor, then transmits movement commands wirelessly over ESP-NOW. The receiving ESP32 on the car receives the commands and controls the left and right motor pairs independently through the L298N driver.

This allows the car to move forward, turn left and right, stop, toggle reverse drive, and also automatically brake when an obstacle is detected in front.

The project includes the CAD model, circuit connections, ESP32 code and the required setup to build and control the car.

<img width="1600" alt="Assembled Gesture Controlled Car" src="assets/PHOTO-2026-10-23-01-56-13.jpg" />

<img width="1600" alt="Car Electronics and Wiring" src="assets/PHOTO-2026-10-23-02-08-37.jpg" />

<img width="1600" alt="3D CAD Model" src="assets/car-3d-model.svg" />

## features

- ESP32 as the transmitter and receiver controller
- MPU-9250 9-axis IMU for hand tilt gesture detection
- Direct peer-to-peer wireless communication via ESP-NOW (no Wi-Fi router or Bluetooth lag)
- L298N dual H-bridge motor driver
- 4-wheel drive with four BO geared DC motors
- Skid / differential steering
- Forward and reverse drive modes
- Left and right turning
- Automatic obstacle avoidance using HC-SR04 ultrasonic sensor
- Hardware PWM speed control
- Custom 3D CAD chassis designed in Fusion 360
- Battery powered with rechargeable 7.4V 2S Li-ion battery pack

## BOM – bill of materials

| component | quantity | price (USD) | total (USD) | purpose |
|---|:---:|---:|---:|---|
| [ESP32 development board (ESP-WROOM-32)](https://robu.in/product/esp-wroom-32-wifi-bluetooth-networking-smart-component-development-board/) | 2 | $4.49 | $8.98 | hand controller & car receiver |
| [MPU-9250 9-DOF IMU sensor](https://robu.in/product/mpu-9250-9-axis-attitude-gyro-accelerometer-magnetometer-sensor-module/) | 1 | $4.20 | $4.20 | measures hand tilt gestures |
| [L298N dual H-bridge motor driver](https://robu.in/product/l298n-2a-based-motor-driver-module/) | 1 | $2.50 | $2.50 | drives the 4 DC motors |
| [TT / BO geared DC motors (6V)](https://robu.in/product/dual-shaft-bo-motor-straight/) | 4 | $1.10 | $4.40 | 4-wheel drive system |
| [Rubber wheels (65 mm)](https://robu.in/product/smart-car-robot-plastic-tire-wheel/) | 4 | $0.80 | $3.20 | wheels for BO motors |
| [HC-SR04 ultrasonic sensor](https://robu.in/product/ultrasonic-transceiver-module-hc-sr04/) | 1 | $1.20 | $1.20 | front obstacle detection |
| [7.4V 2S Li-ion battery pack](https://robu.in/product/orange-2s-7-4v-2200mah-25c-lithium-polymer-battery-pack-lipo/) | 1 | $9.50 | $9.50 | car power supply |
| Jumper wires & mini breadboard | as required | $2.00 | $2.00 | circuit connections |
| Power rocker switch | 1 | $0.40 | $0.40 | main power switch |
| 3D printed chassis parts | as required | — | — | custom chassis & mounts |

## circuit connections

<img width="881" height="610" alt="Controller Circuit Diagram" src="assets/circuit-diagram.png" />

The transmitter ESP32 reads the tilt angles from the MPU-9250 sensor over I2C and transmits commands wirelessly via ESP-NOW.

The receiver ESP32 on the car reads the commands and controls the L298N motor driver using direction pins and hardware PWM. It also reads the HC-SR04 ultrasonic distance sensor.

### Hand controller: MPU-9250 to ESP32

| MPU-9250 | ESP32 |
|---|---|
| VCC | 3V3 |
| GND | GND |
| SDA | GPIO 15 |
| SCL | GPIO 14 |

### Car receiver: ESP32 to L298N motor driver

| L298N Pin | ESP32 Pin | Function |
|---|---|---|
| IN1 | GPIO 26 | Left motors direction 1 |
| IN2 | GPIO 25 | Left motors direction 2 |
| IN3 | GPIO 17 | Right motors direction 1 |
| IN4 | GPIO 16 | Right motors direction 2 |
| ENA | GPIO 32 | Left motor speed (PWM Channel 0) |
| ENB | GPIO 33 | Right motor speed (PWM Channel 1) |

### Car receiver: HC-SR04 ultrasonic sensor to ESP32

| HC-SR04 | ESP32 | Notes |
|---|---|---|
| VCC | 5V | Powers the ultrasonic sensor |
| GND | GND | Common ground |
| TRIG | GPIO 18 | Trigger output pulse |
| ECHO | GPIO 19 | Echo input *(use voltage divider: 5V to 3.3V)* |

### motors

| L298N Terminal | Motors Connected |
|---|---|
| OUT1 / OUT2 | Two left BO motors (in parallel) |
| OUT3 / OUT4 | Two right BO motors (in parallel) |

If a motor spins in the opposite direction, simply swap its two motor wires on the L298N terminal block.

## power connections

The motors are powered from the 7.4V battery connected to the L298N `12V` and `GND` terminals.

The car ESP32 is powered via its `VIN` pin (or 5V regulated output), and the logic pins use common ground.

```text
7.4V Battery (+)
       |
       ├── Power Switch
       |      |
       |      └── L298N 12V (VMS) Terminal
       |
       └── Step-down / 5V source ──► ESP32 VIN & HC-SR04 VCC

7.4V Battery (-)
       |
       ├── L298N GND
       └── ESP32 GND
```

All grounds must be connected together:

```text
Battery (-)
   |
   ├── L298N GND
   ├── ESP32 GND
   └── HC-SR04 GND
```

Do not power the motors directly from the ESP32 3.3V or 5V pins. The L298N power terminal and logic ground must share a common ground reference with the ESP32.

## how to build this

**step 1:** Gather all components listed in the Bill of Materials (BOM).

**step 2:** 3D print the chassis parts using the CAD files provided in the `3D Cad/` folder.

**step 3:** Mount the four BO geared motors onto the chassis brackets and attach the 65 mm rubber wheels.

**step 4:** Mount the L298N motor driver and the car ESP32 onto the chassis deck.

**step 5:** Wire the two left motors in parallel to `OUT1`/`OUT2` and the two right motors to `OUT3`/`OUT4`.

**step 6:** Connect the L298N control pins (`IN1`, `IN2`, `IN3`, `IN4`, `ENA`, `ENB`) to the car ESP32.

**step 7:** Mount the HC-SR04 ultrasonic sensor to the front bumper and wire `TRIG` and `ECHO` to the car ESP32.

**step 8:** Wire the hand controller: connect the MPU-9250 IMU to the transmitter ESP32 (`3V3`, `GND`, `GPIO 15`, `GPIO 14`).

**step 9:** Power on the car ESP32, upload a MAC address finder sketch, and note down its station MAC address.

**step 10:** In `GestureController/GestureController.ino`, replace `carMAC` with your car ESP32's MAC address.

**step 11:** Upload `GestureController.ino` to the hand controller ESP32 and `GestureCarReceiver.ino` to the car ESP32.

**step 12:** Lift the car off the ground and test motor responses for all tilt gestures.

**step 13:** Place the car on a flat surface and test wireless driving and obstacle avoidance.

## How the car works????

The working of the car is based on gesture detection, ESP-NOW wireless transmission, and differential motor drive.

```text
Hand movement
      ↓
MPU-9250 IMU (Accelerometer)
      ↓
Transmitter ESP32
      ↓
Calculates Pitch & Roll angles
      ↓
Selects Command (Forward / Left / Right / Brake / Reverse)
      ↓
ESP-NOW wireless packet (100 ms interval)
      ↓
Receiver ESP32
      ↓
Checks HC-SR04 Obstacle Sensor (Front < 20 cm?)
   ├── Yes ──► Auto-Brake + Turn Right
   └── No  ──► Executes Received Command
                    ↓
              L298N Motor Driver
              ┌───────────────┐
              ↓               ↓
         Left Motors     Right Motors
              ↓               ↓
         Left Wheels     Right Wheels
```

The transmitter continuously reads the accelerometer axes ($a_x$, $a_y$, $a_z$) and computes the tilt angles:

```text
pitch = atan2(ax, sqrt(ay * ay + az * az)) * 180 / PI
roll  = atan2(ay, sqrt(ax * ax + az * az)) * 180 / PI
```

The tilt angle is matched against a 15° threshold:
- Pitch > 15°: Forward
- Pitch < -15°: Reverse toggle
- Roll > 15°: Turn Right
- Roll < -15°: Turn Left
- Flat: Brake (Stop)

The command packet is sent directly via ESP-NOW to the car ESP32, which drives the motors accordingly.

## gesture controls

The car responds to hand gestures with skid steering:

```text
Tilt Forward

Left wheels   →→→
Right wheels  →→→
(Both sides move forward)
```

```text
Tilt Left

Left wheels   ■■■ (Stopped)
Right wheels  →→→ (Forward)
(Car pivots to the left)
```

```text
Tilt Right

Left wheels   →→→ (Forward)
Right wheels  ■■■ (Stopped)
(Car pivots to the right)
```

```text
Hand Flat

Left wheels   ■■■ (Braked)
Right wheels  ■■■ (Braked)
(Car comes to a full stop)
```

```text
Tilt Backward

Toggles drive direction between Forward and Reverse.
```

## obstacle avoidance

The HC-SR04 ultrasonic sensor guards the front of the car:

```text
HC-SR04 Sensor
       ↓
Measures distance to obstacle
       ↓
Distance <= 20 cm?
 ├── Yes:
 │     1. Brake motors immediately
 │     2. Pause 200 ms
 │     3. Turn Right for 350 ms
 │     4. Brake and wait for clear path
 └── No:
       Follow hand controller commands
```

This prevents head-on crashes into walls and furniture during driving.

## speed control

The ESP32 uses built-in hardware LEDC PWM to control the speed of the motors through the L298N driver `ENA` and `ENB` pins.

- Frequency: `1000 Hz`
- Resolution: `8-bit (0 – 255)`
- Default Driving Speed: `180 / 255`

This provides plenty of torque while keeping the car manageable to steer indoors.

## Testing

Before driving the completed car on the floor, test each subsystem independently.

### 1. Transmitter & IMU test
Open the Serial Monitor at **115200 baud** for the controller ESP32. Tilt your hand forward, backward, left, and right, and verify that `FORWARD`, `LEFT`, `RIGHT`, `BRAKE`, and `REVERSE` appear accurately.

### 2. ESP-NOW connection test
Power both boards. Verify that the receiver serial monitor shows packets arriving every 100 ms and that `esp_now_init()` returns success on both sides.

### 3. Motor direction test
Lift the car so the wheels are in the air. Verify that:
- Tilting forward rotates all 4 wheels forward.
- Tilting left spins the right wheels forward while left wheels stop.
- Tilting right spins the left wheels forward while right wheels stop.
- Holding the hand flat stops all 4 wheels.

### 4. Obstacle sensor test
Wave your hand in front of the HC-SR04 sensor within 20 cm while the car is driving forward. Verify that the car immediately brakes and executes the right turn evasion.

### 5. Floor driving test
Place the car on a smooth, open floor and enjoy driving with intuitive hand motions!

## CAD model

<img width="1200" alt="CAD 3D Chassis View" src="assets/car-3d-model.svg" />

<img width="1000" alt="Chassis and Hardware Inside" src="assets/PHOTO-2026-10-23-02-08-37.jpg" />

The CAD model contains the custom 4WD chassis base, motor mounting brackets, internal battery tray, and front bracket for the HC-SR04 ultrasonic sensor.

The design files are located in the [`3D Cad/`](3D%20Cad/) folder:
- `The Nonchalant (1).f3d`: Editable parametric Autodesk Fusion 360 archive.
- `The Nonchalant.step`: Complete 3D STEP assembly file.
- Individual `.step` component files for custom 3D printing.

## Things to keep in mind

1. All grounds (battery, ESP32, L298N, and sensors) must be tied together.
2. Do not attempt to power the DC motors directly from the ESP32 3.3V or 5V pins.
3. The HC-SR04 echo pin outputs 5V logic; use a voltage divider (e.g. 1kΩ / 2kΩ resistors) before connecting it to ESP32 GPIO 19.
4. Keep the drive wheels off the ground during your first upload and motor test.
5. Make sure to replace the dummy MAC address in `GestureController.ino` with your car ESP32's actual MAC address.
6. Check that the MPU-9250 is oriented correctly on your hand so pitch matches forward/backward and roll matches left/right.
7. Use thick enough wire for motor power connections to minimize voltage drop under load.
8. If one side runs backward when commanding forward, swap that motor side's wires on the L298N terminal.
9. Keep the battery securely fastened so it does not slide during sudden acceleration or turns.
10. If the receiver does not receive packets, verify that both ESP32 boards are set to the same Wi-Fi channel.
11. The backward gesture toggles reverse mode; keep your hand flat after tilting backward to avoid multiple toggles.
12. Ensure the L298N onboard 5V regulator jumper is enabled if supplying 7.4V to the motor terminal.

## Final result

This project creates a wireless, intuitive gesture-controlled 4WD robot using affordable, accessible maker hardware.

The MPU-9250 tracks natural hand movement, ESP-NOW provides near-instantaneous packet delivery with zero Wi-Fi network dependency, and the L298N drives the 4-wheel drivetrain smoothly with front obstacle protection.

The main idea is simple:

```text
Hand Gesture
     ↓
 MPU-9250
     ↓
Transmitter ESP32
     ↓
ESP-NOW Wireless
     ↓
 Receiver ESP32
     ↓
L298N Motor Driver + HC-SR04
     ↓
4x Geared DC Motors
     ↓
Gesture-Controlled Car
```

## Demo

- 📹 **Demo Video**: [Watch the Car Driving Video (MP4)](assets/VIDEO-2026-10-23-02-05-10.mp4)
- 📷 **Assembled Build**: [assets/PHOTO-2026-10-23-01-56-13.jpg](assets/PHOTO-2026-10-23-01-56-13.jpg)
- 📷 **Interior Electronics**: [assets/PHOTO-2026-10-23-02-08-37.jpg](assets/PHOTO-2026-10-23-02-08-37.jpg)
- 📓 **Build Journal**: Step-by-step development reflections and notes in [JOURNAL.md](JOURNAL.md).
