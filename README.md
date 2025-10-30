ESP32 Bedside Patient Monitor

A simple, inexpensive wireless presence and motion monitoring system for patient care. Uses LD2410C mmWave radar with visual feedback via 8x8 RGB LED matrix. No internet connection required between sensor and receiver—communication via ESP-NOW protocol.

Project Overview
This project creates a non-intrusive patient monitoring solution that tracks presence, motion, and inactivity without cameras. A bedside LD2410C radar sensor wirelessly transmits energy data to a receiver with an LED ring display showing:

Outer ring (green): Stationary energy (patient presence)
Inner ring (blue): Moving energy (patient activity)
Center ring (red): Elapsed time since last movement (progressively brightens, then flashes)

Perfect for elderly care, post-surgery monitoring, or assisting caregivers without invasive surveillance.

Hardware Components
ComponentCostNotesESP32 Dev Module (x2)~$15Doit Kit or similarLD2410C mmWave Radar Sensor~$122ft detection range8x8 WS2812B RGB LED Matrix~$8Common addressable matrixUSB 5V Power Blocks (x2)~$12For ESP32 boards5V 2.5A Transformer~$10Powers LED matrixJumper wires, micro USB cables~$5Standard componentsTotal~$60Complete working system
Optional:

3D printed sensor enclosure (STL included)
3D printed matrix display box (STL included)
Hospital bedside table clamp (STL from Thingiverse)

Critical Hardware Notes
⚠️ Sensor Orientation: The LD2410C is highly directional. Mounting the sensor pointing away from the back is essential for reliable detection. Back-facing orientation shows almost no response.
⚠️ USB Cable Type: Use data+power cables, NOT charging-only cables. Many cheap cables lack data lines, causing communication failures.
Wiring Diagrams
Sender ESP32 (LD2410C Sensor)
LD2410C          ESP32
─────────────────────
5V       →       5V
GND      →       GND
TX       →       GPIO17
RX       →       GPIO16
Receiver ESP32 (LED Matrix)
WS2812B Matrix   ESP32
────────────────────
5V       →       5V (via 5V transformer)
GND      →       GND
DIN      →       GPIO32
Software Setup
Requirements

Arduino IDE: 1.8.19 (tested version)
ESP32 Board Package: 3.3.0 (or compatible)
Libraries:

FastLED (by Daniel Garcia)
ld2410 (by Limengdu)



Arduino IDE Installation

Add ESP32 Board Package:

File → Preferences
Paste in "Additional Boards Manager URLs": https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
Tools → Board Manager → Search "ESP32" → Install


Install Libraries:

Sketch → Include Library → Manage Libraries
Search and install:

FastLED (Daniel Garcia)
ld2410 (Limengdu)




Board Configuration:

Tools → Board → ESP32 Dev Module
Tools → Upload Speed → 115200
Tools → Flash Size → 4MB (32Mb)
Tools → Partition Scheme → Default 4MB with SPIFFS



Upload Steps

Program Receiver First

Connect receiver ESP32 via USB
Open receiver/receiver.ino
Upload (Sketch → Upload)
Open Serial Monitor (115200 baud)
Copy the MAC address printed (e.g., A4:CF:12:AB:CD:EF)


Program Sender

Connect sender ESP32 via USB
Open sender/sender.ino
Paste receiver's MAC address into receiverMAC[] array
Upload
Monitor Serial to verify LD2410 initialization


Test Connection

Sender should show: "movingEnergy: X  stationaryEnergy: Y"
Receiver matrix should light up
Wave hand in front of sensor to test



Configuration
All timing constants are #defined at the top of receiver.ino for easy tweaking:
cpp#define UPDATE_INTERVAL      100      // Matrix update frequency (ms)
#define TIMEOUT              3000     // No-data timeout (ms)
#define TIME_TO_FULL         10000    // Time for red to reach full (ms)
#define FLASH_PERIOD         500      // Red flash period when timeout (ms)
#define ENERGY_THRESHOLD     10       // Energy level to trigger red timer
#define OUTER_MULTIPLIER     1.8f     // Green ring sensitivity
#define INNER_MULTIPLIER     1.8f     // Blue ring sensitivity
Tuning Tips:

Increase multipliers if rings don't fill enough
Lower TIME_TO_FULL for faster red buildup
Adjust ENERGY_THRESHOLD based on sensor idle readings

Troubleshooting
ESP-NOW Connection Issues

"Failed to add peer": Receiver MAC address copied incorrectly—recheck Serial output
No data received: Ensure both boards are on same channel (default 0)
Intermittent connection: May be IDE/board version incompatibility—see versions below

LD2410 Not Working

"Initializing radar...FAILED":

Check UART pins (GPIO16/17)
Verify baud rate (256000)
Confirm data wires are connected (not charging-only cable)
Try different USB power source


Always reads ~100 energy:

Sensor may not be initialized—check Serial output
Verify sensor orientation (back should NOT face patient)



Matrix Not Lighting

No LEDs light: Check GPIO32 connection, verify FastLED library installed
Only partial light: Adjust multiplier constants upward
Wrong colors: Verify COLOR_ORDER is GRB (not RGB)

Known Working Configuration
Arduino IDE: 1.8.19
ESP32 Board: 3.3.0
Board: "ESP32 Dev Module"
Upload Speed: 115200
Flash Mode: QIO
Flash Freq: 80MHz
Partition Scheme: Default 4MB with SPIFFS (1.2MB APP/1.5MB SPIFFS)
File Structure
esp32-bedside-monitor/
├── sender/
│   └── sender.ino              # LD2410 sensor transmitter
├── receiver/
│   └── receiver.ino            # LED matrix receiver
├── 3d_prints/
│   ├── sensor_enclosure.stl    # Housing for LD2410C
│   ├── matrix_box.stl          # Display housing
│   └── bedside_clamp.stl       # Hospital table mount (Thingiverse)
├── README.md                   # This file
└── FUTURE_ML.md                # Machine learning expansion guide
Future Enhancements: Machine Learning
This project is designed to expand into predictive patient monitoring:
Short Term (No ML - Pure Thresholds)

Activity Baseline: Store 7-day average of hourly energy, alert on significant deviations
Stillness Detection: Alert if stationary >30min during normal wake hours
Sleep Pattern Learning: Detect unusual waking times (possible pain/confusion)
Circadian Analysis: Track sleep/wake cycles for deterioration signs

Medium Term (TinyML on ESP32)

TensorFlow Lite Micro: Run ML models directly on receiver
Edge Impulse: Easy-to-use platform for training on device
Fall Detection: Detect sudden zero energy → motion pattern
Breathing Analysis: Extract respiration patterns from subtle radar motion

Long Term (Cloud Analytics)

Data Logging: SD card storage of activity data
Pattern Recognition: Upload to cloud for advanced ML
Caregiver Alerts: Trend-based alerts (activity decline, sleep disruption)
Health Insights: Correlation with medical outcomes

Suggested ML Problems to Solve:

Predict patient deterioration 24hrs in advance
Distinguish between types of movement (normal vs distressed)
Detect falls with high confidence, low false positives
Identify delirium patterns (acute disorientation behavior)
Optimize alert timing to reduce caregiver fatigue

See FUTURE_ML.md for implementation roadmap.
How It Works

Sensor (Sender ESP32):

Reads LD2410C radar every 500ms
Extracts stationary and moving energy values
Transmits via ESP-NOW (no WiFi needed)


Receiver (ESP32):

Listens for incoming sensor packets
Maps energy to LED counts using square root curve
Updates matrix display every 100ms
Tracks elapsed time since last energy detected


Visual Feedback:

Outer ring fills with green (stationary presence)
Inner ring fills with blue (active movement)
Center ring glows red (grows brighter over 10 seconds, then flashes)
All LEDs dim after 3 seconds with no data (connection loss indicator)



Physical Setup
Sensor Placement

Position 2 feet from patient bed
Point sensor perpendicular to patient (not at an angle)
Avoid reflective surfaces directly behind sensor
Mount at chest height for best detection

Receiver Display

Place on bedside table facing caregiver
Position where visible from doorway/hallway
Use 3D-printed housing to protect matrix
Mount near call button for quick reference

Credits

Claude (Anthropic) - Code development, ESP-NOW implementation, troubleshooting
Limengdu - LD2410 Arduino library
Daniel Garcia - FastLED library
Thingiverse community - Bedside clamp design reference

License
MIT License - Feel free to use, modify, and distribute for personal or commercial use.
Contributing
Found a bug? Have improvements? Have questions about ML expansion?

Document the issue with your board/IDE versions
Include Serial output if applicable
Share your timing constant tweaks if they worked better

Contact & Support
If you build this project, please document and share your setup:

What board revision did you use?
Did different Arduino IDE versions cause issues?
What timing constants worked best for your patient use case?
Did you add any custom features?

This real-world feedback helps others tremendously.

