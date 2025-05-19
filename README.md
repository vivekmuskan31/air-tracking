# 🛩️ RC Plane Autonomous Object Tracking (Raspberry Pi 4)

This project enables an RC plane to detect and track a ground target (e.g., a rover) using onboard computer vision. All processing is performed on a Raspberry Pi 4B+ with no GCS (Ground Control Station). A Flysky 6CH remote is used to switch between manual and autonomous modes.

---

## 🎯 Project Objectives

1. **Manual Flight Mode** – Control via Flysky transmitter (4 channels for pitch, roll, yaw, throttle).
2. **Autonomous Stabilization** – Use MPU6050 to apply PID-based stabilization to maintain level flight.
3. **Object Detection + Tracking Mode**
   - Detect a "rover" or specific ground object using lightweight CV models
   - Once detected, track the object using onboard algorithms
   - Translate object drift into control commands (yaw/pitch) to keep target centered in camera view

---

## ⚙️ System Components

### ✅ Hardware

| Component               | Purpose                              |
|------------------------|--------------------------------------|
| Raspberry Pi 4B+       | Onboard compute                      |
| Pi Camera (v2 or HQ)   | Real-time video input                |
| MPU6050                | Roll/pitch/yaw sensing               |
| Flysky i6 Transmitter  | Manual mode + toggle switch          |
| ESC + Servos           | Control surfaces + propulsion        |

---

## 🧠 Software Architecture

### 🟦 Mode FSM (Finite State Machine)

| Mode              | Trigger (Remote) | Behavior                                  |
|------------------|------------------|-------------------------------------------|
| Manual            | CH5 LOW          | Pass all control to pilot                 |
| Stabilized Manual | CH5 LOW + MPU    | Assist pilot with level flight            |
| Detection Mode    | CH5 HIGH, CH6 LOW| Run object detection every N frames       |
| Tracking Mode     | CH6 HIGH         | Lock onto detected object, track via KCF  |
| Fallback          | Lost target      | Revert to Detection Mode or Idle          |

---

### 🧱 Software Modules

#### 📍 `imu_stabilizer.py`
- Reads MPU6050 data
- Runs PID loop to adjust control surfaces

#### 📍 `object_detector.py`
- Runs lightweight detection model (NanoDet or MobileNet SSD)
- Detects predefined object class (e.g., `rover`)

#### 📍 `object_tracker.py`
- Initializes KCF/CSRT on selected bounding box
- Outputs `x/y` drift vector relative to image center

#### 📍 `fsm_controller.py`
- Handles mode transitions based on CH5/CH6 state
- Triggers detector/tracker/PID accordingly

#### 📍 `servo_controller.py`
- Maps drift vectors to yaw/pitch control
- Interfaces with GPIO/ESC/servo drivers

---

## 🛠️ Tools & Libraries

- Python 3.10+
- OpenCV (`cv2`)
- TFLite/ONNX runtime (for detector)
- `smbus2` for MPU6050
- `RPi.GPIO` or `pigpio`
- Lightweight model (NanoDet/MobileNet SSD)

---

## 📂 Suggested File Structure

```
rc_plane_tracking/
├── imu_stabilizer.py
├── object_detector.py
├── object_tracker.py
├── fsm_controller.py
├── servo_controller.py
├── models/
│ └── rover_model.tflite
├── config/
│ └── pid_params.yaml
├── plane_main.py
├── README.md
└── requirements.txt
```

---

## 🚀 Execution Plan

- [x] Complete manual flight tuning
- [ ] Integrate and test MPU6050 PID stabilization
- [ ] Benchmark detection model (low-res inference)
- [ ] Implement FSM and remote channel parsing
- [ ] Validate object tracking (low-altitude test)
- [ ] Tune response mapping to servo output

---

## 📌 Notes

- Pi 4 can run basic detection + tracking in real time with correct optimizations
- All control surfaces handled on Pi — no ground station needed
- Hardcoded altitude limits (20–40m) ensure object visibility

---

## 🧪 Future Enhancements

- Gimbal-mounted camera stabilization
- PX4-based flight controller integration
- Offline object re-identification / trajectory mapping

---

## 🧠 References

- [NanoDet GitHub](https://github.com/RangiLyu/nanodet)
- [MobileNet SSD](https://github.com/chuanqi305/MobileNet-SSD)
- [OpenCV Tracking API](https://docs.opencv.org/4.x/d9/df8/group__tracking.html)
- [MPU6050 on Pi](https://circuitdigest.com/microcontroller-projects/raspberry-pi-mpu6050-interfacing-tutorial)

