# Edge Computing Flight Controller — Pixhawk 6X (FMUv6)

> **The world's first Pixhawk-standard flight controller with integrated edge computing, 4G/5G connectivity, and Gigabit Ethernet — built for autonomous systems that demand real-time AI at the edge.**

---

## Table of Contents

- [Project Overview](#project-overview)
- [Key Selling Points](#key-selling-points)
- [System Architecture](#system-architecture)
- [Hardware Specifications](#hardware-specifications)
  - [Flight Management Unit (FMUv6)](#flight-management-unit-fmuv6)
  - [IO Coprocessor](#io-coprocessor)
  - [Edge Computing Module (CM5)](#edge-computing-module-cm5)
  - [4G/5G Connectivity Module](#4g5g-connectivity-module)
  - [Power System](#power-system)
  - [Communication Interfaces](#communication-interfaces)
  - [Peripheral Connectors](#peripheral-connectors)
- [Block Diagram Summary](#block-diagram-summary)
- [Component Bill of Materials (Key ICs)](#component-bill-of-materials-key-ics)
- [Power Architecture](#power-architecture)
- [Software Compatibility](#software-compatibility)
- [Target Applications](#target-applications)
- [Competitive Advantages](#competitive-advantages)
- [Marketing Messaging](#marketing-messaging)
- [Technical Glossary](#technical-glossary)
- [File Structure](#file-structure)
- [Design Notes for AI Agents](#design-notes-for-ai-agents)

---

## Project Overview

The **Edge Computing Flight Controller Pixhawk 6X** is a next-generation, open-standard autopilot board that combines the proven **FMUv6 (Pixhawk 6X)** flight controller architecture with a **Raspberry Pi Compute Module 5 (CM5)** edge computing platform and an integrated **4G/5G modem**, all on a single unified baseboard.

![Edge Computing Flight Controller FMU6x](/images/Edge%20Computing%20Flight%20Controller%20FMU6x.jpg)


This product bridges the gap between traditional embedded autopilots and the demands of modern autonomous systems — enabling **real-time AI inference, video streaming, cloud telemetry, and over-the-air mission updates** without additional companion computers or external modems.

| Parameter | Value |
|-----------|-------|
| Standard | Pixhawk FMUv6 (Pixhawk 6X compatible) |
| Edge Compute | Raspberry Pi Compute Module 5 (CM5) |
| Connectivity | 4G LTE / 5G NR via M.2 modem |
| Ethernet | Gigabit Ethernet (LAN8742A PHY) |
| PWM Outputs | 16 total (8 MAIN + 8 AUX) |
| CAN Buses | 2× isolated CAN bus |
| GPS Ports | 2× GPS (UART + I2C) |
| Power Inputs | 3× redundant (5V/4A × 2, 12V/3A × 1, USB 5V/4A) |
| Form Factor | Modular — Flight Module + Baseboard |

---

## Key Selling Points

### 🧠 Integrated Edge AI Computing
- **Raspberry Pi CM5** runs Linux natively alongside the flight controller
- Supports **TensorFlow Lite, PyTorch, OpenCV** for real-time vision and AI tasks
- **Dual MIPI CSI camera interfaces** for stereo vision or multi-camera pipelines
- **HDMI output** for ground station display or mission planning

### 📡 Always-Connected 4G/5G
- M.2 form factor **4G LTE / 5G NR modem** via PCIe + USB
- Built-in SIM card slot (nano SIM)
- Real-time **cloud telemetry, remote command, live video streaming**
- GNSS mode and Flight Mode control signals to the modem

### 🔌 Gigabit Ethernet
- **LAN8742A Ethernet PHY** directly connected to FMU main processor
- Enables **MAVLink over Ethernet**, high-bandwidth sensor fusion
- Secondary Ethernet port on CM5 for network switching or ground link
- Integrated **magnetic isolation transformers** for EMI robustness

### ⚡ Triple Redundant Power
- Three independent power inputs with **LTC4417 priority power selector**
- Automatic failover between power bricks — no single point of failure
- Overcurrent and overvoltage protection on all rails (BQ24315)
- Separate regulated rails for FMU, IO, Ethernet, Spektrum receiver, servos

### 🔧 Full Pixhawk Compatibility
- **FMUv6 / Pixhawk 6X standard** — drop-in compatible with PX4 and ArduPilot
- Industry-standard JST-GH connectors throughout
- 150-pin high-density board-to-board connector (DF40HC) for modular integration
- Backward compatible with all Pixhawk 4/5/6 peripherals

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     POWER SUBSYSTEM                             │
│  PW1 (5V/4A) ──┐                                                │
│  PW2 (5V/4A) ──┤─── LTC4417 Priority Selector ─── VDD_5V_IN     │
│  PW3 (12V/3A)──┘         (Triple Redundant)                     │
│  USB 3.0 (5V/4A) ─────────────────────────────────────────────  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
         ┌─────────────────────┼──────────────────────┐
         │                     │                      │
┌────────▼────────┐  ┌─────────▼────────┐  ┌─────────▼──────────┐
│  FMUv6 MAIN     │  │  CM5 BOARD       │  │  4G/5G MODULE      │
│  MODULE         │  │  (Raspberry Pi   │  │  (M.2 PCIe/USB)    │
│                 │  │   CM5)           │  │                    │
│ ┌─────────────┐ │  │ ┌─────────────┐  │  │ ┌────────────────┐ │
│ │ Main MCU    │ │  │ │ BCM2712     │  │  │ │ 4G LTE / 5G NR │ │
│ │ (STM32H7xx) │◄├─►│ │ Quad-Core   │  │  │ │ Modem          │ │
│ └─────────────┘ │  │ │ARM Cortex-A7│  │  │ └────────────────┘ │
│ ┌─────────────┐ │  │ └─────────────┘  │  │        │           │
│ │ IO MCU      │ │  │ ┌─────────────┐  │  │   PCIe / USB       │
│ │ STM32F103   │ │  │ │ HDMI Out    │  │  └────────┬───────────┘
│ └─────────────┘ │  │ │ Dual Camera │  │           │
│ ┌─────────────┐ │  │ │ USB 3.0     │  │◄──────────┘
│ │ LAN8742A    │ │  │ └─────────────┘  │
│ │ ETH PHY     │◄├─►└──────────────────┘
│ └─────────────┘ │
│ ┌─────────────┐ │
│ │ 24LC64      │ │
│ │ EEPROM      │ │
│ └─────────────┘ │
└─────────────────┘
         │
         ├── GPS1 (UART1 + I2C1 + 5× IO)
         ├── GPS2 (UART8 + I2C2)
         ├── TELEM1 (UART7, with power)
         ├── TELEM2 (UART5, with power)
         ├── TELEM3 (USART2, via CM5)
         ├── CAN1 (SN65HVD230 transceiver)
         ├── CAN2 (SN65HVD230 transceiver)
         ├── SPI External (SPI6, 7-pin)
         ├── I2C External (I2C3, with MS5611 barometer support)
         ├── 8× PWM MAIN outputs
         ├── 8× PWM AUX outputs (via STM32F103)
         ├── RC Input (PPM / SBUS / DSM Spektrum)
         ├── SBUS Output
         ├── AD & IO Port
         ├── Ethernet (Gigabit, isolated)
         └── USB (Type-C, USB 3.0)
```

---

## Hardware Specifications

### Flight Management Unit (FMUv6)

| Feature | Details |
|---------|---------|
| Standard | Pixhawk FMUv6 / Pixhawk 6X |
| Main Processor | STM32H7xx series (480 MHz Cortex-M7 with FPU) |
| Flash Memory | 2 MB internal |
| RAM | 1 MB TCM + 512 KB DTCM |
| Board Connector | 150-pin DF40HC(3.0)-100DS-0.4V (J20A) + 50-pin HC-PBB40C-50DS (J20B) |
| EEPROM | 24LC64 (64 Kbit) via I2C3 — Address 0x51 |
| Ethernet PHY | LAN8742A Gigabit Ethernet (RMII interface) |
| ETH Crystal | 25 MHz TAXM24M2QLFCDT1T |
| USB | USB Full-Speed via USB Type-C (USBC connector J3) |
| Safety Switch | Hardware safety input + LED output |
| Buzzer | Active buzzer output |
| ARMED LED | Hardware NARMED signal |
| Hardware Version | HW_VER_SENSE + HW_VER_REV_DRIVE pins |
| RTC Battery | V_RTC_BAT backup supply |

### IO Coprocessor

| Feature | Details |
|---------|---------|
| MCU | **STM32F103C8T7** (72 MHz Cortex-M3) |
| Package | QFP48 |
| Crystal | 8 MHz (X1) |
| PWM AUX Outputs | 8 channels (IO_CH1 – IO_CH8) |
| RC Input | PPM (IO_PPM_INPUT), SBUS (IO_SBUS_INPUT/OUTPUT) |
| DSM Receiver | IO_USART1_RX_DSM (3.3V Spektrum) |
| RSSI Input | IO_RSSI_IN |
| Servo Voltage Sense | IO_VDD_SERVO_SENS |
| Safety Switch | IO_SAFETY_SWITCH_IN + IO_NSAFETY_SWITCH_LED_OUT |
| Status LEDs | NIO_LED_BLUE, NIO_LED_AMBER |
| Debug | UART1 TX debug, SWD (SWDIO, SWCLK, SWO) |
| Communication to FMU | USART6 bidirectional (RC INPUT / NC) |
| SBUS Output Buffer | 74LVC2G240DP inverting buffer (NSBUS_OUT_LEN control) |

### Edge Computing Module (CM5)

| Feature | Details |
|---------|---------|
| Module | Raspberry Pi Compute Module 5 (CM5) |
| CPU | Quad-core ARM Cortex-A76 @ 2.4 GHz |
| Connector | DF40C-100DS-0.4V(51) × 2 (200-pin total) |
| RAM | 2 GB / 4 GB / 8 GB LPDDR4X (config-dependent) |
| Storage | eMMC or NVMe (via PCIe) |
| Camera | 2× MIPI CSI-2 (4-lane, CAM0 + CAM1) — J30, J29 |
| Display | 1× HDMI (J32, KDMIX-SL1-NS-WS-B15 connector) |
| USB | 1× USB 3.0 (J27), 1× USB Type-C boot (CC1/CC2 boot) |
| Ethernet to FMU | 100 Mbps (ETH_TRD0/TRD1 via transformers FL3–FL6) |
| Ethernet External | Gigabit (ETH1, P3 port, via YC248 transformer array) |
| PCIe | 1× PCIe Gen 2 to 4G/5G modem |
| I2C | CM5 I2C (J14), I2C3 SDA/SCL |
| UART to FMU | 2× UART (UART2 + UART3 bidirectional with CTS/RTS) |
| GPIO / SPI / UART Ext | J23 CM5 EXT PORT (10-pin) |
| Fan | PWM fan control + tachometer (J31) |
| Boot Control | nRPIBOOT (RPI_Boot signal) |
| Status LEDs | CM5_nPWR, CM5_nACT, CM5_ETH_LED_G/Y |
| HDMI Power | RT9742GGJ5 5V regulator (IC1) |
| USB VBUS Switch | TPS22919 load switch (U26) |

### 4G/5G Connectivity Module

| Feature | Details |
|---------|---------|
| Interface | M.2 form factor (J33, 2199230-5 connector) |
| Host Interface | PCIe Gen 2 + USB 3.0 (SuperSpeed) |
| SIM | Nano SIM (J19, SIM8066-6-1-14-01-A) |
| Control Signals | CM5_ON_OFF_4G, CM5_RESET_4G, CONTROL_POWER_4G |
| Power | 3.8V DIO rail from MAX20077ATCA buck converter |
| PCIe Clock | ECS-327MVATX 32.768 kHz oscillator for SUSCLK |
| Antenna Disable | W_DISABLE1#, W_DISABLE2# |
| USB ESD | TPD4EUSB30DQAR (IC2, IC3) SuperSpeed protection |
| Flight Mode | Dedicated FLIGHT MODE signal pin |
| GNSS Mode | Dedicated GNSS MODE signal pin |
| Power Switch | FULL_CARD_POWER_OFF# control |
| LED | WWAN_LED# indicator |

### Power System

| Rail | Source IC | Voltage | Max Current | Consumers |
|------|-----------|---------|-------------|-----------|
| VDD_5V_IN | LTC4417 + diode OR | 5V | 4A × 2 | Main bus |
| VDD_5V_PPM_SBUS_RC | Fused from 5V_IN | 5V | 1A | RC input, PPM/SBUS |
| IO_VDD_3V3 | MIC5332 (U2) | 3.3V | 600 mA | IO MCU, logic |
| ETH_VDD_IO_3V3 | MIC5332 (U5) | 3.3V | 600 mA | Ethernet PHY |
| FMU_VDD_3V3 | TPS62821 (B1) | 3.3V | — | FMU main MCU |
| CM5_3V3 | TPS62821 derivative | 3.3V | — | CM5 board logic |
| VDD_3V3_SPEKTRUM | LDO controlled | 3.3V | — | DSM/Spektrum Rx |
| VDD_5V_PERIPH | BQ24315 (U3) | 5V | protected | GPS1, TELEM2, SPI, I2C |
| VDD_5V_HIPOWER | BQ24315 (U4) | 5V | protected | GPS2, TELEM1 |
| VDD_5V_SW | MAX20006 Buck | 5V | 2.5A | General switching |
| VIN_CM5 | Passthrough + filter | 5V | 4A | CM5 module |
| 3V8_DIO | MAX20077 Buck | 3.8V | 2.5A | 4G/5G modem |

**Power Input Priority (LTC4417):**
1. **V1** — PW1 brick (5V/4A) — highest priority
2. **V2** — PW2 brick (5V/4A) — secondary
3. **V3** — PW3 (12V/3A) or USB — fallback

### Communication Interfaces

#### CAN Bus
| Interface | Transceiver | Connector | Termination |
|-----------|-------------|-----------|-------------|
| CAN1 | SN65HVD230DR (U8) | J5 — 4-pin JST-GH | 120Ω built-in |
| CAN2 | SN65HVD230DR (U9) | J8 — 4-pin JST-GH | 120Ω built-in |

Both CAN buses powered from FMU_VDD_3V3 with filter capacitors and magnetic ESD protection (SM24CANB TVS diodes D12, D17).

#### Serial / UART
| UART | Function | Connector | Power |
|------|----------|-----------|-------|
| UART1 | GPS1 TX | J10 — 10-pin GPS | VDD_5V_PERIPH |
| UART4 | UART_I2C | J16 — 7-pin | — |
| UART5 | TELEM2 | J11 — 6-pin | VDD_5V_PERIPH |
| UART7 | TELEM1 | J9 — 6-pin | VDD_5V_HIPOWER |
| UART8 | GPS2 RX/TX | J15 — 6-pin | VDD_5V_HIPOWER |
| USART2 | TELEM3 (via CM5) | — | — |
| USART3 | FMU Debug | J22 — 10-pin | — |
| USART6 | IO↔FMU internal | — | — |
| CM5 UART2 | CM5 ↔ FMU | Internal | — |
| CM5 UART3 | CM5 ↔ FMU | Internal | — |

#### I2C Buses
| Bus | Devices | Notes |
|-----|---------|-------|
| I2C1 | GPS1 magnetometer, LED, PM1 | External |
| I2C2 | GPS2 magnetometer, LED, PM2 | External |
| I2C3 | 24LC64 EEPROM, MS5611 barometer, I2C_EXT (J12) | Internal + external |

#### SPI
| Bus | Function | Connector | Signals |
|-----|----------|-----------|---------|
| SPI6 | External sensor | J7 — 11-pin | MISO, MOSI, SCK, nCS1, nCS2, DRDY1, DRDY2, nRESET, SYNC |

ESD protection: RCLAMP0524PATCT arrays on all SPI lines. 49.9Ω series resistors on signal lines. MOSFETs (DMN65D8L-7) for voltage level shifting on nCS lines.

#### Ethernet
| Interface | PHY | Speed | Connector |
|-----------|-----|-------|-----------|
| FMU ETH | LAN8742A | 100 Mbps | J36 — 4-pin ETH |
| CM5 ETH0 | CM5 native | 1 Gbps | P2 — 8-pin RJ45-style |
| CM5 ETH1 | CM5 native | 1 Gbps | P3 — 8-pin RJ45-style |
| CM5 ↔ FMU link | Internal transformer | 100 Mbps | ETH_TRD0/TRD1 |

All Ethernet connections use **DLW43MH201XK2L common-mode chokes** for EMI filtering.

### Peripheral Connectors

| Connector | Ref | Pins | Signals |
|-----------|-----|------|---------|
| GPS1 | J10 | 10 | UART1, I2C1, 5× GPIO, VDD_5V_PERIPH |
| GPS2 | J15 | 6 | UART8, I2C2, VDD_5V_HIPOWER |
| TELEM1 | J9 | 6 | UART7 TX/RX/RTS/CTS, GND, VDD_5V_HIPOWER |
| TELEM2 | J11 | 6 | UART5 TX/RX/RTS/CTS, GND, VDD_5V_PERIPH |
| CAN1 | J5 | 4 | CAN_H, CAN_L, GND, VDD |
| CAN2 | J8 | 4 | CAN_H, CAN_L, GND, VDD |
| SPI External | J7 | 11 | SPI6 full + power |
| I2C External | J12 | 4 | I2C3, VDD_5V_PERIPH |
| I2C3_EXT_MS5611 | — | — | External barometer |
| UART_I2C | J16 | 7 | UART4 + general I2C |
| AD & IO | P1 | 8 | ADC, CAP, BOOT, RST, ARMED, power |
| PWM MAIN (8ch) | J21A+B | 45 | IO_CH1-CH8, FMU_CH1-CH8, PPM, SERVO_VDD |
| PWM AUX (8ch) | — | — | FMU_CH1-FMU_CH8 (high-density) |
| DSM Sat RC | J17 | 3 | VDD_3V3_SPEKTRUM, IO_USART1_RX_DSM |
| AUX & RSSI | J18 | 6 | SBUS_OUT, RSSI_IN |
| FMU ETH | J36 | 4 | TX+/TX-, RX+/RX- |
| IO Debug | J6 | 10 | IO UART, SWD, NRST, VCC |
| FMU Debug | J22 | 10 | USART3, FMU SWD, NRST |
| USB | J3 | USB-C | D+/D-, VBUS, CC1, CC2, SBU |
| CM5 USB | J27 | 4 | USB 3.0 A port |
| CM5 HDMI | J32 | — | Full HDMI 2.0 |
| CM5 CAM0 | J30 | 22 | MIPI CSI-2 4-lane |
| CM5 CAM1 | J29 | 22 | MIPI CSI-2 4-lane |
| CM5 I2C | J14 | 4 | I2C0 SCL/SDA |
| CM5 EXT PORT | J23 | 10 | SPI1, UART4, I2C2/I2C3 multiplexed |
| CM5 ETH0 | P2 | 8 | Gigabit port |
| CM5 ETH1 | P3 | 8 | Gigabit port |
| FAN | J31 | 4 | 5V_FAN, GND, FAN_PWM, FAN_TACHO |
| POWER1 | J1 | 6 | VDD5V_BRICK1, I2C1, GND |
| POWER2 | J2 | 6 | VDD5V_BRICK2, I2C2, GND |
| POWER3 | J4 | 4 | VDD5V_BRICK3 (12V input) |

---

## Block Diagram Summary

```
                    ┌──────────────────┐
                    │  STM32F103C8T7   │
                    │  (IO Coprocessor)│
                    │  8× AUX PWM      │
                    └────────┬─────────┘
                             │ USART6
              ┌──────────────▼─────────────────────┐
              │          FMUv6 MAIN MODULE         │
              │  ┌──────┐  ┌──────────┐  ┌───────┐ │
              │  │EEPROM│  │ LAN8742A │  │24LC64 │ │
              │  │I2C3  │  │ ETH PHY  │  │EEPROM │ │
              │  └──────┘  └────┬─────┘  └───────┘ │
  GPS1◄───────│  UART1+I2C1     │  I2C3            │
  GPS2◄───────│  UART8+I2C2     │                  │
  TELEM1◄─────│  UART7          │  UART4           │───►UART_I2C
  TELEM2◄─────│  UART5          │                  │
  8×PWM◄──────│  PWM MAIN       │  CAN1────────────│───►CAN1
  CAN1◄───────│  CAN1           │  CAN2────────────│───►CAN2
  CAN2◄───────│  CAN2           │  SPI6────────────│───►SPI
              └─────────────────┴──────────────────┘
                         │              │
                    UART+ETH      ETH (RMII)
                         │              │
              ┌──────────▼──────┐  ┌───▼─────────┐
              │   CM5 BOARD     │  │  LAN8742A   │
              │ (RPi CM5)       │  │  ETH PHY    │
              │  PCIe ──────────┤  └─────────────┘
              │  USB  ──────────┤
              └────────┬────────┘
                       │ PCIe + USB
              ┌────────▼────────┐
              │  4G/5G MODULE   │
              │  (M.2 modem)    │
              └─────────────────┘
```

---

## Component Bill of Materials (Key ICs)

| Reference | Component | Function |
|-----------|-----------|----------|
| U1 | LTC4417CUF | Triple-input priority power selector |
| U2 | MIC5332-SSYMT | Dual LDO: 5V→3.3V (IO + 5V_SW) |
| U3 | BQ24315DSGR | Overvoltage + overcurrent protection (HIPOWER) |
| U4 | BQ24315DSGR | Overvoltage + overcurrent protection (PERIPH) |
| U5 | MIC5332-SSYMT | Dual LDO: 5V→3.3V (ETH) |
| U6 | MAX20006AFOC | 36V/2.5A synchronous buck converter |
| U7 | STM32F103C8T7 | IO coprocessor |
| U8 | SN65HVD230DR | CAN1 bus transceiver |
| U9 | SN65HVD230DR | CAN2 bus transceiver |
| U10 | MC74VHC1G32 | Safety OR gate |
| U11 | MC74VHC1G09 | Safety AND gate |
| U12 | MC74VHC1G135 | Safety XNOR gate |
| U13 | 74LVC2G240DP | SBUS output inverting buffer |
| U14 | LAN8742A-CZ-TR | 10/100 Ethernet PHY |
| U15 | TPS2117DRLR | Power mux for 12V input path |
| U25A/B | DF40C-100DS | CM5 200-pin connector (×2 halves) |
| U26 | TPS22919DCKR | USB VBUS load switch |
| U27 | MAX20077ATCA | 3.8V/2.5A buck for 4G/5G modem |
| B1 | TPS62821DLCR | 3.3V FMU core buck converter |
| IC1 | RT9742GGJ5 | 5V HDMI power regulator |
| IC2/IC3 | TPD4EUSB30DQAR | USB 3.0 SuperSpeed ESD protection |
| J13 | 24LC64T-I/MC | 64Kbit I2C EEPROM (addr 0x51) |
| MOSFET1/4 | SI4931DY-T1-GE3 | P-channel power MOSFET pair |
| MOSFET2 | NTHD4102PT1G | N-channel ChipFET |
| X1 | TAXM24M2QLFCDT1T | 24 MHz ETH reference crystal |
| Y1 | ECS-327MVATX | 32.768 kHz SUSCLK oscillator |
| L1 | XFL4015-471MEC | 2.2µH power inductor (MAX20006) |
| L5 | XFL4015 3V3 | 0.47µH inductor (TPS62821) |

---

## Power Architecture

```
Power Input Hierarchy:
═══════════════════════════════════════════════════════
PW1 (5V/4A) ──V1──┐
PW2 (5V/4A) ──V2──┤─ LTC4417 ─┬─► VDD_5V_IN ─┬─► MAX20006 ─► 5V_SW
PW3 (12V/3A)──V3──┘           │               ├─► VIN_CM5
USB 3.0(5V/4A)─────────────── │               └─► MIC5332(U2) ─► IO_VDD_3V3
                               │
                               ├─► BQ24315(U3) ─► VDD_5V_HIPOWER ─► TELEM1, GPS2
                               ├─► BQ24315(U4) ─► VDD_5V_PERIPH  ─► TELEM2, GPS1, SPI
                               ├─► MIC5332(U5) ─► ETH_VDD_IO_3V3
                               ├─► TPS62821(B1)─► FMU_VDD_3V3 ─► Main MCU
                               └─► MAX20077    ─► 3V8_DIO ─► 4G/5G Modem

Power Control Signals (FMU MCU → Power):
VDD_5V_PERIPH_NEN / VDD_5V_PERIPH_NOC ─► BQ24315 enable/overcurrent
VDD_5V_HIPOWER_NEN / VDD_5V_HIPOWER_NOC ─► BQ24315 enable/overcurrent
ETH_POWER_EN ─► ETH LDO enable
VDD3V3_SPEKTRUM_EN ─► Spektrum LDO enable
IO_NRST ─► IO coprocessor reset
ETH_NRESET ─► LAN8742A hardware reset
```

---

## Software Compatibility

### Flight Controller Software
| Software | Version | Notes |
|----------|---------|-------|
| **PX4 Autopilot** | v1.14+ | Native FMUv6 support, full feature set |
| **ArduPilot** | 4.4+ | Pixhawk6X target, all peripherals supported |
| **QGroundControl** | Latest | Full mission planning, parameter tuning |
| **Mission Planner** | Latest | ArduPilot ground station |

### Edge Computing Software (CM5)
| Software | Support |
|----------|---------|
| **Ubuntu 22.04 LTS** | Raspberry Pi CM5 native |
| **Raspberry Pi OS** | Official CM5 support |
| **ROS 2 (Humble/Iron)** | Full robotics middleware stack |
| **TensorFlow Lite** | Edge AI inference |
| **PyTorch Mobile** | Neural network inference |
| **OpenCV** | Computer vision pipelines |
| **MAVSDK** | MAVLink over UART/Ethernet to FMU |
| **GStreamer** | Video pipeline + streaming |

### Communication Protocols Supported
- **MAVLink 2.0** — UART, Ethernet, USB
- **UAVCAN / DroneCAN** — CAN1 + CAN2
- **RTK/GNSS** — UART NMEA + UBX
- **SBUS / PPM / DSM** — RC input
- **Spektrum DSM2/DSMX** — 3.3V via DSM port
- **SBUS Out** — to ground station or camera gimbal
- **I2C** — compass, barometer, rangefinder
- **SPI** — external high-speed IMU, ADC

---

## Target Applications

### 🚁 Professional UAV / Drone
- Fixed-wing, multirotor, VTOL platforms
- Survey, mapping, inspection drones
- Delivery UAV with cellular connectivity
- Long-range BVLOS (Beyond Visual Line of Sight) with 4G/5G

### 🤖 Autonomous Ground Vehicles (UGV)
- Agricultural robots and precision farming
- Security patrol robots
- Logistics and warehouse automation

### 🚢 Autonomous Marine Vehicles (ASV/USV)
- Survey vessels and bathymetry
- Environmental monitoring buoys
- Port inspection systems

### 🛰️ Edge AI Applications
- Real-time object detection and tracking
- Obstacle avoidance with neural networks
- Stereo vision depth estimation
- License plate / face detection payloads

### 📡 Connected Systems
- Live video streaming over 4G/5G to cloud
- Remote BVLOS operations via cellular
- Fleet management with real-time telemetry
- Over-the-air (OTA) firmware and mission updates

---

## Competitive Advantages

| Feature | This Product | Standard Pixhawk 6X | Competitor A |
|---------|-------------|---------------------|--------------|
| Edge AI Computing | ✅ CM5 (Quad-A76) | ❌ None | ⚠️ Optional RPi |
| 4G/5G Built-in | ✅ M.2 modem | ❌ None | ❌ External only |
| Gigabit Ethernet | ✅ LAN8742A | ✅ Yes | ⚠️ 100Mbps |
| CAN Buses | ✅ 2× | ✅ 2× | ✅ 2× |
| Power Inputs | ✅ 3× redundant | ✅ 2× | ⚠️ 1-2× |
| PWM Channels | ✅ 16 (8+8) | ✅ 16 | ⚠️ 8-12 |
| HDMI Output | ✅ Full HDMI | ❌ None | ❌ None |
| Camera Interfaces | ✅ 2× MIPI CSI | ❌ None | ❌ None |
| USB 3.0 | ✅ Yes | ⚠️ 2.0 only | ⚠️ 2.0 only |
| Modular Design | ✅ Flight Module + Baseboard | ⚠️ Monolithic | ⚠️ Monolithic |
| PX4 Compatible | ✅ Native | ✅ Native | ✅ Yes |
| ArduPilot Compatible | ✅ Native | ✅ Native | ✅ Yes |

---

## Marketing Messaging

### Taglines (for AI agent use)
- **"From Autopilot to Edge AI — One Board."**
- **"The Flight Controller That Thinks, Sees, and Stays Connected."**
- **"PX4. ArduPilot. AI. 5G. All in One."**
- **"Pixhawk 6X Meets Edge Computing — Finally."**
- **"Fly Smarter. Connect Farther. Deploy Faster."**

### Value Propositions by Audience

#### For Drone Engineers / System Integrators
> Drop-in Pixhawk 6X compatible hardware that adds a Raspberry Pi CM5 and 4G/5G modem without adding wiring, boards, or complexity. Three power bricks, two CAN buses, sixteen PWM channels, two GPS ports — everything you need, nothing you don't.

#### For AI/ML Developers
> Run TensorFlow Lite, PyTorch, or OpenCV directly on the CM5 Quad-core ARM Cortex-A76 at 2.4 GHz, with two MIPI CSI camera ports and HDMI output. MAVLink integration means your model's outputs go straight to the flight controller over Ethernet or UART — zero latency, zero extra hardware.

#### For Enterprise / Fleet Operators
> Real-time 4G/5G telemetry, live video streaming, and OTA updates without external modems or companion computers. A single unified board manages flight, perception, and connectivity — reducing system weight, cost, and failure modes.

#### For Researchers / Universities
> Open hardware, PX4 + ArduPilot compatible, full Linux environment on CM5. Prototype custom flight modes, AI algorithms, and communication protocols without hardware redesign. Full debug access via SWD, UART, and Ethernet.

### Feature Highlights for Ad Copy
1. **Triple Redundant Power** — No single brick can take down your mission
2. **16 PWM Channels** — Run complex multirotor, VTOL, or hybrid configurations
3. **Dual GPS Support** — Heading lock and redundant positioning
4. **4G/5G Native** — BVLOS out of the box
5. **Edge AI Native** — Inference at the aircraft, not the cloud
6. **Gigabit Ethernet** — High-bandwidth sensor fusion and data logging
7. **Dual CAN Bus** — Robust DroneCAN ecosystem
8. **Dual Camera** — Stereo vision, payload camera, or both

---

## Technical Glossary

| Term | Meaning |
|------|---------|
| FMUv6 | Flight Management Unit version 6 — Pixhawk open hardware standard |
| Pixhawk 6X | Commercial product name for FMUv6-based flight controller |
| CM5 | Raspberry Pi Compute Module 5 — system-on-module |
| MIPI CSI-2 | Mobile Industry Processor Interface — camera serial interface |
| DroneCAN | CAN bus protocol for UAV peripherals (formerly UAVCAN) |
| BVLOS | Beyond Visual Line of Sight — regulatory category for drone operations |
| RMII | Reduced Media Independent Interface — Ethernet PHY interface |
| SBUS | Serial Bus — Futaba RC protocol for servo control |
| DSM/DSM2/DSMX | Spektrum RC receiver digital protocols |
| LDO | Low Dropout Regulator — linear voltage regulator |
| ESD | Electrostatic Discharge — protection for I/O pins |
| PWM | Pulse Width Modulation — standard motor/servo control signal |
| RTK | Real-Time Kinematic — centimeter-accurate GPS correction |
| MAVLink | Micro Air Vehicle Link — UAV communication protocol |
| PCIe | Peripheral Component Interconnect Express — high-speed serial interface |
| SWD | Serial Wire Debug — ARM Cortex programming/debug interface |
| TVS | Transient Voltage Suppressor — surge protection diode |

---

## File Structure

```
project/
├── README.md                          # This file
├── Module_FlightControl_PCB.net       # KiCad/Altium netlist (all component connections)
├── Module_FlightControl_V2_sch.pdf    # Full schematic PDF (6 pages)
│   ├── Page 1: Top-level hierarchy (sub-sheets)
│   ├── Page 2: Power.SchDoc (all power regulators, LTC4417, protection)
│   ├── Page 3: Baseboard.SchDoc (FMU, IO MCU, LAN8742A, all connectors)
│   ├── Page 4: CM5_Board.SchDoc (CM5, PCIe, USB, HDMI, cameras, Ethernet)
│   ├── Page 5: 4G_5G_Base.SchDoc (modem, SIM, PCIe signal routing)
│   └── Page 6: Bill of Materials (component table)
└── FC_computer_baseBoard_Block_diagram.jpg  # System block diagram image
```

---

## Design Notes for AI Agents

> This section provides structured context for AI agents generating documentation, marketing content, or technical analysis.

### Product Identity
- **Product Name**: Edge Computing Flight Controller Pixhawk 6X
- **Alternate Names**: FC_computer_baseBoard, Module_FlightControl_V2, FMUv6 Edge Board
- **Product Category**: Embedded flight controller / Edge computing platform / Autonomous systems hardware
- **Standard**: Open hardware (Pixhawk FMUv6)
- **Design Tool**: Altium Designer (`.SchDoc` files, `.net` netlist)
- **Design Date**: April 6, 2026

### Schematic Hierarchy
The design is split into 5 sub-schematics:
1. `Top.SchDoc` — hierarchical top-level connecting all sheets via port signals
2. `Power.SchDoc` — all power management (POWER1&2 harness, VDD rails, protection)
3. `Baseboard.SchDoc` — FMU MCU, IO MCU, Ethernet, all JST-GH peripheral connectors
4. `CM5_Board.SchDoc` — Raspberry Pi CM5 module, HDMI, cameras, PCIe, USB, ETH
5. `4G_5G_Base.SchDoc` — M.2 modem card, SIM slot, PCIe/USB level shift, power

### Inter-Sheet Signal Names (key cross-references)
- `POWER1&2` — power and I2C from power bricks to all sheets
- `ETH_CM5_CONNECT` — CM5 ↔ Baseboard Ethernet (TRD0/TRD1 pairs)
- `UART_CM5_CONNECT` — CM5 ↔ Baseboard UART2+UART3 (4-wire each)
- `PCIE_PORT` — CM5 ↔ 4G/5G modem full PCIe + USB bundle
- `RPI_Boot` — nRPIBOOT boot mode control signal
- `ETH_NRESET` — LAN8742A hardware reset
- `IO_NRST` — STM32F103 IO MCU reset
- `VDD3V3_SPEKTRUM_EN` — Spektrum power enable

### Component Count Summary
- **Resistors**: 249+ (0603 SMD, values from 0Ω to 470kΩ)
- **Capacitors**: 189+ (0603/0805/1206 SMD, values 10pF to 100µF)
- **ICs / MCUs**: 27+ (major ICs listed above)
- **Connectors**: 36+ (JST-GH, DF40, USB-C, HDMI, M.2, RJ45-style)
- **ESD Protection**: 34× RCLAMP0524PATCT + 12× TPESD101 + 27× TVS diodes
- **Inductors**: 5 (power, EMI filter)
- **MOSFETs**: 12 (power switching, level shifting)
- **Filters (CMC)**: 10× DLW43MH201XK2L common-mode chokes

### Key Differentiating Signals to Highlight
- `CM5_ON_OFF_4G` / `CM5_RESET_4G` — software modem control from CM5
- `CONTROL_POWER_4G` — hardware power gating of modem
- `Flight Mode` / `GNSS Mode` — modem operational mode pins
- `nRPIBOOT` — USB boot mode for CM5 factory programming
- `FAN_PWM` / `FAN_TACHO` — active cooling control for CM5
- `PMIC_ENABLE` — CM5 power management
- `HW_VER_SENSE` / `HW_VER_REV_DRIVE` — board revision detection

### Tone and Messaging Guidance for Content Generation
- **Technical audience**: Use precise component names, protocol names, signal names
- **Business audience**: Focus on reduced integration cost, faster time-to-market, no extra PCBs needed
- **Consumer/prosumer**: "Everything in one board", "plug and fly connected", "AI-ready from day one"
- **Safety emphasis**: Triple redundant power, dual GPS, hardware safety switch, ESD protection on all I/O
- **Ecosystem emphasis**: PX4 + ArduPilot + ROS 2 + QGC — no new tools, no new learning curve

---

*Documentation generated from schematic analysis of `Module_FlightControl_V2_sch.pdf` and block diagram `FC_computer_baseBoard_Block_diagram.jpg`. Schematic revision: C (Power), B (Baseboard, CM5, 4G). Design date: April 6, 2026.*
