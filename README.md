# rpipico2-new

# DIY Meshastic Node (RP2350 + G-NiceRF LoRa2021F33-2G4)

This repository contains the hardware configuration and firmware variant definition for a custom **Meshastic** node built using the **Raspberry Pi RP2350** microcontroller paired with the **G-NiceRF LoRa2021F33-2G4** (2.4GHz) LoRa module.

## 🛠️ Hardware Specifications & Configuration

Based on the custom `variant.h` configuration, the pin mapping for the RP2350 to the LoRa module and peripherals is defined below.

### 📡 LoRa Module Pinout (G-NiceRF LR2021)
The module is driven by RadioLib using the `USE_LR2021` definitions, utilising a custom TCXO voltage setup and automated RF switching.

| Function | RP2350 GPIO Pin | Description |
| :--- | :---: | :--- |
| **LORA_SCK** | **10** | SPI Clock |
| **LORA_MISO** | **12** | SPI Master In Slave Out |
| **LORA_MOSI** | **11** | SPI Master Out Slave In |
| **LORA_CS / NSS** | **13** | SPI Chip Select |
| **LORA_RESET** | **7** | Reset Pin (`LR2021_NRESET_PIN`) |
| **LORA_DIO1** | **6** | Interrupt Request (`LR2021_IRQ_PIN`) |
| **LORA_DIO2** | **8** | Busy Indicator Pin (`LR2021_BUSY_PIN`) |

**Special RF Settings:**
* **TCXO Voltage:** Programmatically set to **1.6V** via `LR2021_DIO3_TCXO_VOLTAGE`.
* **RF Switch:** Managed automatically via `LR2021_DIO_AS_RF_SWITCH`.
* **IRQ Line:** Configured as `LR2021_IRQ_DIO_NUM 9`.

---

### 🔌 Peripherals & Modules

| Peripheral | GPIO Pin | Description |
| :--- | :---: | :--- |
| **I2C SDA** | **4** | Standard I2C Data (Display / Sensors) |
| **I2C SCL** | **5** | Standard I2C Clock (Display / Sensors) |
| **Serial TXD** | **8** | Recommended for SerialModule |
| **Serial RXD** | **9** | Recommended for SerialModule |
| **BUTTON_PIN** | **17** | User Input / Wake / Pair Button |
| **EXT_NOTIFY_OUT**| **22** | External Notification Output (Buzzer/Vibe) |
| **EXT_PWR_DETECT**| **24** | VBUS / External Power Detection |
| **LED_POWER** | `PIN_LED` | System Power Indicator Status LED |

---

### 🔋 Power & Battery Management

* **Battery Sense Pin:** Connected to **GPIO 29** (`BATTERY_PIN`).
* **ADC Resolution:** Configured for **12-bit** (`BATTERY_SENSE_RESOLUTION_BITS 12`).
* **ADC Multiplier:** **3.0** to correctly calculate scaled battery voltage divider circuits.
* **USB Power Handling:** `NO_BATTERY_LEVEL_ON_CHARGE` is explicitly defined to suppress false "100% battery" reports while operating purely via USB power (VSYS).

---

## 🚀 Firmware Compilation

To integrate this configuration into your local **Meshastic firmware** build environment:

1. Copy your custom configuration or modify the targets under `firmware/variants/` to include these pin mappings.
2. Ensure your build environment utilizes the RP2350 platform environment framework.
3. Verify that **RadioLib** is correctly targeted to build with **LR2021** (2.4GHz) support enabled.

