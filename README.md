# ESPHome Zigbee ToF Awning Sensor

A smart, low-power, battery-operated distance sensor designed to monitor and control motorized garden awnings in real-time. 

By utilizing the **VL53L1X Time-of-Flight (ToF) laser sensor** combined with an **ESP32-H2 / ESP32-C6** microcontroller running **ESPHome via Zigbee (Zigbee2MQTT/ZHA)**, this project exposes the exact opening percentage directly to **Home Assistant**, even if your awning uses local wall switches or standard dry-contact relays.

### 🔋 Key Features
* **100% Wireless & Smart Power Management:** Optimized for battery operation with a custom daylight-conditioned Deep Sleep logic to prevent battery drain at night.
* **ToF Precision:** Measures the exact distance between the awning body and the front bar, converting raw millimeters into an accurate 0-100% position entity.
* **Fully Dynamic Configuration:** Calibration points (zero position and max length), sleep duration, and laser poll intervals are exposed as entities and can be modified on-the-fly directly from Home Assistant without reflashing.

## 📦 Bill of Materials (BOM)

| Component | Description | Qty | Notes |
| :--- | :--- | :--- | :--- |
| **Home Assistant** | Central smart home server | 1 | The main automation hub |
| **ESPHome & Zigbee2MQTT** | Firmware framework & Zigbee bridge | 1 | Handles local logic and Zigbee communication |
| **Waveshare ESP32-H2-Zero** | Ultra-compact Zigbee microcontroller | 1 | Based on the ESP32-H2FH4S chip with ceramic antenna |
| **VL53L1X** | Time-of-Flight (ToF) laser distance sensor | 1 | Range up to 4 meters (Long Mode) |
| **TP4056H Charging Module** | USB-C Li-Ion battery charger with protection | 1 | Must include double protection (6 pads, e.g., HW-107) |
| **18650 Li-Ion Battery** | 3.7V ricaricabile cell (3200 mAh) | 1 | *Unprotected* (flat top) model recommended for outdoors |
| **100kΩ Resistors** | Carbon or metal film resistors | 2 | 1% tolerance for the ADC voltage divider |

---

## 📌 Pinout & Hardware Layout (ESP32-H2-Zero)

According to the official specs of the [Waveshare ESP32-H2-Zero](https://waveshare.com), this board exposes essential pins in an ultra-small form factor. Below is the mapping used for this project:

| Board Pin | Native Function | Project Connection | Description |
| :--- | :--- | :--- | :--- |
| **5V** | Power Input (VCC) | Connected to **`OUT+`** of the TP4056H | Receives regulated battery power (3.7V - 4.2V) |
| **GND** | Ground | Connected to **`OUT-`** / Common Ground | Main reference ground for the entire system |
| **GPIO 6** | LP_I2C_SDA / GPIO | Connected to **`SDA`** of the VL53L1X | I2C Data line for the laser sensor |
| **GPIO 7** | LP_I2C_SCL / GPIO | Connected to **`SCL`** of the VL53L1X | I2C Clock line for the laser sensor |
| **GPIO 5** | MTMS / GPIO | Connected to **`XSHUT`** of the VL53L1X | Shutdown control pin to turn off the laser in Deep Sleep |
| **GPIO 1** | ADC1_CH0 / GPIO | Connected to the center of the divider | Analog pin used to measure battery voltage |

> 💡 **Power Saving Tip:** To fully eliminate parasitic power drain during Deep Sleep, it is highly recommended to desolder or cut the trace of the onboard **WS2812B** RGB LED. Otherwise, it will continuously draw around 1mA even when the chip is asleep.

---

## 🔌 Wiring List

All grounds must merge into a single logical point (**Common GND**). Follow this step-by-step wiring guide:

1. **Power & Charging Circuit:**
   * Battery `(+)` terminal ──► **`B+`** pad on the TP4056H module
   * Battery `(-)` terminal ──► **`B-`** pad on the TP4056H module
   * **`OUT+`** pad on the TP4056H ──► **`5V`** pin on the ESP32-H2-Zero **AND** **`VCC`** pin on the VL53L1X sensor
   * **`OUT-`** pad on the TP4056H ──► **`GND`** pin on the ESP32-H2-Zero **AND** **`GND`** pin on the VL53L1X sensor

2. **Laser Sensor Data Bus:**
   * Laser **`SDA`** ──► **`GPIO 6`** pin on the ESP32-H2-Zero
   * Laser **`SCL`** ──► **`GPIO 7`** pin on the ESP32-H2-Zero
   * Laser **`XSHUT`** ──► **`GPIO 5`** pin on the ESP32-H2-Zero

3. **Battery Monitor Voltage Divider:**
   * **`OUT+`** pad on the TP4056H ──► Input of the first 100kΩ resistor (**R1**)
   * Output of R1 ──► Input of the second 100kΩ resistor (**R2**) **AND** **`GPIO 1`** pin on the ESP32-H2-Zero
   * Output of R2 ──► **Common GND** (`OUT-`)

---

## ⚡ How to Flash via ESPHome

The ESP32-H2-Zero uses a native **USB Type-C port** (USB CDC) managed directly by the main chip without a dedicated UART chip. Follow these steps for the first-time compilation and flashing:

### 1. Environment Preparation
1. Open your **Home Assistant** instance and go to the **ESPHome** dashboard.
2. Click **New Device** and name it (e.g., `esphome-zigbee-tof-awning-sensor`).
3. Pick **ESP32** as the platform. When prompted, make sure to select the `esp-idf` framework (required for native Zigbee support on H2/C6 chips).
4. Copy the YAML configuration code provided in this repository and save it.

### 2. Initial Wired Flashing (Web Method)
Since the ESP32-H2 uses Zigbee instead of Wi-Fi, the initial flash must be done via a USB cable:
1. Connect the ESP32-H2-Zero to your PC using a proper USB-C data cable.
2. Inside ESPHome, click the three dots on your device and select **Install** ──► **Manual Download**.
3. Wait for compilation to finish. Once done, download the **Factory (`.bin`)** file.
4. Go to the official [ESPHome Web tool](https://esphome.io) using a Chromium-based browser (Chrome or Edge).
5. Click **Connect** and pick the COM port linked to the board (it will show up as *USB JTAG/serial debug unit*).
   * ⚠️ *If the board isn't detected:* Unplug the USB cable, press and hold the tiny onboard **BOOT** button, plug the cable back in, and release the button. Now try connecting again.
6. Select the downloaded `.bin` file and click **Install**. Wait for the process to complete.

### 3. Pairing with Zigbee2MQTT / ZHA
After flashing, the board will automatically reboot and start in Zigbee pairing mode:
1. Open your **Zigbee2MQTT** dashboard in Home Assistant.
2. Click **Permit Join (All)**.
3. Within a few seconds, the laser sensor will be discovered as a new Zigbee node, instantly exposing all 9 configured entities to your smart home.
