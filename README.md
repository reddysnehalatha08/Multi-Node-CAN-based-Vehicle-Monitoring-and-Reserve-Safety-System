# CAN-Based Vehicle Monitoring and Reverse Safety System

![Protocol](https://img.shields.io/badge/Protocol-CAN-blue)
![Microcontroller](https://img.shields.io/badge/MCU-LPC2129-orange)
![Language](https://img.shields.io/badge/Language-Embedded%20C-red)
![IDE](https://img.shields.io/badge/IDE-Keil%20µVision-brightgreen)

A three-node embedded systems prototype demonstrating vehicle-status monitoring and driver alerts using the Controller Area Network (CAN) protocol. The system combines engine-temperature measurement, interrupt-driven indicator requests, forward/reverse mode selection, and ultrasonic reverse-obstacle detection.

> **Project scope:** This is an educational prototype, not production automotive safety equipment. It has not been represented as automotive-certified or safety-validated software.

## Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Operating Sequence](#operating-sequence)
- [CAN Message Flow](#can-message-flow)
- [Expected Demonstration](#expected-demonstration)
- [Node Responsibilities](#node-responsibilities)
- [Hardware and Software](#hardware-and-software)
- [Pin Connections](#pin-connections)
- [Sensor Operation](#sensor-operation)
- [CAN Communication](#can-communication)
- [Build and Flash](#build-and-flash)
- [Hardware Implementation and Output Evidence](#hardware-implementation-and-output-evidence)
- [Testing Checklist](#testing-checklist)
- [Limitations and Safety Notes](#limitations-and-safety-notes)
- [Future Improvements](#future-improvements)
@@ -406,41 +407,95 @@

Do not describe the firmware as build-verified until all three images have been compiled successfully with the actual toolchain.

## Hardware Implementation and Output Evidence

This section contains photographs of the physical prototype. The diagrams earlier in this README explain the architecture and intended data flow; the photographs below document the assembled boards and LCD states observed during the demonstration. They are complementary forms of documentation: diagrams explain the design, while photographs show the actual setup.

> **Evidence note:** These photographs are qualitative evidence of the displayed states and physical setup. They do not, by themselves, prove sensor accuracy, CAN error-free operation, response time, or automotive safety. Use measured test results for those claims.

### 1. LCD startup screen

The LCD displays the project title during startup.

![LCD startup screen showing the project title](images/output-title-screen.jpeg)

### 2. Complete hardware setup

The prototype uses multiple LPC2129 development boards connected with jumper wires. The main-node LCD and the ultrasonic sensor are visible in the assembled setup.

![Complete hardware prototype](images/output-hardware-setup.jpeg)

### 3. Forward driving mode

The LCD shows the temperature reading, indicator status, and forward-mode state in this captured demonstration.

![Forward driving mode displayed on the LCD](images/output-forward-mode.jpeg)

### 4. Reverse mode and obstacle alert

The LCD shows reverse mode and a distance/alert status in this captured demonstration. The exact threshold behavior should be documented using separate measurements at known distances.

![Reverse mode and distance alert output](images/output-reverse-alert.jpeg)

### 5. Left-indicator demonstration

This photograph records the hardware while the left-indicator input/state is being demonstrated.

![Left-indicator demonstration](images/output-left-indicator.jpeg)

### 6. Right-indicator demonstration

This photograph records the hardware while the right-indicator input/state is being demonstrated.

![Right-indicator demonstration](images/output-right-indicator.jpeg)

### Output observations

Based on the captured LCD screens and prototype photographs:

- The LCD can display the project title and operating information.
- Temperature values are shown in degrees Celsius; the photographs include readings around 38–40 °C.
- The display shows forward and reverse operating states.
- Indicator inputs/states are demonstrated on the assembled hardware.
- Reverse-mode output includes a distance or alert status from the ultrasonic sensing path.

These are visual observations, not calibrated measurements. Before publishing quantitative performance claims, record known target distances, displayed distances, CAN message reception, alert activation/deactivation points, and any test failures in a results table.

## Testing Checklist

- [ ] LCD initialization and character/string display verified
- [ ] DS18B20 presence detection and temperature reading verified against a known reference
- [ ] Mode switch toggles between forward and reverse
- [ ] Left and right switch interrupts trigger the expected CAN commands
- [ ] Indicator Node receives `0x101` and sequences the LEDs in the correct direction
- [ ] Reverse Node receives `0x102` and measures distance only in reverse mode
- [ ] Test distances on both sides of the 20 cm threshold
- [ ] Main Node receives `0x201` and activates/deactivates the warning correctly
- [ ] CAN communication checked for wiring, termination, common ground, bit rate, and error conditions
- [ ] Invalid/missing ultrasonic Echo handled without a false close-obstacle indication
- [ ] System tested after power cycling and with disconnected-sensor conditions

Add real photographs, LCD output images, measured test results, and a short demonstration video only after capturing them from the working hardware. Do not use placeholder images as proof of completed tests.

## Limitations and Safety Notes

- This is an educational prototype and must not be used as the sole safety system in a real vehicle.
- A single fixed distance threshold does not account for vehicle speed, braking distance, sensor blind zones, or environmental conditions.
- Ultrasonic readings can be unreliable on angled, soft, narrow, or irregular obstacles.
- Add handling for sensor failures, stale CAN data, CAN bus-off/error states, switch bounce, and communication timeouts before making stronger reliability claims.
- Verify all pin mappings, voltage levels, pull-ups, transceiver wiring, and CAN termination against the actual hardware and datasheets.
- The code's use of byte-sized payload fields limits the directly transmitted distance range. Define explicit scaling and validity/error codes if the protocol is extended.

## Future Improvements

- Add CAN timeout and error-state handling, plus a safe response when sensor data becomes stale.
- Add sensor-fault detection and explicit invalid-measurement status.
- Improve switch debounce and indicator state handling.
- Add temperature threshold warnings and, if required, a documented CAN temperature message.
- Create a CAN database/message specification and automated host-side test tool.
- Add system-level testing, timing measurements, and a hardware wiring diagram.
- Explore data logging or IoT monitoring as a separate extension.

## Conclusion

This project demonstrates modular Embedded C development on the LPC2129 and practical integration of CAN communication, digital temperature sensing, external interrupts, LCD output, LED control, and ultrasonic distance measurement. Its three-node structure makes it useful for learning how distributed embedded controllers exchange commands and sensor status over a shared communication bus.
