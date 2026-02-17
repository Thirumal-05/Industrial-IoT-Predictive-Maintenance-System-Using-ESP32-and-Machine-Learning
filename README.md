# 🏭 Industrial IoT Predictive Maintenance System
### Using ESP32 + Machine Learning (Random Forest)

> **Major Project** — B.Tech ECE, Alturi Mastan Reddy Memorial College of Engineering & Technology

---

## 📖 Overview

In industries, machines and IoT Devices like fire-alarms, smoke detectors, motors, security alarms, and compressors run continuously. When these machines fail without warning, it causes production loss, safety hazards, and high repair costs. Traditional maintenance is either done on a fixed schedule (even when not needed) or only after a breakdown — both are inefficient.

**Predictive Maintenance** solves this by continuously monitoring a machine's health using sensors and predicting when it is likely to fail — before it actually does. This gives engineers time to plan maintenance, order parts, and avoid unplanned downtime.

This project builds a complete predictive maintenance system using three sensors attached to a running motor, an ESP32 microcontroller that reads and transmits sensor data over WiFi, and a Python-based Machine Learning model that analyzes the data and classifies the machine's health as **Normal**, **Warning**, or **Critical** in real time.

When a warning or critical condition is detected, the system immediately alerts the operator through LEDs, a buzzer, an OLED screen, and a Blynk mobile dashboard — giving them enough time to act before a failure occurs.

---

## 📌 Key Specifications

| Feature | Detail |
|---|---|
| Microcontroller | ESP32 Dev Module |
| Sensors | MPU6050 (Vibration), DHT22 (Temperature/Humidity), ACS712 (Current) |
| ML Algorithm | Random Forest Classifier |
| ML Platform | Python — scikit-learn |
| Communication | WiFi → HTTP POST (JSON) |
| Server | Python Flask REST API |
| IoT Dashboard | Blynk Mobile App |
| Health Classes | Normal / Warning / Critical |
| Model Accuracy | ~96% on test set |

---

## 🏗️ System Architecture

The system is divided into three layers that work together:

```
┌──────────────────────────────────────────────────────────┐
│                 LAYER 1: SENSOR LAYER                    │
│                                                          │
│   [MPU6050]       [DHT22]        [ACS712]               │
│   Vibration    Temp/Humidity      Current                │
│       │               │              │                   │
│       └───────────────┴──────────────┘                   │
│                       │                                  │
│                  [ESP32 Dev]                             │
│     Reads sensors · Shows on OLED · Sends over WiFi     │
└───────────────────────┬──────────────────────────────────┘
                        │  HTTP POST every 5 seconds
                        ▼
┌──────────────────────────────────────────────────────────┐
│                 LAYER 2: ML SERVER LAYER                 │
│                                                          │
│         Python Flask API receives JSON data              │
│                        │                                 │
│          Random Forest Model runs prediction             │
│                        │                                 │
│        Returns: Normal / Warning / Critical              │
│                        │                                 │
│          Logs every reading to CSV file                  │
└───────────────────────┬──────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────────┐
│                 LAYER 3: ALERT LAYER                     │
│                                                          │
│   🟢 Green LED   → Normal (machine is healthy)          │
│   🟡 Yellow LED  → Warning (maintenance needed soon)    │
│   🔴 Red LED     → Critical (stop machine immediately)  │
│   🔔 Buzzer      → Sounds on Warning / Critical         │
│   📱 Blynk App   → Live gauges + push notifications     │
│   📺 OLED Screen → Real-time sensor values on device    │
└──────────────────────────────────────────────────────────┘
```

---

## 🔩 Components List

| Component | Quantity | Purpose |
|---|---|---|
| ESP32 Dev Board | 1 | Main controller + WiFi data transmission |
| MPU6050 (Accelerometer + Gyroscope) | 1 | Measures vibration and tilt of the machine |
| DHT22 Temperature & Humidity Sensor | 1 | Monitors motor temperature and environment |
| ACS712 Current Sensor (5A module) | 1 | Measures motor current draw |
| OLED Display 0.96" I2C (SSD1306) | 1 | Shows live sensor readings on the device |
| Red LED | 1 | Critical fault indicator |
| Yellow LED | 1 | Warning indicator |
| Green LED | 1 | Normal / healthy status indicator |
| Active Buzzer (5V) | 1 | Audible alarm for critical faults |
| Resistors 220Ω | 3 | Current limiting for each LED |
| Breadboard + Jumper Wires | 1 set | Prototyping and connections |
| 5V Power Supply / USB Power Bank | 1 | Power for standalone deployment |
| PC or Raspberry Pi | 1 | Runs the Python Flask ML server |



---

## ⚡ Circuit / Wiring Diagram

> 📷 *See `/circuit/wiring_diagram.png` for the full diagram.*

### How the Circuit is Organized

All sensors and output components are connected to the ESP32. The ESP32 uses two communication protocols to talk to its components:

- **I2C Protocol** (GPIO 21 = SDA, GPIO 22 = SCL) — used by MPU6050 and OLED display. Both share the same two wires but have different addresses (`MPU6050 = 0x68`, `OLED = 0x3C`), so they do not interfere with each other.
- **Digital/Analog GPIO** — DHT22 uses a single digital data pin, ACS712 outputs an analog voltage read by the ESP32's ADC, and LEDs/buzzer are controlled by digital output pins.

### Pin Connection Table

| Component | Component Pin | ESP32 Pin | Notes |
|---|---|---|---|
| MPU6050 | VCC | 3.3V | Power supply |
| MPU6050 | GND | GND | Ground |
| MPU6050 | SDA | GPIO 21 | I2C data line — shared with OLED |
| MPU6050 | SCL | GPIO 22 | I2C clock line — shared with OLED |
| DHT22 | VCC | 3.3V | Power supply |
| DHT22 | GND | GND | Ground |
| DHT22 | DATA | GPIO 4 | Requires 10kΩ pull-up to 3.3V |
| ACS712 | VCC | VIN (5V) | Needs 5V — use VIN pin |
| ACS712 | GND | GND | Ground |
| ACS712 | OUT | GPIO 34 (ADC1) | Analog voltage output |
| OLED SSD1306 | VCC | 3.3V | Power supply |
| OLED SSD1306 | GND | GND | Ground |
| OLED SSD1306 | SDA | GPIO 21 | Shared I2C — address 0x3C |
| OLED SSD1306 | SCL | GPIO 22 | Shared I2C — address 0x3C |
| Red LED | Anode (+) | GPIO 25 via 220Ω | Series resistor required |
| Red LED | Cathode (−) | GND | Ground |
| Yellow LED | Anode (+) | GPIO 26 via 220Ω | Series resistor required |
| Yellow LED | Cathode (−) | GND | Ground |
| Green LED | Anode (+) | GPIO 27 via 220Ω | Series resistor required |
| Green LED | Cathode (−) | GND | Ground |
| Buzzer | + | GPIO 32 | No resistor needed |
| Buzzer | − | GND | Ground |

### Important Wiring Notes

**MPU6050 + OLED on same I2C bus:**
Both the MPU6050 and the OLED display are connected to the same two wires (SDA and SCL). This is possible because I2C is a bus protocol that allows multiple devices on the same line, each identified by a unique address. MPU6050 uses address `0x68` and the OLED uses `0x3C` — the ESP32 calls each one individually by its address.

**DHT22 Pull-up Resistor:**
The DHT22 DATA pin needs a 10kΩ resistor connected between DATA and 3.3V. Without this pull-up, the data line floats and gives incorrect readings. The resistor keeps the line HIGH by default and the sensor pulls it LOW to communicate.

**ACS712 Current Sensor:**
The ACS712 works by measuring the magnetic field created by current flowing through it. It outputs an analog voltage centered at 2.5V (representing 0A). When current flows, the voltage goes above or below 2.5V depending on direction and magnitude. The formula to convert: `Current (A) = (Voltage - 2.5) / 0.185` for the 5A module.

**Why GPIO 34 for ACS712:**
On ESP32, the ADC2 pins (GPIO 0, 2, 4, 12–15, 25–27) conflict with WiFi when WiFi is active. GPIO 34 belongs to ADC1, which works reliably alongside WiFi — making it the correct choice for any analog sensor in an IoT project.

**LED Resistors:**
Each LED requires a 220Ω resistor in series between the GPIO pin and the LED anode. Without this, the GPIO pin would supply too much current (up to 40mA), which can permanently damage both the LED and the ESP32's GPIO pin. The 220Ω resistor limits the current to a safe ~10–15mA.

---

## 📡 How Each Sensor Works

### MPU6050 — Vibration & Acceleration Sensor
The MPU6050 is a 6-axis Inertial Measurement Unit (IMU) that contains a 3-axis accelerometer and a 3-axis gyroscope on a single chip. In this project, the accelerometer is used to detect vibration.

A healthy motor running normally produces low, steady vibration. As internal parts like bearings wear down, become misaligned, or begin to fail, the vibration intensity and pattern changes significantly. The MPU6050 captures this change by measuring acceleration on the X, Y, and Z axes. From these three values, we calculate a single **RMS (Root Mean Square)** value that represents the overall vibration intensity. We also calculate the **standard deviation** of vibration over the last 10 readings, which captures sudden spikes or instability.

### DHT22 — Temperature & Humidity Sensor
The DHT22 uses a capacitive humidity sensor and a thermistor to measure the ambient temperature and humidity. In this project it monitors motor body temperature. As a motor works harder than normal — due to overloading, bearing friction, poor ventilation, or electrical faults — it generates excess heat. A rising temperature trend is one of the earliest signs of impending failure. The DHT22 communicates over a single digital wire using a proprietary one-wire protocol and provides readings every 2 seconds.

### ACS712 — Current Sensor
The ACS712 is a Hall-effect based current sensor. It measures the actual current being drawn by the motor without breaking the circuit — the motor wire simply passes through the sensor. A healthy motor draws a relatively constant current at a given load. When a motor is about to fail (due to winding faults, overloading, rotor imbalance, or mechanical friction), its current draw increases noticeably. Conversely, if current drops suddenly, it could indicate a broken connection or phase loss. Monitoring current is one of the most reliable indicators of electrical health.

---

## 🤖 Machine Learning Model

### Why Machine Learning?

Simple threshold-based systems (e.g., "alert if temperature > 70°C") miss complex failure patterns that only appear when multiple sensor readings change together. For example, a slightly elevated temperature combined with increased vibration and higher current draw could signal a bearing fault — but none of the individual values might cross their threshold alone. Machine Learning can detect these combined patterns and classify them accurately.

### Algorithm — Random Forest Classifier

A Random Forest is an ensemble learning method that builds many decision trees during training and combines their votes to make a final prediction. Each tree is trained on a random subset of the data and features, which prevents overfitting and makes the model robust to noisy sensor data.

**Why Random Forest for this project:**
- Handles non-linear relationships between multiple sensor features naturally
- Resistant to outlier readings from sensors
- Provides a **feature importance** score showing which sensor contributes most to predictions
- Does not require sensor values to be normalized or scaled
- Gives a **confidence percentage** with each prediction, not just a class label
- Fast enough for real-time inference on a server

### Health Classification

| Class | Label | Meaning | Action Required |
|---|---|---|---|
| 0 | **Normal** | All sensor values within expected safe range | No action — continue monitoring |
| 1 | **Warning** | One or more values showing elevated readings | Schedule maintenance within 24–48 hrs |
| 2 | **Critical** | Readings indicate imminent failure | Stop machine immediately |

### Feature Engineering — 8 Model Inputs

Raw sensor readings alone are not always the best inputs to a model. We engineer additional features to better represent the machine's state:

| Feature | Source | What It Captures |
|---|---|---|
| `accel_x` | MPU6050 | Vibration on X-axis |
| `accel_y` | MPU6050 | Vibration on Y-axis |
| `accel_z` | MPU6050 | Vibration on Z-axis |
| `accel_rms` | MPU6050 | Overall vibration intensity (computed) |
| `temperature` | DHT22 | Motor body temperature |
| `humidity` | DHT22 | Ambient humidity |
| `current` | ACS712 | Motor current draw |
| `vibration_std` | MPU6050 | Vibration instability over last 10 readings (computed) |

`accel_rms = √((ax² + ay² + az²) / 3)` — combines all three axes into one value representing total vibration magnitude.

`vibration_std` — the standard deviation of the last 10 RMS readings. A stable motor has a low std dev. A motor with intermittent faults will show a high std dev due to sudden spikes.

### Dataset

| Property | Value |
|---|---|
| Total Samples | ~3,000 labeled readings |
| Normal samples | ~1,800 (60%) |
| Warning samples | ~750 (25%) |
| Critical samples | ~450 (15%) |
| Format | CSV — `data/equipment_data.csv` |
| Train / Test Split | 80% training / 20% testing (stratified) |

### Model Performance

```
              precision    recall  f1-score   support

      Normal       0.97      0.98      0.97       362
     Warning       0.93      0.91      0.92       148
    Critical       0.94      0.95      0.94        90

    accuracy                           0.96       600

  5-Fold Cross-Validation Accuracy: 0.954 ± 0.012
```

### Feature Importance
The model reveals that vibration and current are the two strongest predictors of failure, together contributing over 62% of the model's decision weight:

```
accel_rms      ██████████████████  38%  ← most predictive
current        ████████████        24%
temperature    ████████            17%
vibration_std  ██████              12%
accel_z / x/y  █████                9%
```

---

## 🌐 Dashboard & Alerts

### Blynk Mobile App

The Blynk app displays live sensor data from the ESP32 over WiFi. The following virtual pins are configured:

| Virtual Pin | Data Sent | Widget Used |
|---|---|---|
| V0 | Temperature (°C) | Gauge — range 0 to 100 |
| V1 | Vibration RMS | Gauge — range 0 to 20 |
| V2 | Current (A) | Gauge — range 0 to 10 |
| V3 | Prediction label (0 / 1 / 2) | LED indicator |
| V4 | Confidence (%) | Value display |

When a Warning or Critical prediction is made, the ESP32 calls `Blynk.logEvent()` which sends a push notification directly to the operator's phone — even if the app is closed.

### Physical Alerts on Device

| Condition | Red LED | Yellow LED | Green LED | Buzzer |
|---|---|---|---|---|
| Normal | OFF | OFF | ON | OFF |
| Warning | OFF | ON | OFF | OFF |
| Critical | ON | OFF | OFF | ON (continuous) |

---

## 📁 Folder Structure

```
industrial-iot-predictive-maintenance/
│
├── src/
│   └── predictive_maintenance.ino   ← ESP32 Arduino firmware
│
├── ml/
│   ├── train_model.py               ← ML training script
│   └── flask_server.py              ← REST API prediction server
│
├── data/
│   ├── equipment_data.csv           ← Labeled training dataset
│   └── predictions_log.csv          ← Live prediction log (auto-created)
│
├── model/
│   └── rf_model.pkl                 ← Saved trained model
│
├── circuit/
│   └── wiring_diagram.png           ← Full circuit diagram
│
├── results/
│   ├── confusion_matrix.png         ← Model evaluation plot
│   └── feature_importance.png       ← Feature ranking plot
│
├── requirements.txt                 ← Python dependencies
└── README.md
```

---

## ⚙️ How to Run

### Part 1 — ESP32 Firmware

1. Install these libraries in Arduino IDE via Library Manager:
   `Adafruit MPU6050`, `Adafruit Unified Sensor`, `DHT sensor library`, `Adafruit SSD1306`, `ArduinoJson`, `Blynk`

2. Open `src/predictive_maintenance.ino` and update the credentials:
   ```cpp
   const char* SSID       = "Your_WiFi_Name";
   const char* PASSWORD   = "Your_WiFi_Password";
   const char* SERVER_URL = "http://YOUR_PC_IP:5000/predict";
   char auth[]            = "Your_Blynk_Auth_Token";
   ```

3. Select `Tools → Board → ESP32 Dev Module`, choose the correct port, and click Upload.

### Part 2 — Python ML Server

```bash
# Install dependencies
pip install flask scikit-learn pandas numpy joblib matplotlib seaborn

# Step 1: Train the model (only needed once)
cd ml
python train_model.py

# Step 2: Start the prediction server
python flask_server.py
# Server will run at http://0.0.0.0:5000
```

---

## 🔮 Future Improvements

- [ ] **LSTM / Deep Learning** — Replace Random Forest with an LSTM neural network for better time-series pattern recognition across longer sequences
- [ ] **Edge Inference** — Run a compressed TensorFlow Lite model directly on the ESP32, removing the need for an external server
- [ ] **Remaining Useful Life (RUL) Prediction** — Add a regression model to estimate the exact number of hours remaining before failure
- [ ] **MQTT Protocol** — Replace HTTP POST with MQTT for lower latency, lower power consumption, and better scalability to multiple machines
- [ ] **Auto-Retraining Pipeline** — Automatically retrain the model as new confirmed labeled data is collected from the field
- [ ] **Multi-Machine Monitoring** — Scale the system to monitor 10+ machines simultaneously using an MQTT broker
- [ ] **Dockerize Server** — Package the Flask app and model into a Docker container for consistent deployment anywhere
- [ ] **PCB Design** — Replace the breadboard with a custom PCB designed in KiCad, with DIN rail mounting for installation inside a control panel

---

## 📄 License

This project is open-source under the [MIT License](LICENSE).

---

## 👤 Author

**Thirumala Rao Bommineni**
B.Tech ECE — Alturi Mastan Reddy Memorial College of Engineering & Technology
📧 thirumal.ofcl@gmail.com

---

> ⭐ If this project helped you, consider giving it a star on GitHub!
