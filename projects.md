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
        - [gmock-sfinae-concepts-calculator](https://github.com/royyandzakiy/gmock-sfinae-concepts-calculator)

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

## Teaching
- [Beyond the Compiler - ITB](https://docs.google.com/presentation/d/1bUpNFzojfTJrPyuKZOrNM5WjTlhIKrd-ILqr3q75GxI/edit?slide=id.g3cadaa0abd0_0_501#slide=id.g3cadaa0abd0_0_501)
- [Teknologi Budidaya Berbasis IoT - IPB](https://docs.google.com/presentation/d/12a08WXFfVqDvR8ned3B6okymbPfukO0dl5oTxakRE1U/edit?slide=id.g13a78b40027_0_78#slide=id.g13a78b40027_0_78)

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
- zephyr project boilerplate: etl, devcont, cicd, docs, rls pg
- zephyr sysbuild, secure boot
- zephyr jenkins
- C++ library project template
- zephyr ble dfu