# 🛡️ EMRC-IDS — Embedded Multi-Band RF & CAN Intrusion Detection System

A standalone embedded security device that simultaneously monitors three wireless attack vectors in real-time — 2.4GHz RF, Sub-GHz RF, and CAN bus — displaying live threat levels on a TFT screen. Targeted at UAVs, military vehicles, and naval platforms.

---

## 📌 Overview

EMRC-IDS fuses data from three independent sensor modules into a single weighted threat score. It learns the normal communication baseline of any vehicle on first boot and flags anomalies in real time — no code modification required when switching between platforms.

---

## 🛠️ Components Required

| Component | Quantity |
|-----------|----------|
| ESP32 DevKit V1 (38-pin) | 1 |
| MCP2515 CAN Bus Module | 1 |
| NRF24L01 PA+LNA with Adapter | 1 |
| CC1101 Sub-GHz RF Module | 1 |
| ILI9341 2.8" TFT Display | 1 |
| Micro SD Card Module | 1 |
| Jumper Wires | As required |
| Breadboard / Custom PCB | 1 |
| 12V Power Supply or Battery | 1 |

---

## 🔌 Wiring — ESP32 Pin Connections

### Shared SPI Bus
| ESP32 Pin | Function |
|-----------|----------|
| GPIO 18 | SCK (all modules) |
| GPIO 19 | MISO (all modules) |
| GPIO 23 | MOSI (all modules) |

### MCP2515 (CAN Bus)
| MCP2515 | ESP32 |
|---------|-------|
| VCC | 3.3V |
| GND | GND |
| CS | GPIO 5 |
| SO | GPIO 19 |
| SI | GPIO 23 |
| SCK | GPIO 18 |
| INT | GPIO 35 |

> ⚠️ Use GPIO 35 for INT — not GPIO 2 (strapping pin, causes upload failures)

### NRF24L01 PA+LNA (via adapter)
| Adapter | ESP32 |
|---------|-------|
| VCC | **5V** (not 3.3V) |
| GND | GND |
| CE | GPIO 4 |
| CSN | GPIO 17 |
| SCK | GPIO 18 |
| MOSI | GPIO 23 |
| MISO | GPIO 19 |

> ⚠️ PA+LNA variant must use 5V via adapter

### CC1101 (Sub-GHz RF)
| CC1101 | ESP32 |
|--------|-------|
| VCC | 3.3V |
| GND | GND |
| CSN | GPIO 16 |
| SCK | GPIO 18 |
| MOSI | GPIO 23 |
| MISO | GPIO 19 |

### ILI9341 TFT Display
| TFT | ESP32 |
|-----|-------|
| VCC | 3.3V |
| GND | GND |
| CS | GPIO 15 |
| DC/RS | GPIO 25 |
| RST | GPIO 27 |
| SDI | GPIO 23 |
| SCK | GPIO 18 |
| LED | 3.3V |
| SDO | GPIO 19 |

---

## ⚙️ How It Works

### Detection Pipeline
```
[ UAV / Military Vehicle CAN Bus ]
         │
    [ MCP2515 ] ← passive listener
    [ NRF24L01] ← 2.4GHz channel sweep
    [ CC1101  ] ← Sub-GHz band sweep
         │
    [ ESP32 Threat Engine ]
         │
    [ ILI9341 TFT ]  ←→  [ SD Card ]
```

### Threat Scoring Formula
```
Total Score = (0.4 × RF_2.4G) + (0.4 × RF_SubGHz) + (0.2 × CAN)

Score 0–3  → LOW    (green)
Score 4–6  → MEDIUM (yellow)
Score 7–10 → HIGH   (red)
```

### Learning Mode
On first boot, the system listens silently for 30 seconds and builds a baseline of all CAN IDs and their normal frame rates. Detection begins automatically after learning completes. No code changes needed between vehicles.

---

## 🔍 Detection Capabilities

### CAN Bus (MCP2515)
| Detection | Method |
|-----------|--------|
| Unknown ID | Compares against learned baseline |
| Frame Flooding | Bus-wide frame rate > 200 fps |
| Sudden Value Jump | Byte delta > 80 in one frame |

### 2.4GHz RF (NRF24L01)
| Detection | Method |
|-----------|--------|
| RF Activity | 125-channel sweep with testCarrier() |
| Anomaly Scoring | Cumulative hits over 3-second window |

### Sub-GHz RF (CC1101)
| Detection | Method |
|-----------|--------|
| Signal Detection | RSSI sweep across 315 / 433 / 868 / 915 MHz |
| Dynamic Threshold | Auto-calibrated on boot from noise floor |

---

## 🖥️ TFT Dashboard

```
┌─────────────────────────────────────────┐
│ EMRC-IDS             MONITORING         │
│ RF 2.4G   4/10  [████░░░░░░░░░░░░]     │
│ RF SubGHz 6/10  [██████░░░░░░░░░░]     │
│ CAN Bus   2/10  [██░░░░░░░░░░░░░░]     │
│─────────────────────────────────────────│
│ THREAT: MEDIUM    Score: 4.0/10         │
│ LAST ALERT: Unknown ID 0x7FF            │
└─────────────────────────────────────────┘
```

---

## 💻 Serial Test Commands

| Command | Action |
|---------|--------|
| `u` | Simulate unknown CAN ID |
| `f` | Simulate CAN flood |
| `j` | Simulate value jump |
| `r` | Reset all scores to 0 |
| `s` | Print current status |
| `c` | Print raw CC1101 RSSI values |

---

## 📚 Libraries Required

Install via Arduino IDE → Library Manager:
- `MCP_CAN` by coryjfowler
- `RF24` by TMRh20
- `ELECHOUSE_CC1101_SRC_DRV`
- `Adafruit ILI9341`
- `Adafruit GFX`

---

## 🔧 Software Architecture

```
EMRC_IDS/
├── EMRC_IDS.ino          ← main loop + threat engine
├── can_ids.cpp/.h         ← MCP2515 detection
├── rf_24g.cpp/.h          ← NRF24L01 detection
├── rf_subg.cpp/.h         ← CC1101 detection
├── tft_ui.cpp/.h          ← display logic
├── sd_manager.cpp/.h      ← logging + profiles
└── config.h               ← all pin definitions + thresholds
```

---

## 🚀 Real-World Deployment

Connect the MCP2515 CAN-H and CAN-L to any vehicle's CAN bus (e.g. OBD-II port for cars, flight controller CAN port for UAVs). The system learns that vehicle's baseline on first boot and monitors passively — no active transmission, no bus interference.

---

## 🔮 Future Improvements

- SD card logging with timestamped CSV events
- Vehicle-agnostic baseline profiles saved to SD
- Custom PCB with aluminium enclosure for field deployment
- FreeRTOS task structure for real-time guarantees
- Real vehicle / UAV CAN bus validation
- Alert GPIO output for external buzzer or indicator

---

## 👤 Author

Dinesh Jakate
B.Tech Electronics Engineering | D Y Patil College of Engineering Akurdi
📧 dineshjakate14@gmail.com
