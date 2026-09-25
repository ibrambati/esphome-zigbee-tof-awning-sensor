# ESPHome Zigbee ToF Awning Sensor 🎪⚡

A smart, low-power, battery-operated distance sensor designed to monitor and control motorized garden awnings in real-time. 

By utilizing the **VL53L1X Time-of-Flight (ToF) laser sensor** combined with an **ESP32-H2 / ESP32-C6** microcontroller running **ESPHome via Zigbee (Zigbee2MQTT/ZHA)**, this project exposes the exact opening percentage directly to **Home Assistant**, even if your awning uses local wall switches or standard dry-contact relays.

### 🔋 Key Features
* **100% Wireless & Smart Power Management:** Optimized for battery operation with a custom daylight-conditioned Deep Sleep logic to prevent battery drain at night.
* **ToF Precision:** Measures the exact distance between the awning body and the front bar, converting raw millimeters into an accurate 0-100% position entity.
* **Fully Dynamic Configuration:** Calibration points (zero position and max length), sleep duration, and laser poll intervals are exposed as entities and can be modified on-the-fly directly from Home Assistant without reflashing.
