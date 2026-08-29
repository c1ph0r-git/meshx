# MeshX: Solar-Powered Dual-Band LoRa Bridge

**MeshX** is an autonomous, off-grid cross-band repeater and protocol bridge designed for MeshCore and Meshtastic communication networks. The name derives from **Mesh** (mesh networking) and **X**, representing both the cross-section between frequency bands and firmwares and an adaptable platform variable. MeshX enables continuous remote relay capabilities with extreme power efficiency.

Author: c1ph0r

---

## Key Features

* **Dual-Band Wireless Bridge:** Integrated dual EBYTE SX1262 LoRa modules covering sub-GHz bands (433MHz and 868MHz) for cross-band forwarding and mesh extension.
* **Solar Battery Management:** Onboard CN3163 solar charger paired with an integrated XB8089D0 Li-ion protection circuit.
* **Hardware UVLO Protection:** Active hardware under-voltage lock-out (HX810T @ 3.08V reset) prevents deep discharge battery degradation.
* **Switched Low-Side Power Gates:** Dedicated high-side AO3401A P-channel and SI2312CDS N-channel MOSFET gates for dynamic peripheral power management.
* **Hardware Interface Support:** Onboard Bosch BMP280 environmental sensor, dual thumbwheel configuration switches, passive buzzer, vibration motor driver, SPI E-Paper display header, two I2C display headers, and a dedicated GPS module interface.

---

## Power Path Architecture
```
Solar Panel (5V 1W) ---> [SOLAR Switch] ---> CN3163 Solar Charger
|
v
[Battery 2000mAh] <---> XB8089D0 BMS Circuit <---> BATTERY_POSITIVE
|
+---> HX810T UVLO (3.08V Cutoff)
|           |
v           v
+---> [NODE Switch] ---> BATTERY_LOAD
|
+---> nRF52840 MCU / Regulators
+---> Switched Power Gates (3.3V_1, 3.3V_2)
```

1. **Solar Input:** Designed for a 5V 1-2W solar panel connected via a 2-pin jst 1.25mm header (`+SOL-`) and passes through the `SOLAR` slide switch (`SK12D07VG3`) to the `CN3163` charger.
2. **Charging & BMS:** The `CN3163` manages constant-current/constant-voltage (CC/CV) charging. The `XB8089D0` provides onboard Li-ion over-charge, over-discharge, and short-circuit protection.
3. **Under-Voltage Lockout (UVLO):** An `HX810T-3.08V` voltage detector monitors `BATTERY_POSITIVE`. If battery voltage falls below 3.08V, it drives a P-MOSFET (`Q4`, `AO3401A`) to isolate `BATTERY_LOAD`.
4. **Switched Power Rail Distribution:** MCU-controlled MOSFET power gates (`1-PMOS`, `2-PMOS`) control power rails `3.3V_1` and `3.3V_2`, allowing individual power-down of the 433MHz (`U3`) and 868MHz (`U4`) transceiver blocks when idle, or isolation of peripherals while radio listening to drasticaly reduce quiescent battery consumption.

---

## System Pinout Reference (nRF52840 SuperMini)

| Functional Block | Signal Name | MCU / Schematic Net | nRF52840 GPIO / Pin | Description / Notes |
| :--- | :--- | :--- | :--- | :--- |
| **LoRa Radio 1 (433MHz)** | `1-BUSY` | GPIO / Interrupt | P0.03 | Radio 1 busy signal line |
| | `1-RST` | GPIO Output | P0.28 | Radio 1 active-low hardware reset |
| | `1-CS` | SPI Bus 1 Chip Select | P0.29 | SPI CS line for E22-400M22S |
| | `1-MISO` | SPI Bus 1 MISO | P0.02 | Shared SPI data input |
| | `1-MOSI` | SPI Bus 1 MOSI | P1.15 | Shared SPI data output |
| | `1-SCK` | SPI Bus 1 Clock | P1.13 | Shared SPI serial clock |
| | `1-RXEN` | GPIO Output | P0.10 | RX enable / RF switch control |
| | `1-DIO1` | GPIO Interrupt | P0.09 | Radio 1 interrupt line |
| | `1-PMOS` | Power Gate Control | P0.17 | Gate control for 3.3V_1 power rail (AO3401A) |
| **LoRa Radio 2 (868MHz)** | `2-BUSY` | GPIO / Interrupt | P0.24 | Radio 2 busy signal line |
| | `2-RST` | GPIO Output | P0.00 | Radio 2 active-low hardware reset |
| | `2-CS` | SPI Bus 2 Chip Select | P0.13 | SPI CS line for E22-900M22S |
| | `2-MISO` | SPI Bus 2 MISO | P0.20 | Shared SPI data input |
| | `2-MOSI` | SPI Bus 2 MOSI | P0.22 | Shared SPI data output |
| | `2-SCK` | SPI Bus 2 Clock | P1.00 | Shared SPI serial clock |
| | `2-RXEN` | GPIO Output | P0.15 | RX enable / RF switch control |
| | `2-DIO1` | GPIO Interrupt | P0.14 | Radio 2 interrupt line |
| | `2-PMOS` | Power Gate Control | P0.11 | Gate control for 3.3V_2 power rail (AO3401A) |
| **Sensors & I2C** | `1-SDA` | I2C Bus 1 Data | P0.26 | Primary I2C data (BMP280 U2, OLED Header [1]) |
| | `1-SCL` | I2C Bus 1 Clock | P0.31 | Primary I2C clock (BMP280 U2, OLED Header [1]) |
| | `2-SDA` | I2C Bus 2 Data | P0.06 | Secondary I2C data (OLED Header [2], Expansion) |
| | `2-SCL` | I2C Bus 2 Clock | P0.08 | Secondary I2C clock (OLED Header [2], Expansion) |
| **User Controls** | `1-Button A/B/C` | Analog / GPIO Matrix | P0.04 | Thumbwheel array 1 input (QS-301-AGS5P U6) |
| | `2-Button A/B/C` | Analog / GPIO Matrix | P0.05 | Thumbwheel array 2 input (QS-301-AGS5P U13) |
| | `1-RST_BTN` | MCU Hardware Reset | RESET (P0.18) | Dual hardware reset line (via D3/D4 diodes) |
| **Peripherals** | `GPS_RX` | UART RX | P0.01 | Dedicated GPS module serial receive |
| | `GPS_TX` | UART TX | P0.12 | Dedicated GPS module serial transmit |
| | `MOT` | GPIO Driver Output | P0.19 | Haptic vibration motor driver gate (Q2) |
| | `BUZZ` | PWM / GPIO Output | P0.16 | Passive buzzer audio driver gate (Q1) |
| | `3-BUSY` | SPI Bus / Control | P0.07 | E-Paper display busy line (EPAPER Header) |
| | `3-RES` | SPI Bus / Control | P0.21 | E-Paper display reset line |
| | `3-DC` | SPI Bus / Control | P0.23 | E-Paper display Data/Command selection line |
| | `3-CS` | SPI Bus / Control | P0.25 | E-Paper display SPI Chip Select line |

## Schematic Diagram

![schematic]("/schematic/schematic.png")


---

## Compliance with Radio Emission Regulations

The MeshX platform is designed to operate strictly within the legal framework established for license-free Industrial, Scientific, and Medical (ISM) and Short Range Device (SRD) radio frequency allocations.

* **Harmonized Sub-GHz Frequency Band Allocations:**
  * **433 MHz Operation:** Operating within the 433.050 MHz – 434.790 MHz band under **ETSI EN 300 220** (Europe) and equivalent international SRD regulations. Power outputs must adhere to maximum effective radiated power (ERP) limits (typically $\le 10\text{ mW}$ / $+10\text{ dBm}$ depending on sub-band and duty cycle).
  * **868 MHz / 915 MHz Operation:** Operating within the 863.000 MHz – 870.000 MHz band (EU ETSI EN 300 220, maximum $+14\text{ dBm}$ to $+22\text{ dBm}$ ERP depending on duty cycle channel access) and the 902.000 MHz – 928.000 MHz band (FCC Part 15.247 / 15.249 in Region 2).

* **Harmonic Suppression & RF Output Filtering:**
  * Both the E22-400M22S and E22-900M22S modules feature multi-stage LC low-pass impedance matching networks at the RF output pins to attenuate second and third harmonic emissions below ETSI ($-36\text{ dBm}$ for $f < 1\text{ GHz}$, $-30\text{ dBm}$ for $f > 1\text{ GHz}$) and FCC Part 15 limits.

* **Duty Cycle & Channel Access Rules:**
  * To maintain regulatory compliance without licensed spectrum access, the firmware layer (MeshCore / Meshtastic) enforces strict transmission duty cycle limits (e.g., $1\%$ or $0.1\%$ transmit limits per hour on ETSI channels) or utilizes Listen-Before-Talk (LBT) / Carrier Sense Multiple Access (CSMA) protocols to prevent co-channel interference.

* **Maximum Radiated EIRP / ERP Limitations:**
  * While the EBYTE SX1262 power amplifiers are capable of producing up to $+22\text{ dBm}$ ($160\text{ mW}$) conducted output power, system installers must configure system transmit power registers according to local country regulations, accounting for antenna gain:
$$\text{EIRP (dBm)} = P_{\text{conducted}} (\text{dBm}) - L_{\text{cable}} (\text{dB}) + G_{\text{antenna}} (\text{dBi})$$

---

## Component Selection Rationale

* **MCU Core (nRF52840 SuperMini):** Selected for its ultra-low deep sleep current draw, ARM Cortex-M4F core, native USB support, and Bluetooth Low Energy (BLE) capabilities.
* **LoRa Modules (EBYTE E22-400M22S & E22-900M22S):** Built on the Semtech SX1262 core, offering up to +22 dBm transmit power, excellent sensitivity, and lower current consumption compared to legacy SX127x series modules. This module is widely used in industrial applications and therefore is robust and widely available. 
* **Solar Charger (CN3163):** ESOP-8 solar power management IC featuring built-in internal adaptive MPPT performance for small photovoltaic panels. Compared to buck-boost MPPT alternatives it is more power efficient. Buck-boost chargers consume the power overhead of the MPTT in its circuit (small solar panel: MPPT efficiency gained lost by regulator in small current design). Moreover, it introduces less EMI noise to the sistem.
* **Protection Circuitry (XB8089D0 & HX810T-3.08V):** Compact SOIC-8 battery protection IC combined with an accurate SOT-23 supervisor to eliminate deep discharge scenarios in unattended installations. Redundant and bypassable design for safety purposes.
* **Sensor (BMP280):** Bosch BMP280 provides barometric pressure/temperature data for node telemetry.
* **Haptics (SMD9018):** SMD passive buzzer and vibration motor provide audible and haptic alert capabilities.
---

## Target Applications

* **Cross-Band Mesh Relay:** Autonomous bridging between 433MHz and 868MHz Meshtastic or MeshCore nodes, linking disparate local networks across extended geographic ranges.
* **Off-Grid Solar Repeater:** Solar-powered hill-top or rooftop repeater nodes requiring long battery runtime and zero maintenance.
* **Environmental Sensor Node:** Remote weather telemetry node streaming ambient temperature and pressure metrics over LoRa.
* **Portable Tactical Mesh Node:** Dual-frequency field communicator with user interface support via E-Paper/OLED displays and thumbwheel navigation.
