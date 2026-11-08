# 🌱 Smart Garden Controller & Environmental Monitoring System

STM32F407VET6 | ARM Cortex-M4 | Embedded C | STM32 HAL

An embedded smart garden system based on the **STM32F407VET6 microcontroller** that monitors environmental conditions and automatically controls irrigation, ventilation, and lighting.

The system integrates multiple sensors, relay-based actuators, an SPI TFT display, and a push-button interface. It supports both automatic and manual operating modes, with configurable environmental thresholds and built-in pump safety protection.

* * *

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

* * *

## 🌿 Project Overview

The Smart Garden Controller is designed to monitor and maintain suitable environmental conditions for plant growth.

The STM32F407VET6 collects data from environmental sensors, processes the measurements, and controls external devices according to predefined thresholds.

Sensor readings and system states are displayed on a 3.2-inch TFT LCD using the ILI9341 controller.

Users can interact with the system through four physical push buttons to configure operating parameters, switch operating modes, and manually control connected devices.

### System Architecture

```
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
    ILI9341  Push Buttons   Flash
     LCD      UP/DOWN      Settings
              SET/NEXT     Storage
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

* * *

## ✨ Features

### 1. Environmental Monitoring

- Measure ambient temperature using DHT11.
- Measure ambient relative humidity using DHT11.
- Monitor soil moisture using the HW-390 analog sensor.
- Monitor ambient light intensity using the LM393 module.
- Process and display sensor measurements in real time.

### 2. Automatic Control

The system automatically controls three external devices based on configurable environmental thresholds.

| Parameter       | Control Condition          | Controlled Device |
|-----------------|----------------------------|-------------------|
| Soil Moisture   | Below configured threshold | Water Pump        |
| Temperature     | Above configured threshold | Fan               |
| Light Intensity | Below configured threshold | Grow Light        |

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

### 5. Configuration Storage

- Store the threshold settings in the last Flash sector (sector 7) of the MCU.
- Settings are retained when power is removed.
- Flash is erased and written only when a changed setting is saved with the SET button.
- Validate stored parameters using a signature and checksum, falling back to defaults if invalid.

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

* * *

## 🔧 Hardware Components

| Component     | Description                     | Interface     |
|---------------|---------------------------------|---------------|
| STM32F407VET6 | Main microcontroller            | MCU           |
| DHT11         | Temperature and humidity sensor | GPIO          |
| HW-390        | Soil moisture sensor            | ADC           |
| LM393         | Light detection module          | ADC + GPIO    |
| ILI9341       | 3.2-inch TFT LCD                | SPI2          |
| Relay Module  | Pump, fan, and light control    | GPIO          |
| Push Buttons  | User input interface            | GPIO          |
| ST-LINK V2    | Programming and debugging       | SWD           |

* * *

## 📍 Hardware Pin Configuration

### Microcontroller

- MCU: STM32F407VET6
- Core: ARM Cortex-M4
- System Clock: 168 MHz
- Package: LQFP-100

### Sensor Interfaces

| Signal               | STM32 Pin | Peripheral Configuration                                   |
|----------------------|-----------|------------------------------------------------------------|
| Soil Moisture Analog | PA0       | ADC1_IN0                                                   |
| Light Sensor Digital | PA2       | GPIO Input                                                 |
| Light Sensor Analog  | PA3       | ADC1_IN3                                                   |
| DHT11 Data           | PD15      | GPIO Output Open-Drain (external 4.7 kΩ pull-up to 3.3 V)  |

The analog sensors (HW-390, LM393) must be powered from **3.3 V** to keep their outputs within the ADC input range.

### Relay Outputs

| Device     | STM32 Pin | GPIO Configuration |
|------------|-----------|--------------------|
| Water Pump | PD7       | Output Push-Pull   |
| Fan        | PD6       | Output Push-Pull   |
| Grow Light | PD5       | Output Push-Pull   |

**Initial output state:** All relay control outputs are configured to LOW (OFF), assuming an active-HIGH relay driver.

### TFT LCD – ILI9341 (SPI2)

| LCD Signal | STM32 Pin        | Configuration                       |
|------------|------------------|-------------------------------------|
| SCK        | PB13             | SPI2_SCK                            |
| MISO       | PB14             | SPI2_MISO                           |
| MOSI       | PB15             | SPI2_MOSI                           |
| CS         | PB12             | GPIO Output                         |
| DC         | PD8              | GPIO Output                         |
| RESET      | NRST             | Shared with the MCU reset line      |
| BACKLIGHT  | Module dependent | Powered from the module supply      |

### Push Buttons

| Button | STM32 Pin | Configuration                                   |
|--------|-----------|-------------------------------------------------|
| UP     | PE2       | GPIO Input, external 10 kΩ pull-up (active-low) |
| DOWN   | PE3       | GPIO Input, external 10 kΩ pull-up (active-low) |
| SET    | PE4       | GPIO Input, external 10 kΩ pull-up (active-low) |
| NEXT   | PE5       | GPIO Input, external 10 kΩ pull-up (active-low) |

### UART1

| Signal    | STM32 Pin |
|-----------|-----------|
| TX        | PA9       |
| RX        | PA10      |
| Baud Rate | 115200    |

### Clock and Debug Interfaces

| Peripheral | Configuration      |
|------------|--------------------|
| HSE        | 8 MHz Crystal      |
| SWDIO      | PA13               |
| SWCLK      | PA14               |

* * *

## 🏗️ Firmware Architecture

The firmware is written with STM32 HAL and organized as a small set of source modules.

The application is designed without an RTOS. Instead, periodic tasks are scheduled using a non-blocking time-based mechanism.

### Firmware Modules

| Module                | Description                                                                         |
|-----------------------|-------------------------------------------------------------------------------------|
| `app.c`               | Application logic: sensor processing, automatic/manual control, pump safety, button handling, UI pages, Flash configuration storage, UART logging |
| `app_config.h`        | Sensor calibration, default thresholds, safety times, task intervals                |
| `dht11.c`             | DHT11 driver (pulse timing with TIM6 at 1 µs)                                       |
| `ili9341.c`           | ILI9341 SPI driver with text and rectangle drawing                                  |
| `font5x7.c`           | 5x7 ASCII font                                                                      |

* * *

## ⚙️ Operating Modes

### Automatic Mode

In Automatic Mode, the controller continuously evaluates sensor measurements against configured thresholds.

| Environmental Condition         | System Action        |
|---------------------------------|----------------------|
| Soil moisture below threshold   | Activate water pump  |
| Temperature above threshold     | Activate cooling fan |
| Light intensity below threshold | Activate grow light  |

Hysteresis is applied to each control condition to improve system stability.

### Manual Mode

In Manual Mode, users directly control the three relay outputs through the physical push buttons.

Automatic control decisions are disabled while Manual Mode is active.

The pump safety protection applies in Automatic Mode only. In Manual Mode the user is responsible for switching the pump off.

* * *

## 🛡️ Pump Safety Mechanism

A timer-based protection prevents excessive pump operation in Automatic Mode.

### Operating Sequence

```
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

* * *

## ⏱️ Task Scheduling

The application uses a cooperative, non-blocking scheduler based on the STM32 HAL system tick.

| Task                              | Execution Interval |
|-----------------------------------|-------------------:|
| Button Scanning                   | 10 ms              |
| ADC Sampling                      | 200 ms             |
| Control Processing + Pump Safety  | 500 ms             |
| DHT11 Acquisition                 | 2 seconds          |
| LCD Refresh                       | 250 ms             |
| UART Debug Logging                | 2 seconds          |

The intervals are defined in `app_config.h` and can be adjusted.

* * *

## 💻 Development Environment

| Tool / Technology       | Purpose                                      |
|-------------------------|----------------------------------------------|
| STM32CubeMX             | Peripheral configuration and code generation |
| STM32CubeIDE / Keil MDK | Firmware development and compilation         |
| STM32 HAL               | Hardware abstraction and peripheral drivers  |
| Embedded C              | Firmware implementation                      |
| ST-LINK V2              | Flash programming and debugging              |
| STM32CubeProgrammer     | Firmware flashing and device management      |
| Serial Terminal         | UART debugging and monitoring                |

### Clock Configuration

| Clock Source | Frequency  |
|--------------|-----------:|
| HSE          | 8 MHz      |
| SYSCLK       | 168 MHz    |

The clock tree is configured according to the STM32F407 operating limits and peripheral clock requirements.

* * *

## 🚀 Getting Started

### 1. Hardware Setup

Connect the sensors, relay module, TFT LCD, push buttons, and debugging interface according to the hardware pin configuration.

Verify all power supply connections and signal voltage levels before powering on the system.

### 2. STM32CubeMX Configuration

1. Select `STM32F407VET6`.
2. Configure the system clock to 168 MHz.
3. Enable ADC1 and configure Channels 0 and 3 (scan mode, 480-cycle sampling, DMA2 Stream 0 in circular mode).
4. Enable SPI2 for the ILI9341 display.
5. Enable USART1 at 115200 baud.
6. Enable TIM6 with Prescaler = 83 and Period = 65535 (1 µs tick for the DHT11).
7. Configure GPIO pins according to the pin mapping.
8. Enable Serial Wire Debug (SWD).
9. Generate the STM32 HAL project.

### 3. Firmware Integration

Copy the application sources into `Core/Inc` and `Core/Src`, then call `App_Init()` once before the main loop and `App_Loop()` inside the main loop of `main.c`.

Adjust the sensor calibration values in `app_config.h` (`SOIL_RAW_DRY`, `SOIL_RAW_WET`, `LIGHT_INVERT`) after reading the raw values shown on the LCD.

### 4. Build and Flash

1. Open the generated project in STM32CubeIDE or Keil MDK.
2. Build the firmware.
3. Connect the ST-LINK V2 debugger.
4. Flash the firmware to STM32F407VET6.
5. Reset the microcontroller and verify system initialization.

### 5. Debugging

Connect a serial terminal to USART1 using the following settings:

```
Baud Rate: 115200
Data Bits: 8
Parity: None
Stop Bits: 1
Flow Control: None
```

Monitor sensor measurements, operating modes, relay states, and safety events.

* * *

## 📂 Project Structure

```
Smart_Garden/
│
├── Core/
│   ├── Inc/
│   │   ├── app.h
│   │   ├── app_config.h
│   │   ├── dht11.h
│   │   ├── ili9341.h
│   │   └── font5x7.h
│   └── Src/
│       ├── main.c
│       ├── app.c
│       ├── dht11.c
│       ├── ili9341.c
│       └── font5x7.c
│
├── Readme.md
└── STM32F407VET6.ioc
```

* * *

## 🔮 Future Improvements

- Improve sensor filtering and measurement stability.
- Add sensor disconnection and fault detection.
- Integrate an Independent Watchdog (IWDG).
- Implement low-power operating modes.
- Add ESP8266 Wi-Fi connectivity.
- Implement MQTT communication for remote monitoring.
- Develop a web-based environmental monitoring dashboard.

* * *

## 🎯 Technical Skills Demonstrated

This project demonstrates practical embedded firmware development skills:

- Embedded C programming.
- STM32F4 microcontroller architecture.
- STM32CubeMX and STM32 HAL.
- GPIO configuration and control.
- ADC acquisition with DMA and sensor interfacing.
- SPI communication and TFT LCD driver development.
- Timing-critical single-wire protocol (DHT11) using a hardware timer.
- UART communication and debugging.
- Flash memory programming for persistent settings.
- Non-blocking firmware design.
- Hardware-software integration.

* * *

## 📋 Project Information

| Category              | Details                              |
|-----------------------|--------------------------------------|
| Project Name          | Smart Garden Controller              |
| Microcontroller       | STM32F407VET6                        |
| CPU Architecture      | ARM Cortex-M4                        |
| Core Frequency        | 168 MHz                              |
| Programming Language  | Embedded C                           |
| Development Framework | STM32 HAL                            |
| Configuration Tool    | STM32CubeMX                          |
| Operating System      | Bare-metal (No RTOS)                 |
| Display Interface     | SPI2                                 |
| Debug Interface       | USART1 / SWD                         |
| Project Domain        | Embedded Systems / Smart Agriculture |
