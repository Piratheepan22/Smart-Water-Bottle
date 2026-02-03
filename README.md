# Smart Drinking Bottle | Consumption Tracking System💧
**A Fully Standalone, Sensor-Fused Hydration Tracker on Custom PCB**

## 📖 Project Overview
Developed as part of the **Embedded Product Design** module under the Department of Electrical and Electronic Engineering, University of Peradeniya, this project is a complete embedded system designed to track daily fluid consumption with high precision.

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
* **Benefit:** This makes the bottle **Universal**. It can track water, juice, milk, Medical liquids (that requiring intake measurement) or any other beverage with equal accuracy, regardless of the liquid's color or transparency.
  
Here the image shows the design of Floating - Ring mechanism
![float-ring mechanism](images/design_3d/V1_1_8.png)


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

## 🧠 The "Stability-First" Logic Algorithm
The code implements a Finite State Machine (FSM) that prevents false readings caused by fluid dynamics (sloshing).

### Phase 1: Motion Gating (The BMI270 Watchdog)
The system polls the IMU every 20ms using a **Madgwick Filter** to calculate Pitch and Roll.
* **Tilt Threshold:** If the bottle is tilted > `10.0°` (configurable), measurements are blocked.
* **Motion Detection:** If acceleration vectors deviate from 1G (gravity), the system flags the state as `Unstable`.

### Phase 2: Hysteresis (The 15-Second Rule)

1.  **Stop Detected:** When the bottle becomes upright and still, a timer starts.
2.  **The Wait:** The system waits for a strict `STABILITY_WAIT_MS` of **15,000ms (15 seconds)**.
3.  **Re-Verification:** After 15 seconds, the code checks the IMU one last time. If the bottle moved *at all* during that window, the process aborts.

### Phase 3: Statistical Sampling (The 10-Second Scan)
Once stability is confirmed, the VL53L0X doesn't just take *one* reading. It enters a sampling loop:
* **Duration:** It scans the water level for **10,000ms (10 seconds)**.
* **Filtering:** It captures `MAX_SAMPLES` and calculates the mean.
* **Outlier Removal:** Any sample deviating more than `10mm` from the mean is discarded.
* **Result:** The final "Cleaned Average" is used to calculate volume.

## 🧮 Volume Calculation & Physics
The firmware calculates volume based on the differential distance to the liquid surface.

* **Refill vs. Drink Logic:**
    * **Consumption:** If distance *increases* (`Current > Last` by > 0.05dm), water was consumed.
    * **Refill:** If distance *decreases* significantly (`Current < Last` by > 0.2dm), the system detects a refill event and resets the baseline without adding to the "Consumed" total.
* **Unit Conversion:**
    * `Cross Section Area` = 0.4225 dm²
    * `Volume Delta` = `Delta Distance` × `Cross Section Area`

## ⚙️ Operating Modes

| Mode | Functionality | Reset Behavior |
| :--- | :--- | :--- |
| **Mode 1: Daily Goal** | Tracks intake against a user-set limit (e.g., 2.5L). | **Automatic:** Resets intake to 0 after 24 Hours from the time when limit set. |
| **Mode 2: Continuous** | Unrestricted tracking (e.g., for weekly stats). | **Manual:** Only resets when user requests. |

## 🎮 User Interface & Controls
Interaction is handled entirely through a **capacitive touch pad embedded in the bottle cap**. This button-free design maintains a sleek, waterproof profile while allowing full control over settings and modes.

Simply tap the top of the cap to wake the device or modify settings. The system recognizes specific tap patterns to navigate the menu:

| Gesture | Action | Description |
| :--- | :--- | :--- |
| **Single Tap (1x)** | **Increment / Next** | Cycles through menu options or increases the daily goal value. |
| **Double Tap (2x)** | **Wake / Confirm** | Wakes the OLED screen from sleep. Confirms a selection or enters "Set Goal" mode. |
| **Triple Tap (3x)** | **Switch Mode** | Toggles the device between **Daily Goal Mode** and **Continuous Tracking Mode**. |
| **Idle (2 sec)** | **Timeout / Cancel** | If no input is detected for 2 seconds, the screen sleeps or cancels the current setting change. |

## 👥 Team members
**Product Design Team:**
* **Piratheepan Mathivathanan** 
* **Sarusan Sivanesan**
* **E. M. T. L. K. Ekanayake** 
