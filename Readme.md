# 🌱 Smart Garden Controller & Environmental Monitoring System


  STM32F407VET6 | ARM Cortex-M4 | Embedded C | STM32 HAL


An embedded smart garden system based on the **STM32F407VET6 microcontroller** that monitors environmental conditions and automatically controls irrigation, ventilation, and lighting.

The system integrates multiple sensors, relay-based actuators, an SPI TFT display, real-time clock functionality, and a push-button interface. It supports both automatic and manual operating modes, with configurable environmental thresholds and built-in pump safety protection.

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Features](#-features)
- [Hardware Components](#-hardware-components)
- [Hardware Pin Configuration](#-hardware-pin-configuration)
- [Firmware Architecture](#-firmware-architecture)
- [Operating Modes](#-operating-modes)
- [Pump Safety Mechanism](#-pump-safety-mechanism)
- [Task Scheduling](#-task-scheduling)
- [Development Environment](#-development-environment)
- [Getting Started](#-getting-started)
- [Project Structure](#-project-structure)
- [Future Improvements](#-future-improvements)
- [Technical Skills Demonstrated](#-technical-skills-demonstrated)

---

## 🌿 Project Overview

The Smart Garden Controller is designed to monitor and maintain suitable environmental conditions for plant growth.

The STM32F407VET6 collects data from environmental sensors, processes the measurements, and controls external devices according to predefined thresholds.

Sensor readings and system states are displayed on a 3.2-inch TFT LCD using the ILI9341 controller.

Users can interact with the system through four physical push buttons to configure operating parameters, switch operating modes, and manually control connected devices.

### System Architecture

```text
        ENVIRONMENTAL SENSORS
       ┌──────────────────────┐
       │                      │
       │  DHT11               │
       │  HW-390              │
       │  LM393               │
       │                      │
       └──────────┬───────────┘
                  │
                  ▼
       ┌──────────────────────┐
       │                      │
       │   STM32F407VET6      │
       │                      │
       │  Sensor Processing   │
       │  Control Logic       │
       │  Safety Management   │
       │  User Interface      │
       │                      │
       └───────┬──────────────┘
               │
       ┌───────┼──────────────┐
       │       │              │
       ▼       ▼              ▼
    ILI9341  Push Buttons   RTC
     LCD      UP/DOWN      Backup
              SET/NEXT     Registers
               │
               ▼
       ┌──────────────────────┐
       │   RELAY CONTROLLER   │
       └──────────┬───────────┘
                  │
          ┌───────┼────────┐
          ▼       ▼        ▼
        Pump     Fan      Light
```

---

## ✨ Features

### 1. Environmental Monitoring

- Measure ambient temperature using DHT11.
- Measure ambient relative humidity using DHT11.
- Monitor soil moisture using the HW-390 analog sensor.
- Monitor ambient light intensity using the LM393 module.
- Process and display sensor measurements in real time.

### 2. Automatic Control

The system automatically controls three external devices based on configurable environmental thresholds.

| Parameter | Control Condition | Controlled Device |
|---|---|---|
| Soil Moisture | Below configured threshold | Water Pump |
| Temperature | Above configured threshold | Fan |
| Light Intensity | Below configured threshold | Grow Light |

Hysteresis control is implemented to reduce frequent relay switching when sensor readings fluctuate around threshold values.

### 3. Manual Control

- Independently control the water pump, fan, and light.
- Switch between Automatic and Manual modes.
- Configure environmental thresholds through the button interface.
- Display current operating states on the TFT LCD.

### 4. TFT LCD User Interface

- 3.2-inch TFT LCD based on the ILI9341 controller.
- SPI2 communication interface.
- Real-time sensor data visualization.
- Operating mode and actuator status display.
- Button-based menu navigation and configuration.

### 5. RTC and Configuration Storage

- RTC operation using an external 32.768 kHz LSE crystal.
- Store system configuration in RTC Backup Registers.
- Retain backup-domain data using a CR1220 battery.
- Validate stored parameters using a configuration signature.

### 6. Pump Safety Protection

- Maximum automatic pump runtime: 60 seconds.
- Pump lockout period: 5 minutes.
- Prevent repeated automatic activation during lockout.
- Non-blocking time management using `HAL_GetTick()`.

### 7. UART Debugging

- USART1 serial communication.
- Baud rate: 115200 bps.
- Runtime status monitoring.
- Sensor measurement debugging.
- System event logging.

---

## 🔧 Hardware Components

| Component | Description | Interface |
|---|---|---|
| STM32F407VET6 | Main microcontroller | MCU |
| DHT11 | Temperature and humidity sensor | GPIO |
| HW-390 | Soil moisture sensor | ADC |
| LM393 | Light detection module | ADC + GPIO |
| ILI9341 | 3.2-inch TFT LCD | SPI2 |
| Relay Module | Pump, fan, and light control | GPIO |
| Push Buttons | User input interface | GPIO |
| RTC | Real-time clock | Internal RTC |
| CR1220 | RTC backup battery | Backup Domain |
| ST-LINK V2 | Programming and debugging | SWD |

---

## 📍 Hardware Pin Configuration

### Microcontroller

- MCU: STM32F407VET6
- Core: ARM Cortex-M4
- System Clock: 168 MHz
- Package: LQFP-100

### Sensor Interfaces

| Signal | STM32 Pin | Peripheral Configuration |
|---|---|---|
| Soil Moisture Analog | PA0 | ADC1_IN0 |
| Light Sensor Digital | PA2 | GPIO Input |
| Light Sensor Analog | PA3 | ADC1_IN3 |
| DHT11 Data | PD15 | GPIO Open-Drain / Input |

### Relay Outputs

| Device | STM32 Pin | GPIO Configuration |
|---|---|---|
| Water Pump | PD7 | Output Push-Pull |
| Fan | PD6 | Output Push-Pull |
| Grow Light | PD5 | Output Push-Pull |

**Initial output state:** All relay control outputs are configured to LOW (OFF), assuming an active-HIGH relay driver.

### TFT LCD – ILI9341 (SPI2)

| LCD Signal | STM32 Pin | Configuration |
|---|---|---|
| SCK | PB13 | SPI2_SCK |
| MISO | PB14 | SPI2_MISO |
| MOSI | PB15 | SPI2_MOSI |
| CS | PB12 | GPIO Output |
| DC | PD8 | GPIO Output |
| RESET | To be assigned | GPIO Output |
| BACKLIGHT | Module dependent | GPIO / Power |

### Push Buttons

| Button | STM32 Pin | Configuration |
|---|---|---|
| UP | PE2 | GPIO Input |
| DOWN | PE3 | GPIO Input |
| SET | PE4 | GPIO Input |
| NEXT | PE5 | GPIO Input |

### UART1

| Signal | STM32 Pin |
|---|---|
| TX | PA9 |
| RX | PA10 |
| Baud Rate | 115200 |

### Clock and Debug Interfaces

| Peripheral | Configuration |
|---|---|
| HSE | 8 MHz Crystal |
| LSE | 32.768 kHz Crystal |
| RTC Backup | CR1220 Battery |
| SWDIO | PA13 |
| SWCLK | PA14 |

---

## 🏗️ Firmware Architecture

The firmware follows a modular, layered architecture using STM32 HAL.

The application is designed without an RTOS. Instead, periodic tasks are scheduled using a non-blocking time-based mechanism.

### Software Layers

| Layer | Responsibilities |
|---|---|
| Application Layer | Main application logic, operating modes, and system state management |
| Service Layer | Sensor management, relay control, safety, configuration, and UI management |
| Driver Layer | Low-level sensor, display, button, and peripheral drivers |
| Hardware Layer | STM32F407VET6 and external hardware components |

### Firmware Modules

| Module | Description |
|---|---|
| `sensor_manager` | Collects and processes environmental sensor data |
| `control_manager` | Implements automatic and manual control logic |
| `relay_manager` | Controls pump, fan, and light relay outputs |
| `button_manager` | Handles button scanning and debouncing |
| `ui_manager` | Manages LCD rendering and menu navigation |
| `safety_manager` | Implements pump runtime and lockout protection |
| `config_manager` | Handles configuration storage and retrieval |
| `logger` | Provides UART-based debugging and event logging |

---

## ⚙️ Operating Modes

### Automatic Mode

In Automatic Mode, the controller continuously evaluates sensor measurements against configured thresholds.

| Environmental Condition | System Action |
|---|---|
| Soil moisture below threshold | Activate water pump |
| Temperature above threshold | Activate cooling fan |
| Light intensity below threshold | Activate grow light |

Hysteresis is applied to each control condition to improve system stability.

### Manual Mode

In Manual Mode, users directly control the three relay outputs through the physical push buttons.

Automatic control decisions are disabled while Manual Mode is active.

Pump safety protection remains enabled regardless of the selected operating mode.

---

## 🛡️ Pump Safety Mechanism

A state machine is used to prevent excessive pump operation.

### Operating Sequence

```text
              ┌──────────┐
              │   IDLE   │
              └────┬─────┘
                   │
          Soil moisture below
             configured limit
                   │
                   ▼
              ┌──────────┐
              │ PUMP ON  │
              └────┬─────┘
                   │
          Runtime >= 60 seconds
                   │
                   ▼
              ┌──────────┐
              │ LOCKOUT  │
              └────┬─────┘
                   │
           Lockout >= 5 minutes
                   │
                   ▼
              ┌──────────┐
              │   IDLE   │
              └──────────┘
```

The mechanism uses non-blocking time management based on `HAL_GetTick()` instead of blocking delays.

This allows the firmware to continue handling sensor acquisition, button inputs, display updates, and other system functions during pump operation and lockout periods.

---

## ⏱️ Task Scheduling

The application uses a cooperative, non-blocking scheduler based on the STM32 HAL system tick.

| Task | Execution Interval |
|---|---:|
| Button Scanning | 10 ms |
| ADC Sampling | 100 ms |
| Control Processing | 100 ms |
| Pump Safety Monitoring | 10 ms |
| DHT11 Acquisition | 2 seconds |
| LCD Refresh | 200 ms |
| UART Debug Logging | 1 second |

These intervals are initial design targets and can be adjusted during system integration and performance testing.

---

## 💻 Development Environment

| Tool / Technology | Purpose |
|---|---|
| STM32CubeMX | Peripheral configuration and code generation |
| STM32CubeIDE / Keil MDK | Firmware development and compilation |
| STM32 HAL | Hardware abstraction and peripheral drivers |
| Embedded C | Firmware implementation |
| ST-LINK V2 | Flash programming and debugging |
| STM32CubeProgrammer | Firmware flashing and device management |
| Serial Terminal | UART debugging and monitoring |

### Clock Configuration

| Clock Source | Frequency |
|---|---:|
| HSE | 8 MHz |
| SYSCLK | 168 MHz |
| LSE | 32.768 kHz |

The clock tree is configured according to the STM32F407 operating limits and peripheral clock requirements.

---

## 🚀 Getting Started

### 1. Hardware Setup

Connect the sensors, relay module, TFT LCD, push buttons, and debugging interface according to the hardware pin configuration.

Verify all power supply connections and signal voltage levels before powering on the system.

### 2. STM32CubeMX Configuration

1. Select `STM32F407VET6`.
2. Configure the system clock to 168 MHz.
3. Enable ADC1 and configure Channels 0 and 3.
4. Enable SPI2 for the ILI9341 display.
5. Enable USART1 at 115200 baud.
6. Enable RTC and configure the LSE clock source.
7. Configure GPIO pins according to the pin mapping.
8. Enable Serial Wire Debug (SWD).
9. Generate the STM32 HAL project.

### 3. Firmware Development

Implement and integrate the following modules:

- Sensor drivers.
- Display driver.
- Button driver.
- Sensor management.
- Automatic and manual control.
- Relay management.
- Pump safety state machine.
- RTC configuration storage.
- UART debugging.

### 4. Build and Flash

1. Open the generated project in STM32CubeIDE or Keil MDK.
2. Build the firmware.
3. Connect the ST-LINK V2 debugger.
4. Flash the firmware to STM32F407VET6.
5. Reset the microcontroller and verify system initialization.

### 5. Debugging

Connect a serial terminal to USART1 using the following settings:

```text
Baud Rate: 115200
Data Bits: 8
Parity: None
Stop Bits: 1
Flow Control: None
```

Monitor sensor measurements, operating modes, relay states, and safety events.

---

## 📂 Project Structure

```text
Smart_Garden/
│
├── Core/
│   ├── Inc/
│   └── Src/
│
├── Drivers/
│   ├── DHT11/
│   │   ├── dht11.c
│   │   └── dht11.h
│   │
│   ├── ILI9341/
│   │   ├── ili9341.c
│   │   └── ili9341.h
│   │
│   └── Buttons/
│       ├── button.c
│       └── button.h
│
├── Application/
│   ├── app.c
│   ├── sensor_manager.c
│   ├── control_manager.c
│   ├── relay_manager.c
│   ├── button_manager.c
│   ├── ui_manager.c
│   ├── safety_manager.c
│   └── config_manager.c
│
├── Debug/
│   ├── logger.c
│   └── logger.h
│
├── README.md
└── STM32F407VET6.ioc
```

*Note: The directory structure represents the proposed firmware organization and can be adjusted to match the actual implementation.*

---

## 🔮 Future Improvements

- Implement SD Card data logging.
- Add DMA-based ADC acquisition.
- Improve sensor filtering and measurement stability.
- Add sensor disconnection and fault detection.
- Integrate an Independent Watchdog (IWDG).
- Implement low-power operating modes.
- Add ESP8266 Wi-Fi connectivity.
- Implement MQTT communication for remote monitoring.
- Develop a web-based environmental monitoring dashboard.

---

## 🎯 Technical Skills Demonstrated

This project demonstrates practical embedded firmware development skills:

- Embedded C programming.
- STM32F4 microcontroller architecture.
- STM32CubeMX and STM32 HAL.
- GPIO configuration and control.
- ADC acquisition and sensor interfacing.
- SPI communication and TFT LCD driver development.
- UART communication and debugging.
- RTC and Backup Register management.
- State Machine implementation.
- Non-blocking firmware design.
- Hardware-software integration.
- Modular firmware architecture.

---

## 📋 Project Information

| Category | Details |
|---|---|
| Project Name | Smart Garden Controller |
| Microcontroller | STM32F407VET6 |
| CPU Architecture | ARM Cortex-M4 |
| Core Frequency | 168 MHz |
| Programming Language | Embedded C |
| Development Framework | STM32 HAL |
| Configuration Tool | STM32CubeMX |
| Operating System | Bare-metal (No RTOS) |
| Display Interface | SPI2 |
| Debug Interface | USART1 / SWD |
| Project Domain | Embedded Systems / Smart Agriculture |

---

