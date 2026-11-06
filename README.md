# Multi-Node CAN-Based Vehicle Monitoring and Reverse Safety System

> An educational embedded-systems prototype that demonstrates communication between multiple ARM7-based nodes over a Controller Area Network (CAN) bus. The system combines vehicle-status monitoring, indicator control and reverse obstacle detection.

**Project platform:** ARM7 LPC2129 • Embedded C • CAN  
**Sensors:** DS18B20 temperature sensor • HC-SR05-compatible ultrasonic sensor

> **Important:** This README is a project-ready draft based on the project details currently available and the linked reference repository. Verify every implementation-specific value (GPIO pins, CAN IDs, baud rate, threshold, payload format and exact component model) against your own source code and hardware before publishing. Do not claim tests or results that you have not performed.

---

## Contents

- [Project Overview](#project-overview)
- [Objectives](#objectives)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Node Responsibilities](#node-responsibilities)
- [Hardware Requirements](#hardware-requirements)
- [Software Requirements](#software-requirements)
- [Working Principle](#working-principle)
- [CAN Communication](#can-communication)
- [Sensor Details](#sensor-details)
- [Hardware Connections](#hardware-connections)
- [Build and Flash](#build-and-flash)
- [Testing Checklist](#testing-checklist)
- [Limitations](#limitations)
- [Future Enhancements](#future-enhancements)

## Project Overview

The project demonstrates how independent microcontroller nodes can exchange information using the CAN protocol. Each node performs a dedicated task and communicates with the other nodes through a shared CAN network.

The documented concept contains three functional nodes:

1. **Main Node:** Monitors temperature using a DS18B20 sensor, handles vehicle control inputs and processes received reverse-alert information.
2. **Indicator Node:** Receives indicator commands over CAN and controls the indicator LEDs.
3. **Reverse Alert Node:** Uses an ultrasonic sensor to measure the distance to an obstacle and reports the reverse-alert status over CAN.

The CAN bus allows the nodes to exchange messages without requiring a separate point-to-point communication link between every pair of controllers.

## Objectives

- Understand multi-node communication using the CAN protocol.
- Interface digital sensors with an ARM7 microcontroller.
- Monitor temperature and display relevant status information.
- Control indicator outputs using CAN commands.
- Detect obstacles during reverse operation.
- Generate visual and audible warnings based on received status.

## Key Features

- Multi-node embedded system architecture.
- CAN-based communication between independent controllers.
- Temperature monitoring using DS18B20.
- Ultrasonic distance measurement for reverse obstacle detection.
- Indicator control through CAN messages.
- LED and buzzer-based warning indication, where implemented.
- Modular Embedded C firmware.

## System Architecture

The following diagram illustrates the functional architecture. Confirm that the node allocation matches your actual implementation.

![System Architecture](images/system_architecture.png)

### CAN physical bus

Each microcontroller's CAN controller connects to the physical bus through a CAN transceiver. The transceivers connect to the differential CANH and CANL lines.

![CAN Bus Connection](images/can_bus_connection.png)

**Important wiring points:**

- Connect CANH to CANH and CANL to CANL across all nodes.
- Provide a common ground reference between the nodes.
- Use termination resistors at the two physical ends of the bus, according to the transceiver and network design.
- Do not add a 120-ohm termination resistor at every node.
- Check supply voltage and logic-level compatibility before connecting hardware.

## Node Responsibilities

### 1. Main Node

**Purpose:** Central monitoring and command node.

Typical responsibilities:

- Initialize the microcontroller peripherals and CAN controller.
- Read temperature data from the DS18B20 sensor.
- Display temperature or system status on the LCD, if included in the hardware.
- Read the left-indicator, right-indicator and forward/reverse mode inputs, if implemented.
- Transmit the relevant control commands through CAN.
- Receive reverse-alert information from the Reverse Alert Node.
- Activate the warning output when the received status requires it.

### 2. Indicator Node

**Purpose:** Control the vehicle indicator LEDs.

Typical responsibilities:

- Initialize the CAN controller and LED GPIO pins.
- Receive CAN frames from the Main Node.
- Check the message identifier.
- Decode the indicator command.
- Run the left or right LED sequence, or switch the indicators off.

### 3. Reverse Alert Node

**Purpose:** Detect obstacles while the vehicle is in reverse mode.

Typical responsibilities:

- Initialize the ultrasonic sensor interface and CAN controller.
- Receive the vehicle-mode information.
- Perform distance measurement when reverse mode is active.
- Calculate the distance from the Echo pulse duration.
- Compare the measured distance with the configured threshold.
- Send distance and alert information to the Main Node over CAN.

### Reverse-node firmware flow

![Reverse Node Flowchart](images/reverse_node_flow.png)

## Hardware Requirements

| Component | Purpose |
|---|---|
| LPC2129 development boards | Main, Indicator and Reverse Alert nodes |
| CAN transceiver modules | Physical CANH/CANL interface |
| DS18B20 digital temperature sensor | Temperature monitoring |
| HC-SR05-compatible ultrasonic module | Reverse obstacle detection |
| LEDs | Indicator and warning indication |
| Buzzer | Audible warning, if implemented |
| LCD | Display, if implemented |
| Push buttons / switches | Indicator and mode inputs, if implemented |
| CAN wiring | Interconnection of the nodes |
| Power supply | Power for the controllers and modules |

Update the quantities and exact module names to match the hardware you used.

## Software Requirements

- Embedded C
- Keil µVision
- ARM7 LPC2129 device support and startup files
- Flash Magic, if used for programming
- CAN, GPIO, timer and sensor driver source files

## Working Principle

### Step 1 — System initialization

Each node initializes its required GPIO pins, peripherals, sensor interfaces and CAN controller.

### Step 2 — Temperature monitoring

The Main Node communicates with the DS18B20 using the 1-Wire protocol. The sensor provides a digital temperature reading. The firmware processes the reading and displays or uses it according to the implemented application.

### Step 3 — Indicator operation

When an indicator input is activated, the Main Node sends an indicator command through CAN. The Indicator Node receives the frame, identifies the command and controls the corresponding LEDs.

### Step 4 — Reverse mode

When reverse mode is selected, the Reverse Alert Node performs ultrasonic distance measurement. The sensor emits an ultrasonic pulse and receives its reflection from an obstacle.

### Step 5 — Obstacle detection

The microcontroller measures the Echo pulse duration and calculates the approximate distance:

```text
Distance (cm) ≈ Echo time (microseconds) / 58
```

This is a common approximation for HC-SR04-style modules. Confirm the formula and timer calibration for the exact sensor and driver used.

### Step 6 — Alert communication

The Reverse Alert Node sends the measured distance and alert status to the Main Node through CAN. The Main Node processes the received information and activates the warning output if required by the application logic.

## CAN Communication

CAN is a serial communication protocol designed for reliable communication between multiple electronic control units. Nodes share the same bus, and messages are identified by message identifiers rather than by individual destination addresses.

### Message-flow concept

![CAN Message Flow](images/can_message_flow.png)

### Example application message table

The following identifiers and payload meanings are shown in the linked reference design. **Use them only if they match your actual CAN source code.**

| Example CAN ID | Sender → Receiver | Example payload | Purpose |
|---|---|---|---|
| `0x101` | Main → Indicator | `L`, `R` or `S` | Left, right or stop indicator command |
| `0x102` | Main → Reverse Alert | `0` or `1` | Forward / reverse mode |
| `0x201` | Reverse Alert → Main | Distance and alert flag | Reverse obstacle status |

The CAN bit rate must be identical across all nodes. Check the actual CAN configuration header or initialization function to document the correct bit rate.

### Why CAN is used

- Multiple nodes can communicate over a shared bus.
- Message identifiers support message prioritization.
- CAN includes error detection mechanisms.
- It reduces the amount of point-to-point wiring required.
- It is widely used in automotive and industrial embedded systems.

## Sensor Details

### DS18B20 — Digital Temperature Sensor

- Communicates using the 1-Wire protocol.
- Uses a single data line for communication.
- Provides digital temperature readings.
- Does not require an external ADC for temperature conversion.
- Requires correct power, ground and data-line pull-up connections.

### Ultrasonic Sensor — Reverse Obstacle Detection

- Sends an ultrasonic burst when triggered.
- Receives the reflected signal from an obstacle.
- The Echo pulse duration represents the round-trip travel time.
- The microcontroller calculates the approximate distance using the measured pulse duration.
- The application compares the distance with its configured alert threshold.

Check the ultrasonic module's Echo output voltage. If it exceeds the LPC2129 input voltage limits, use suitable level shifting.

## Hardware Connections

Document your actual wiring in the following tables before publishing. The example pin numbers in another repository should not automatically be treated as your board's wiring.

| Node | Module / Signal | LPC2129 GPIO or peripheral pin | Direction |
|---|---|---|---|
| Main | DS18B20 data | Add actual pin | Bidirectional |
| Main | CAN TX / CAN RX | Add verified CAN pins | TX / RX |
| Main | LCD signals | Add actual pins, if used | Output |
| Main | Switch inputs | Add actual pins, if used | Input |
| Main | Buzzer / warning LED | Add actual pins, if used | Output |
| Indicator | LED outputs | Add actual pins | Output |
| Indicator | CAN TX / CAN RX | Add verified CAN pins | TX / RX |
| Reverse Alert | Ultrasonic Trigger | Add actual pin | Output |
| Reverse Alert | Ultrasonic Echo | Add actual pin | Input |
| Reverse Alert | CAN TX / CAN RX | Add verified CAN pins | TX / RX |

## Build and Flash

1. Open Keil µVision.
2. Create or open a project for the exact LPC2129 device.
3. Add the source files required by the selected node.
4. Add the common CAN and peripheral driver files.
5. Verify startup code, clock configuration, linker settings and pin multiplexing.
6. Compile the project and resolve errors and warnings.
7. Generate the firmware output file.
8. Flash the correct firmware image to its corresponding controller using the supported programming method.
9. Repeat the process for each node.
10. Connect the CAN bus and test the nodes together.

Build each node as a separate firmware image if the implementation uses separate controllers for the three nodes.

## Testing Checklist

Use this checklist during hardware validation. Mark a test as passed only after observing the actual result.

| Test | Expected observation | Status |
|---|---|---|
| Main Node initialization | Controller and configured peripherals initialize | Not yet documented |
| Temperature sensing | DS18B20 reading is obtained correctly | Not yet documented |
| CAN communication | Nodes exchange valid CAN frames | Not yet documented |
| Left indicator | Correct left-indicator output is activated | Not yet documented |
| Right indicator | Correct right-indicator output is activated | Not yet documented |
| Reverse mode | Reverse mode is recognized by the relevant node | Not yet documented |
| Obstacle detection | Distance changes when obstacle position changes | Not yet documented |
| Warning output | Warning activates according to the configured threshold | Not yet documented |

Add real photographs of the assembled hardware, wiring and output display to provide evidence of implementation.

## Limitations

- This is an educational prototype, not a production automotive safety system.
- Sensor accuracy depends on the module, environment and calibration.
- CAN communication depends on correct bit timing, wiring, grounding and termination.
- A single ultrasonic sensor may not detect every obstacle or surface reliably.
- The system must not be used as a replacement for certified vehicle safety equipment.

## Future Enhancements

- Add additional sensor nodes to the CAN network.
- Add fault detection and communication timeout handling.
- Add data logging for temperature and obstacle measurements.
- Improve the display interface.
- Add more robust filtering and validation of sensor readings.
- Develop a detailed hardware schematic and test report.

---

## Repository Structure

Suggested structure (rename folders and files to match your actual repository):

```text
Multi-Node-CAN-Based-Vehicle-Monitoring/
│
├── README.md
│
├── images/
│   ├── system_architecture.png
│   ├── can_bus_connection.png
│   ├── reverse_node_flow.png
│   └── can_message_flow.png
│
├── Main_Node/
│   └── [main node source files]
│
├── Indicator_Node/
│   └── [indicator node source files]
│
├── Reverse_Alert_Node/
│   └── [reverse node source files]
│
└── Common/
    └── [shared CAN and peripheral drivers]
```

## Author

**Daggupati Reddy Snehalatha**  
B.Tech — Electronics and Communication Engineering

---

If you use this project for academic or portfolio purposes, make sure the source code, diagrams, pin connections and test evidence all describe the same actual implementation.
