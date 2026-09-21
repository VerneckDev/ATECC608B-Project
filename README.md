# ATECC608B Project

This project focuses on the integration, configuration, and testing of the **Microchip ATECC608B** secure element using an Arduino-based development setup.

The main objective was to establish reliable communication with the ATECC608B through the **I²C interface**, verify the hardware configuration, and test the functionality provided by the device using the available Microchip/Arduino libraries.

The project was developed as part of a cybersecurity/hardware security study, with particular focus on **hardware-based cryptographic security and secure key storage**.

---

## Project Overview

The **ATECC608B** is a secure cryptographic element from Microchip designed to provide hardware-based security functions such as cryptographic operations, secure key storage, authentication, and random number generation.

This project investigates the communication and operation of the device when connected to an Arduino through the I²C bus.

The main stages of the project were:

1. Hardware assembly and electrical connection.
2. Configuration of the ATECC608B.
3. Establishment of I²C communication.
4. Testing of the device using the available libraries and example programs.
5. Analysis of the I²C communication signals using an oscilloscope.
6. Verification of the pull-up resistor configuration.
7. Execution of cryptographic and security-related tests.

The ATECC608B is supported by Microchip's **CryptoAuthLib**, which provides APIs for communicating with Microchip CryptoAuthentication devices, including the ATECC608B.

---

## Hardware

### Main Components

* **ATECC608B-SSHDA-T**
* Arduino development board
* I²C communication interface
* Pull-up resistors
* Oscilloscope for signal analysis
* External power supply where required

The ATECC608B communicates with the microcontroller through the **I²C bus**.

### I²C Connection

The basic connection consists of:

```text
Arduino              ATECC608B

  SDA    ─────────►    SDA
  SCL    ─────────►    SCL
  VCC    ─────────►    VCC
  GND    ─────────►    GND
```

Pull-up resistors are required on the SDA and SCL lines because I²C uses open-drain/open-collector signalling.

In the initial setup, **10 kΩ pull-up resistors** were used.

---

## Hardware Verification

Before performing the software tests, the electrical connection between the Arduino and the ATECC608B was verified.

Particular attention was given to the I²C signal levels because incorrect pull-up resistance can affect the communication speed and signal integrity.

An oscilloscope was used to observe the SDA and SCL signals during communication with the ATECC608B.

The measured signals showed that the logic levels were sufficiently separated and that the **10 kΩ pull-up resistors were not the cause of the communication problem**.

This step was important because it allowed the hardware configuration to be separated from possible software or library-related problems.

---

## Software

The project uses an Arduino development environment together with libraries for communicating with the ATECC608B.

The Microchip **CryptoAuthLib** provides the underlying API for communicating with CryptoAuthentication devices and supports the ATECC608B.

Depending on the test, the project can use the Arduino-compatible interface and the corresponding cryptographic functions provided by the library.

### Main Software Components

The repository is organized into three main sections:

```text
ATECC608B-Project/
│
├── ATECC608B_config/
│   └── Configuration and setup tests
│
├── ATECC608B_test/
│   └── Initial ATECC608B communication and functionality tests
│
├── ATECC608B_test2/
│   └── Additional device tests
│
└── README.md
```

---

## ATECC608B Configuration

The first stage of the project consists of configuring and initializing the ATECC608B.

The device contains internal memory areas used for configuration, data, and cryptographic keys. The configuration of these areas determines how the device can be used and which operations are permitted.

The configuration stage was therefore treated separately from the general communication tests.

The `ATECC608B_config` directory contains the code associated with this stage.

---

## Communication Test

The `ATECC608B_test` directory contains the initial tests used to verify communication between the Arduino and the ATECC608B.

The general communication flow is:

```text
Arduino
   │
   │ I²C command
   ▼
ATECC608B
   │
   │ Response
   ▼
Arduino
```

The first objective is to confirm that the microcontroller can correctly detect and communicate with the secure element.

Once communication is established, additional device functions can be tested.

---

## Additional Tests

The `ATECC608B_test2` directory contains additional experiments performed after the initial communication tests.

These tests were used to investigate the behaviour of the device and its cryptographic functionality in more detail.

The exact operations depend on the individual test program contained in the directory.

---

## Pull-Up Resistor Analysis

One of the hardware investigations performed during the project was the analysis of the I²C pull-up resistors.

The initial configuration used:

```text
Rp = 10 kΩ
```

The resistor value was investigated because the pull-up resistance directly affects the rise time of the SDA and SCL signals.

The I²C bus was observed with an oscilloscope while commands were being sent to the ATECC608B.

The objective was to determine whether the logic levels were sufficiently separated and whether the communication signals were being correctly generated.

The measurements showed that the logic levels were clearly distinguishable.

Therefore:

> The 10 kΩ pull-up resistance was considered adequate for the tested configuration and was not identified as the source of the communication problem.

---

## Development Process

The project followed a progressive testing methodology:

```text
Hardware assembly
        │
        ▼
I²C connection verification
        │
        ▼
ATECC608B detection
        │
        ▼
Configuration
        │
        ▼
Basic communication tests
        │
        ▼
Cryptographic functionality
        │
        ▼
Signal analysis and debugging
        │
        ▼
Final verification
```

This approach made it possible to distinguish hardware-related problems from software and configuration problems.

---

## Troubleshooting

Several aspects were considered during the debugging process.

### 1. Hardware Connections

The first step was to verify:

* Power supply
* Ground connection
* SDA connection
* SCL connection
* Pull-up resistors

Incorrect wiring can prevent the ATECC608B from responding correctly.

### 2. Pull-Up Resistance

The initial pull-up value was **10 kΩ**.

The SDA and SCL signals were examined with an oscilloscope to determine whether the voltage transitions were sufficiently clear.

The measurements indicated that the logic levels were adequately separated.

### 3. Library Configuration

The communication library and its configuration were also investigated.

The project uses libraries compatible with Microchip CryptoAuthentication devices. CryptoAuthLib officially supports the ATECC608B and provides APIs for communicating with the device.

### 4. Example Programs

The available example programs were used to verify that the hardware and library configuration were functioning correctly before implementing additional tests.

---

## Results

The project successfully established communication with the ATECC608B and allowed the device to be investigated through an Arduino-based setup.

The hardware investigation also demonstrated that the I²C signal quality was adequate with the selected pull-up configuration.

The oscilloscope measurements showed clearly separated logic levels, allowing the pull-up resistance to be ruled out as the main cause of the communication issue investigated during development.

The project also provided practical experience with:

* Secure elements
* Hardware-based cryptography
* I²C communication
* Cryptographic key storage
* Microcontroller-to-secure-element communication
* Hardware debugging
* Oscilloscope-based signal analysis
* CryptoAuthentication libraries

---

## Repository Structure

```text
ATECC608B-Project/
│
├── ATECC608B_config/
│   └── ATECC608B configuration
│
├── ATECC608B_test/
│   └── Initial device tests
│
├── ATECC608B_test2/
│   └── Sending files tests
│
└── README.md
```

---

## Requirements

### Hardware

* Arduino-compatible development board
* ATECC608B-SSHDA-T
* I²C pull-up resistors
* Breadboard and jumper wires
* USB connection
* Oscilloscope (recommended for hardware debugging)

### Software

* Arduino IDE
* Appropriate Arduino board package
* ATECC608B-compatible library
* Microchip CryptoAuthLib / Arduino cryptographic library

CryptoAuthLib supports the ATECC608B and other Microchip CryptoAuthentication devices.

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/VerneckDev/ATECC608B-Project.git
cd ATECC608B-Project
```

### 2. Connect the hardware

Connect the ATECC608B to the Arduino through the I²C interface.

Verify the SDA and SCL connections and make sure the I²C bus has the required pull-up resistors.

### 3. Install the required library

Install the library required by the selected test program through the Arduino IDE.

For projects using CryptoAuthLib, the library provides the APIs required to communicate with the ATECC608B.

### 4. Open the desired test

Depending on the objective, open the corresponding project:

```text
ATECC608B_config/
ATECC608B_test/
ATECC608B_test2/
```

### 5. Compile and upload

Select the appropriate Arduino board and serial port, then compile and upload the program.

### 6. Monitor the results

Open the Arduino Serial Monitor and observe the output generated by the test program.

---

## References

* Microchip Technology, **CryptoAuthLib – Microchip CryptoAuthentication Library**. The library provides APIs for communication with Microchip secure elements, including the ATECC608B.
* Microchip Technology, **ATECC608B CryptoAuthentication Secure Element**.
* Arduino, **Arduino IDE and Arduino libraries**.
* ATECC608B device documentation and application material.

---

## Author

**João Pedro Verneck**

University of Coimbra
Department of Physics
2025/2026

---

## License

This project is intended for educational and research purposes.

See the repository for the applicable license and project information.
