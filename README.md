# rpipico2-new

# Custom Meshtastic DIY Node (RP2350 + G-NiceRF LoRa2021F33-2G4)

This repository contains the firmware configurations and building blocks for a custom DIY **Meshtastic node** utilizing the high-performance **Raspberry Pi Pico 2 (RP2350)** microcontroller paired with the **G-NiceRF LoRa2021F33-2G4** (SX1280/LR2021 based) 2.4 GHz wireless module.

---

## 🖼️ Hardware Gallery

### Unassembled Components & PCBs
![Unassembled Hardware Components]

<img width="1848" height="2621" alt="image" src="https://github.com/user-attachments/assets/db6e1c4b-e6ea-4334-a5a8-f9d440fd9a51" />

*Left: Raspberry Pi Pico 2 (RP2350) microcontrollers in packaging. Right: Custom integration carrier PCBs alongside the G-NiceRF LoRa2021F33-2G4 modules and antenna.*

### Completed Assembly & Test Run
![Assembled Running Node]

<img width="3198" height="1036" alt="image" src="https://github.com/user-attachments/assets/8f381764-eb2d-446d-835f-2a57db9acf3a" />

*The fully soldered node featuring the RP2350 sandwich-mounted onto the carrier board, powered over USB, and connected to the 2.4 GHz SMA antenna pigtail.*
## NOTE: RP2350 is mounted one row off board !

<img width="780" height="1128" alt="image" src="https://github.com/user-attachments/assets/acbce883-e03f-4393-850e-6693ffc9a368" />

<img width="1036" height="345" alt="image" src="https://github.com/user-attachments/assets/e1447eca-6108-4db5-a86d-5339ae5ff426" />

---

## 🛠️ Hardware Credits & Attribution
The custom breakout / sandwich carrier board layout used in this build is based on the open-source KiCad design by **jlpoltrack**. 

* **Original Repository:** [jlpoltrack/Pi-Pico-NiceRF-LR2021](https://github.com)
* **Design Usage:** The production Gerbers and footprint layout configurations from their repository were utilized to create the compact carrier PCB that adapts the Raspberry Pi form factor directly to the G-NiceRF surface-mount module footprint.

---

## 📂 Source Tree Integration

To map this target into the official Meshtastic firmware source tree, place your configuration file within the target architecture variants folder:

```text
firmware/
└── src/
    └── variants/
        └── pico2_lr2021/
            └── variant.h  <-- Place your configuration file here
```

---

## 📌 Pin Configuration (`variant.h`)

The module communicates over a dedicated SPI bus using the RadioLib abstraction framework. Below is the precise map configured in this project's target `variant.h` file:

### LoRa SPI & Radio Control Bus (G-NiceRF LR2021)

| Signal Name | RP2350 GPIO Pin | Description |
| :--- | :---: | :--- |
| **LORA_SCK** | **10** | SPI Clock |
| **LORA_MOSI** | **11** | SPI Master Out Slave In |
| **LORA_MISO** | **12** | SPI Master In Slave Out |
| **LORA_CS** / NSS | **13** | SPI Chip Select |
| **LORA_RESET** | **7** | Hardware Reset Pin |
| **LORA_DIO1** (IRQ) | **6** | Radio Interrupt Pin (`LR2021_IRQ_PIN`) |
| **LORA_DIO2** (BUSY) | **8** | Radio Busy State Indicator (`LR2021_BUSY_PIN`) |
| **LORA_DIO0** | *RADIOLIB_NC* | Not Connected |
| **LORA_DIO3** | *RADIOLIB_NC* | Not Connected |

### Core System & Power Architecture Flags
* **TCXO Voltage Control:** Activated via `#define LR2021_DIO3_TCXO_VOLTAGE 1.6` to feed a stable **1.6V** reference directly to the module's crystal oscillator.
* **RF Switch Management:** Automated handling enabled using the `#define LR2021_DIO_AS_RF_SWITCH` rule.
* **USB Power Suppression:** `#define NO_BATTERY_LEVEL_ON_CHARGE` is enabled alongside an ADC multiplier of `3.0` on pin `29`. This explicitly suppresses a false "100% full battery" UI report inside Meshtastic clients while the device runs solely on USB VBUS power without a real physical cell connected.

### Peripheral / Extension Breakouts
* **I2C Interface (Screen / Sensors):** SDA = **GPIO 4** | SCL = **GPIO 5**
* **Serial Module (GPS / Telemetry):** TXD = **GPIO 8** | RXD = **GPIO 9**
* **User Input & Indicators:** User Button = **GPIO 17** | Status LED = `PIN_LED`
* **Notification Output:** **GPIO 22** (`EXT_NOTIFY_OUT`)
* **External Power Sense:** **GPIO 24** (`EXT_PWR_DETECT`)

---

## ⚡ Build & Compile Prerequisites
When compiling this firmware flavor using PlatformIO, ensure your environment target points to the target RP2350 system architecture. Due to the custom radio configuration stack, the following baseline parameters are applied during pre-compilation:
```cpp
#define ARDUINO_ARCH_AVR // Architecture fallback mapping descriptor
// RadioLib overrides
// #define RADIOLIB_CUSTOM_ARDUINO 1 
// #define RADIOLIB_TONE_UNSUPPORTED 1 
// #define RADIOLIB_SOFTWARE_SERIAL_UNSUPPORTED 1
```

---

## 🎛️ Diagnostic Verification

If you experience hardware initialization or communication faults, check the following software behaviors:

1. **TCXO Lock Verification:** If the radio fails to initialize (`Radio initialization failed` or `Error -2` loop in the logs), verify if the module requires closer to `1.8` instead of `1.6` volts for the internal oscillator, or ensure the line isn't floating.
2. **Busy Pin Inversion:** The `#define LR2021_BUSY_PIN LORA_DIO2` relies on a fast state change line. Ensure no solder bridge exists across the fine pitched pins on the G-NiceRF module between pins 6, 7, and 8.
3. **Serial Terminal Monitor:** Hook up a serial debugger to verify that the node isn't continuously looping through watchdogs due to missing I2C devices on pins 4/5 if standard modules are expected by default configurations.

---

## 💾 Flashing Guide

To push your compiled `.uf2` binary to the custom target:

1. **Enter Bootloader Mode:** 
   * Press and hold the physical **BOOTSEL** button on the Raspberry Pi Pico 2.
   * Connect the device to your computer via a Micro-USB/USB-C cable.
   * Release the **BOOTSEL** button once the device mounts.
2. **Identify the Drive:** The board will enumerate on your operating system file manager as a mass storage volume named **RP2-V3** (or similar RP2350 standard designation).
3. **Deploy the Firmware:** Drag and drop your compiled `firmware.uf2` binary directly into the root folder of this drive. The drive will automatically unmount, flash the flash memory storage blocks, and reboot as an active Meshtastic mesh participant.

---

## ☕ Support My Work
If this open hardware project brings utility to you, consider supporting my design pipeline!

[![Buy Me A Coffee](https://img.shields.io/badge/buy_me_a_coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/mark.birss)
