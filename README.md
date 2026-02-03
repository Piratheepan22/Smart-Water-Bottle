# Smart Drinking Bottle | Consumption Tracking System💧
**A Fully Standalone, Sensor-Fused Hydration Tracker on Custom PCB**

## 📖 Project Overview
Developed as part of the **EE356 Product Design** module, this project is a complete embedded system designed to track daily fluid consumption with high precision.

We challenged the standard IoT approach: instead of relying on Bluetooth and mobile apps, we engineered a device that is **100% autonomous**. It solves the core engineering challenges of fluid tracking—motion artifacts, liquid transparency, and power reliability—through onboard sensor fusion and a custom hardware architecture.

## ⚡ Hardware Architecture (Chip-Down Design)
This project moves beyond modular prototyping. We designed a custom **SMD PCB** where components are reflowed directly for signal integrity and compact integration.

| Subsystem | Component | Implementation Detail |
| :--- | :--- | :--- |
| **MCU** | **ESP32 C3 Mini** | RISC-V core handling logic, I2C coordination, and NVS storage. |
| **Depth Sensor** | **VL53L0X** | Time-of-Flight (ToF) sensor for millimeter-precision level detection. |
| **Motion Sensor** | **BMI270** | Chip-down IMU integration for stability validation. |
| **Touch IC** | **TTP223** | Dedicated Touch Detector IC driving a custom copper electrode on the top of the bottle. |
| **Display** | **0.96" OLED** | Displays hydration stats, time, date, and battery status. |
| **Power** | **Li-Po / TP4056** | External charging circuit with ESP32 ADC voltage monitoring. |



## 🌟 Key Innovations & Features

### 1. The Floating Ring Mechanism (Universal Compatibility)
**The Problem:** Optical sensors (ToF) struggle with clear liquids (light passes through) or reflective liquids (light scatters).
**Our Solution:** We engineered a buoyant, opaque floating ring.
* **Mechanism:** The VL53L0X measures the distance to the *ring*, not the liquid.
* **Benefit:** This makes the bottle **Universal**. It can track water, juice, milk, or any other beverage with equal accuracy, regardless of the liquid's color or transparency.



### 2. Beyond Hydration: Digital Watch Functionality ⌚
Since this device is designed to sit on a user's desk all day, we integrated full time-keeping features.
* **OLED Dashboard:** The main screen displays the current **Time (HH:MM)** and **Date (DD/MM)** alongside hydration stats.
* **RTC Logic:** Uses a software-based Real-Time Clock to manage date rollovers (automatically resetting the hydration counter after 24Hrs fron limit set time).

### 3. Power Interruption Resilience (Zero Data Loss)
The system is built to handle the real world, where batteries die and devices lose power.
* **Non-Volatile Storage (NVS):** We utilize the ESP32 `Preferences` library to commit data to flash memory.
* **What is Saved:** Current Intake, Daily Goal, and User Settings.
* **Scenario:** If the battery runs out at 2:00 PM with 1.5L consumed, and you charge it back up at 3:00 PM, the device reboots remembering exactly where you left off.

### 4. Custom Capacitive Touch (TTP223)
Instead of mechanical buttons that can wear out or leak, we used a solid-state solution.
* **Hardware:** A **TTP223** Touch Detector IC is mounted on the PCB, connected to a copper pour acting as the sensor pad.
* **Tuning:** The sensitivity is tuned (via capacitor selection) to detect touches *through* the 3D-printed enclosure, offering a sleek, button-less experience.

## 🧠 The "Stability-First" Algorithm
To prevent false readings while the user is walking or drinking, the firmware runs a strict multi-stage filter:

1.  **Motion Gating (BMI270):** The IMU constantly monitors orientation. If the bottle is tilted >10° or acceleration deviates from gravity (1G), measurement is blocked.
2.  **Hysteresis (The 15s Rule):** Once the bottle is placed down, the system waits for a **15-second stability window**. This allows liquid sloshing and ripples to settle.
3.  **Statistical Sampling:** The VL53L0X takes samples for **10 seconds**, discarding outliers (noise) and averaging the valid readings to get a final, precise volume.

## ⚙️ Operating Modes

| Mode | Functionality | Reset Behavior |
| :--- | :--- | :--- |
| **Mode 1: Daily Goal** | Tracks intake against a user-set limit (e.g., 2.5L). | **Automatic:** Resets intake to 0 after 24 Hours from the time when limit set. |
| **Mode 2: Continuous** | Unrestricted tracking (e.g., for weekly stats). | **Manual:** Only resets when user requests. |

## 🎮 User Interface Controls
* **Single Tap:** Increment value / Next screen.
* **Double Tap:** Wake Screen / Confirm / Enter "Set Goal".
* **Triple Tap:** Switch between Mode 1 and Mode 2.

## 👥 Team & Credits
**EE356 Product Design Team:**
* **[Your Name]** – Firmware Architecture, PCB Layout, Algorithm Design.
* **Kavin** – [Role, e.g., Hardware Assembly & Mechanics]
* **Ekanayake** – [Role, e.g., Enclosure Design & Testing]

---
**License:** Open Source (MIT)
