# Projects Portfolio
## Project Setups
- ### Embedded
    - **Zephyr / nRF Connect**:
        - Getting Started: [royyandzakiy/zephyr-getting-started](https://github.com/royyandzakiy/zephyr-getting-started)
        - Devcontainer: TBD
        - Workspace: [royyandzakiy/balancer-robot-app](https://github.com/royyandzakiy/balancer-robot-app)
        - Modular Structure:
            - [royyandzakiy/ncs-app-event-manager](https://github.com/royyandzakiy/ncs-app-event-manager)
            - [royyandzakiy/zephyr-smf-hsm](https://github.com/royyandzakiy/zephyr-smf-hsm)
        - Custom Board: TBD
        - Testing: [royyandzakiy/zephyr-ztest-gpio-emul](https://github.com/royyandzakiy/zephyr-ztest-gpio-emul)
        - Debugging: [royyandzakiy/zephyr-threads-systemview](https://github.com/royyandzakiy/zephyr-threads-systemview)
    - **ESP-IDF**: 
        - Getting Started: [royyandzakiy/esp-idf-bare-minimum](https://github.com/royyandzakiy/esp-idf-bare-minimum)
        - Boilerplate: [royyandzakiy/espidf-boilerplate](https://github.com/royyandzakiy/espidf-boilerplate)
        - Modular Structure: [royyandzakiy/idf-component-cmake](https://github.com/royyandzakiy/idf-component-cmake)
        - Important Feature:
            - [royyandzakiy/mqtt-tinygsm-wifi](https://github.com/royyandzakiy/mqtt-tinygsm-wifi)
        - Testing: [royyandzakiy/esp-idf-hello-pytest](https://github.com/royyandzakiy/esp-idf-hello-pytest)
        - Debugging: [royyandzakiy/memfault-bare-minimum](https://github.com/royyandzakiy/memfault-bare-minimum)
    - **PlatformIO**
        - Testing:
            - **gtest**: [royyandzakiy/unittesting-espidf-pio-gtest](https://github.com/royyandzakiy/unittesting-espidf-pio-gtest)
            - **unity**: [royyandzakiy/unittesting-arduino-pio-unity](https://github.com/royyandzakiy/unittesting-arduino-pio-unity)
- ### Desktop
    - Getting Started:
        - **CMake**: Modern C++ template with CMake for builds, vcpkg for dependency management, pre-configured unit tests, and GitHub Actions CI included. Also features clang-format, clang-tidy, optional sanitizers (ASan, UBSan), CMake presets, and Tracy profiling support. Ready for professional C++ development
            - [royyandzakiy/cpp-project-template](https://github.com/royyandzakiy/cpp-project-template) 
        - **MSBuild**: [royyandzakiy/winrt-projection-console-boilerplate](https://github.com/royyandzakiy/winrt-projection-console-boilerplate)
    - Modular Structure:
        - [royyandzakiy/boost-sml-fsm](https://github.com/royyandzakiy/boost-sml-fsm)
    - Testing:
        - [royyandzakiy/gmock-sfinae-concepts-calculator](https://github.com/royyandzakiy/gmock-sfinae-concepts-calculator)

## Work
- ### LMesh
    - **ADS Device** Firmware: MCU to manage sensor readings from ADS sensors (8-24-32 channels) for Brain Computer Interface, sending data with high throughput and communicating via either BLE, USB Serial. Creating Developed using Nordic Connect SDK (NCS) / Zephyr for Nordic, with full testing suite (Unit Tests, System Tests, CI).
    - **ADS Station** DLL (Library) & Windows Console Application: Windows Native library (C++) to enable a Windows machine to interact with ADS Devices. Engaging with high datarate communications via BLE. Developing mainly via Visual Studio, utilizing Windows SDK, Windows Drivers, Windows low level APIs, Windows Runtime for C++ (WinRT C++). Implements full testing suite (WIP), robust build system (CMake, MSBuild, vcpkg) auto generated Documentation (Doxygen).
    - **ADS Device Tester** Windows Desktop App: App built on top of the ADS Station DLL to give a visual experience when interacting with ADS Devices. Developed based on .NET (C#).
    - **Door Lock Device** Firmware: MCU to manage handling door lock for prisons, ensuring robustness amidst potential fire warning. Handling heavy duty doors with complex railing power protocols. Recreating from legacy PIC platform, adding BLE capability to ensure able to perform FOTA, secured using Secure Boot with Signed firmware.
    - **Door Handle Sensor** Firmware: Controlling an auto door locking mechanism based on proximity sensing.
- ### eFishery
    - **eFishery Smart Feeder** Firmware: Smart feeder for shrimps and fishes. Developed with ESP32 / esp-idf. Manages inputs via BLE App and button. Controls motors to deliver feed to live stock.
    - **Fish Sensor** Firmware: Device that captures movement of pond water ripples using IMU. Developed with nRF52 for low power gains. Inferences data using TinyML to conclude fish behavior.
    - **Water Quality Sensor** Firmware: Device that captures various water qualities. Sends data to BLE App.
    - **Google Action Script Automation**: Developing clasp based system to automate Project & People Performance documents to fulfill ISO9001.
- ### ITB
    - **Mantis GCS**: Ground Control Station application for Mantis VTOL Drone. It is based on the QGroundControl project.
        - [royyandzakiy/mantis-gcs](https://github.com/royyandzakiy/mantis-gcs)
    - **Mavlink Waypoint Generator**: Helps generating waypoints for complex shapes with tunable parameters.
        - [royyandzakiy/mavlink-waypoint-generator](https://github.com/royyandzakiy/mavlink-waypoint-generator)
    - **LoRa RHMesh**: Explores the RHMesh library, elaborating details on how to use on an rfm95.
        - [royyandzakiy/LoRa-RHMesh](https://github.com/royyandzakiy/LoRa-RHMesh)

## Explorations
- **GUI**
    - [royyandzakiy/qt-logsfilter-gui](https://github.com/royyandzakiy/qt-logsfilter-gui)
- **Emscripten**
    - [royyandzakiy/emscripten-projects](https://github.com/royyandzakiy/emscripten-projects)
- **Backend**
    - [royyandzakiy/drogon-visitors-log](https://github.com/royyandzakiy/drogon-visitors-log)
    - [royyandzakiy/crow-modules-backend](https://github.com/royyandzakiy/crow-modules-backend)
- **C++ Standard Library**
    - [royyandzakiy/cpp-concurrency](https://github.com/royyandzakiy/cpp-concurrency)
    - [royyandzakiy/cpp-sensor-log-algo-ranges](https://github.com/royyandzakiy/cpp-sensor-log-algo-ranges)
- **Utilities**
    - [royyandzakiy/cmake-spdlog-logger](https://github.com/royyandzakiy/cmake-spdlog-logger)
- **Google Action Script**
    - [royyandzakiy/clasp-gdocs-gsheets](https://github.com/royyandzakiy/clasp-gdocs-gsheets)

## To Add
- Tracy C++
- zephyr project boilerplate: etl, devcont, aem
- zephyr cicd, jenkins, docs, rls pg
- zephyr sysbuild, secure boot
- C++ library project template
- zephyr ble dfu mtu
- embedded linux: qemu basic, flash script, custom driver

## Portfolio
Per project, answer:
1. What did you build?
2. What scale/complexity did it handle?
3. What measurable impact did it create?

For LMesh - ADS Device Firmware

Current: "sending data with high throughput"
Better:

· "Sustained X Mbps over BLE/USB" (test this with iperf-style tool)
· "Processed X channels at Y Hz" (e.g., "32 channels at 250Hz = 8,000 samples/sec")
· "Achieved <Xms latency from sensor to host"
· "Maintained 0% packet loss over Z meters"

For Door Lock Device (Prisons)

Current: "ensuring robustness amidst potential fire warning"
Better:

· "<100ms fail-safe engagement on fire signal"
· "Zero critical failures across X units/Y months"
· "Reduced power consumption by X% vs legacy PIC platform" (measure with nRF Power Profiler)
· "100% FOTA success rate across X updates"

For eFishery Smart Feeder

Current: "Manages inputs via BLE App and button"
Better:

· "Deployed across X ponds / Y units"
· "Reduced feed waste by X% through precise motor control"
· "Achieved X months on single battery charge" (critical for shrimp farmers)
· "99.X% uptime in humid, outdoor conditions"

For Fish Sensor (TinyML)

Current: "Infers data using TinyML to conclude fish behavior"
Better:

· "X% inference accuracy for feeding detection"
· "Model runs in <X KB RAM / <X ms"
· "Reduced manual monitoring by X hours/day for farmers"
· "Achieved X months battery life on nRF52"

For Mantis GCS

Current: "Ground Control Station application"
Better:

· "Used in X successful flights"
· "Handles waypoint missions up to X km"
· "Reduced mission planning time by X% via complex shape generator"

📊 Quick Fixes You Can Apply Today

Project Add This Metric (Easily Measurable)
ADS Device Max channels × sample rate (e.g., "32ch @ 1kHz")
Door Lock Fail-safe trigger time + deployment count
Smart Feeder Battery life + number of units deployed
Fish Sensor Model size + inference time + accuracy %
Water Sensor Battery life + measurement frequency

🎯 Structural Improvements

```markdown
### ADS Device Firmware
**Brain-Computer Interface Data Acquisition** | *Nordic NCS / Zephyr*

Built high-throughput firmware streaming 32-channel EEG data at 250Hz (8K samples/sec) over BLE/USB. Achieved <10ms latency with 0% packet loss over 5m.

**↓ 8,000 samples/sec · 0% loss**

- Implemented BLE throughput optimization (MTU sizing, connection params) achieving 70% of theoretical max
- Built CI pipeline with unit tests + hardware-in-loop system tests
- Reduced power consumption 40% vs initial prototype through task scheduling optimization
```

🛠️ How to Get Missing Metrics Fast

1. Deployment scale: Check sales records, ask PMs, or estimate from production batches
2. Performance numbers: Run quick benchmarks on existing hardware
3. Reliability: Search Jira/GitHub for "bug count" or "uptime" in production logs
4. Efficiency gains: Compare old vs new power measurements, memory usage, or build times

⚡ One Low-Effort Win

Add a "Selected Metrics" badge to each work project:

```
[8K samples/sec]  [40% power reduction]  [0% packet loss]
```

This visually signals impact before they read the description — exactly what Salman does with ↓ 40% API latency and 99.95% uptime.