# TRD: [Project Name Template]

**Version:** `1.0` (Last Updated: [YYYY-MM-DD])
**PRD:** [Link to PRD Document]

---

# Internal Project Team

- **Contributors**: [Dev team]
  - [Name] - Tech Lead / Embedded Lead (R)
  - [Name] - Firmware Developer (C)
  - [Name] - Desktop/App Developer (C)
  - [Name] - QA / HIL Engineer (C)
  - [Name] - DevOps (C)

*note: RACI (Responsible, Accountable, Contributor, Informed)*

---

# Table of Contents

---

## 1. System Architecture

### 1.1 Architecture Pattern

[Describe architecture: e.g., standalone embedded, BLE to mobile, USB to desktop, cloud-connected]

```
+-------------------------+       [PROTOCOL]       +-------------------------+
|   [Device Name]         |  <------------------>  |   [Companion Name]      |
|   ([MCU] + [RTOS])      |     ([details])        |   ([Language/Framework]) |
+-------------------------+                       +-------------------------+
          ↑                                              |
          |                                              ↓
   +--------------------+                       +---------------------+
   | [Sensors/Periphs]  |                       | [Storage/Dest]      |
   | [list]             |                       | [format]            |
   +--------------------+                       +---------------------+
```

**Alternative:** [If another architecture exists, describe]

### 1.2 Technology Stack

| Layer | Technology | Notes |
| --- | --- | --- |
| **RTOS/SDK** | [e.g., Zephyr, FreeRTOS, NCS, ESP-IDF] | [version] |
| **Connectivity** | [e.g., BLE, Wi-Fi, USB, Ethernet] | [details: speed, version] |
| **MCU** | [e.g., nRF5340, ESP32-S3, STM32H7] | [core, memory] |
| **Sensors/Peripherals** | [list] | [interface: I2C, SPI, etc.] |
| **Firmware Language** | [e.g., C++23, C11, Rust] | [constraints: no RTTI, etc.] |
| **Firmware Libraries** | [e.g., ETL, no-STL, custom HAL] | [key libs] |
| **State Machine (FW)** | [e.g., Zephyr SMF, custom enum, Queues] | [traceability method] |
| **Companion Language** | [e.g., C++23, TypeScript, Python] | [version] |
| **Companion Framework** | [e.g., Qt6, React Native, Tauri] | [details] |
| **Companion State Machine** | [e.g., Boost.SML, XState, custom] | [if applicable] |
| **Build System (FW)** | [e.g., West+CMake, Make, PlatformIO] | [details] |
| **Build System (Companion)** | [e.g., CMake + vcpkg, npm, pip] | [OS/IDE agnostic?] |
| **CI/CD (Non-HIL)** | [e.g., GitHub Actions, GitLab CI] | [native sim, unit tests] |
| **CI/CD (HIL)** | [e.g., Jenkins, self-hosted runner] | [hardware test automation] |
| **Testing (Non-HIL)** | [e.g., native sim, unit test framework] | [details] |
| **Testing (HIL)** | [e.g., pytest, custom harness] | [real hardware validation] |
| **Emulation/Mocking** | [e.g., Zephyr emul, FFF, mockator] | [how to mock drivers] |
| **Debugging** | [e.g., J-Link, RTT, GDB, logic analyzer] | [tools] |
| **Updates** | [e.g., OTA via BLE, USB DFU, no updates] | [bootloader] |
| **Static Analysis** | [e.g., clang-tidy, cppcheck, PVS-Studio] | [CI-enforced?] |
| **Code Formatting** | [e.g., clang-format, black, prettier] | [automatic] |
| **Documentation** | [e.g., Doxygen, Sphinx, Markdown] | [generated] |
| **Version Handling** | [e.g., SemVer + CMake, Git tags] | [single source of truth] |
| **Compiler (FW)** | [e.g., GCC ARM, Clang] | [flags: -Wall -Werror, etc.] |
| **Compiler (Companion)** | [e.g., GCC, Clang, MSVC] | [ASan, UBSan] |
| **Development Environment** | [e.g., Dev Containers, Docker, local] | [IDE extensions] |

### 1.3 Architecture Decision Records (ADR)

| Date | Decision | Description | Link |
| --- | --- | --- | --- |
| [YYYY-MM-DD] | [Decision title] | [Brief motivation] | [Link] |
| [YYYY-MM-DD] | [Decision title] | [Brief motivation] | [Link] |

### 1.4 Additional Technical Documents

- **Architecture Backlog**: [Link]
- **HAL Design**: [Link]
- **Communication Protocol Specification**: [Link]
- **Schematic / Pin Mapping**: [Link]
- **ERD (if applicable)**: [Link]

---

## 2. Repositories

### 2.1 Firmware Repository

**Repository Link:** [Link]
**Framework/SDK:** [e.g., NCS 3.3 + C++23 + West]

**Structure:**

```
[firmware]/
 ├─ .devcontainer/
 │   └─ devcontainer.json          # [if used]
 ├─ src/
 │   ├─ main.[c/cpp]
 │   ├─ [module1]/
 │   ├─ [module2]/
 │   └─ [module3]/
 ├─ include/
 ├─ boards/                         # [custom board definitions]
 ├─ drivers/                        # [custom drivers]
 ├─ emul/                           # [emulators for testing, if any]
 ├─ tests/
 │   ├─ unit/
 │   ├─ integration/
 │   └─ native_sim/
 ├─ scripts/
 ├─ .clang-format                   # [if applicable]
 ├─ .clang-tidy                     # [if applicable]
 ├─ Doxyfile                        # [if applicable]
 └─ CMakeLists.txt / Makefile / ...
```

**Key Dependencies:**

- [dep1] – [purpose]
- [dep2] – [purpose]

**Constrained Features (if embedded):**
- No RTTI: [Yes/No]
- No exceptions: [Yes/No]
- No dynamic polymorphism: [Yes/No]
- STL replacement: [e.g., ETL, none]

**Build Command:**

```shell
[build command]
```

**Flash Command:**

```shell
[flash command]
```

**CI Pipeline (Non-HIL):**

```yaml
[CI steps summary]
```

### 2.2 Companion Repository ([Desktop/Mobile/Web])

**Repository Link:** [Link]
**Framework:** [e.g., Qt6 + C++23, React + TS]

**Structure:**

```
[companion]/
 ├─ .devcontainer/
 │   └─ devcontainer.json
 ├─ src/
 │   ├─ main.[cpp/ts/...]
 │   ├─ [module1]/
 │   ├─ [module2]/
 │   └─ [module3]/
 ├─ tests/
 │   ├─ unit/
 │   └─ integration/
 ├─ [vcpkg.json / package.json / requirements.txt]
 ├─ .clang-format
 ├─ Doxyfile
 └─ CMakeLists.txt
```

**Key Dependencies:**

- [dep1] – [purpose]
- [dep2] – [purpose]

### 2.3 HIL Test Repository (if applicable)

**Repository Link:** [Link]
**Framework:** [e.g., pytest, custom runner]

**Structure:**

```
hil-tests/
 ├─ fixtures/
 │   ├─ device_controller.[py/cpp]
 │   ├─ [protocol]_simulator.[py/cpp]
 │   └─ power_monitor.[py/cpp]
 ├─ tests/
 │   ├─ test_[feature1].py
 │   ├─ test_[feature2].py
 │   └─ ...
 ├─ [requirements.txt / CMakeLists.txt]
 ├─ [Jenkinsfile / .gitlab-ci.yml]
 └─ pytest.ini
```

---

## 3. Functional Modules

### 3.1 [Module Name]

**Purpose:** [What this module does]

**Key Functions / API:**

| Function / Interface | Description |
| --- | --- |
| `[function_name()]` | [description] |
| `[function_name()]` | [description] |

**State Machine (if applicable):**

```
[IDLE] -- [event] --> [ACTIVE] -- [event] --> [ERROR]
```

**Dependencies:** [list other modules it depends on]

### 3.2 [Module Name]

**Purpose:** [What this module does]

**Key Functions / API:**

| Function / Interface | Description |
| --- | --- |
| `[function_name()]` | [description] |

---

## 4. Data Flow

```
[Trigger/Interrupt]
 ↓
[ISR / Event Handler]
 ↓
[Thread/Task Name]
 ↓
[Processing Step]
 ↓
[Queuing / Buffering]
 ↓
[Output / Transmission]
 ↓
[Destination]
```

**Example Flow:**

[Specific scenario] → [action] → [next action] → [result]

---

## 5. Non-Functional Requirements

| Category | Requirement |
| --- | --- |
| **Performance** | [e.g., latency ≤X ms, throughput ≥Y units/sec] |
| **Power** | [e.g., battery life ≥Z days, sleep current ≤X µA] |
| **Memory** | [e.g., RAM ≤X KB, Flash ≤Y MB] |
| **Usability** | [e.g., OTA update <X min, connection <Y sec] |
| **Security** | [e.g., encrypted comms, signed updates, secure boot] |
| **Availability** | [e.g., uptime target, watchdog recovery] |
| **Maintainability** | [e.g., test coverage %, static analysis enforced] |
| **Extensibility** | [e.g., modular drivers via concepts/interfaces] |
| **Traceability** | [e.g., state machine logging, error codes documented] |

---

## 6. CI/CD Pipeline

### 6.1 Non-HIL (e.g., GitHub Actions)

**Triggers:** [e.g., PR, push to non-main]

**Pipeline Steps:**

1. [Setup environment]
2. [Build for native simulation]
3. [Run unit tests]
4. [Run static analysis]
5. [Run formatting checks]
6. [Generate documentation]
7. [Build for target hardware]

### 6.2 HIL (e.g., Jenkins)

**Triggers:** [e.g., merge to main, nightly, manual]

**Hardware Setup:**

- [X] devices in test jig
- [Programmer type]
- [Measurement equipment]
- [Protocol sniffer/analyzer]

**Pipeline Steps:**

1. [Flash devices]
2. [Power cycle + boot verify]
3. [Run test suite]
   - [Test 1]
   - [Test 2]
   - [Test 3]
4. [Generate report]
5. [Deploy if all pass]

---

## 7. Logging & Monitoring

| Layer | Tool | Purpose |
| --- | --- | --- |
| Firmware | [e.g., RTT, UART, logging lib] | [runtime debug, error capture] |
| Companion | [e.g., spdlog, console] | [structured logs] |
| CI (Non-HIL) | [e.g., GitHub Actions logs] | [build/test output] |
| HIL | [e.g., Jenkins, pytest logs] | [test visualization] |
| Post-processing | [e.g., Python script] | [telemetry parsing] |

---

## 8. Deployment / Update Mechanism

| Environment | Target | Purpose |
| --- | --- | --- |
| DEV | [e.g., dev kit] | [development] |
| TEST | [e.g., HIL jig] | [automated testing] |
| PROD | [e.g., field devices] | [production] |

**Update Flow (if OTA/DFU):**

1. [Build + sign firmware]
2. [Upload to server]
3. [Client checks for update]
4. [User confirms (or automatic)]
5. [Transfer via protocol]
6. [Bootloader swaps images]
7. [Reboot]

---

## 9. Error Handling

| Error / Condition | Recovery Action |
| --- | --- |
| [e.g., Sensor read failure] | [e.g., re-init bus, retry x3, reset sensor] |
| [e.g., Comm disconnect] | [e.g., re-enter advertising, preserve state] |
| [e.g., Flash corruption] | [e.g., fallback to factory partition] |
| [e.g., Update verify fail] | [e.g., abort, reboot old image] |
| [e.g., Low battery] | [e.g., reduce sample rate, enter sleep] |

**Watchdog:** [e.g., X-second window, fed from main loop only]

---

## 10. Security Overview

| Category | Implementation |
| --- | --- |
| **Communication Security** | [e.g., LE Secure Connections, TLS, none] |
| **Privacy** | [e.g., RPA, MAC randomization] |
| **Firmware Signing** | [e.g., MCUboot ECDSA, no signing] |
| **Rollback Protection** | [e.g., version check, anti-rollback] |
| **Secure Storage** | [e.g., flash encryption, OTP] |
| **Debug Port** | [e.g., disabled in production, locked] |
| **Device Identity** | [e.g., unique serial, certificate] |

---

## 11. Naming Conventions

| Context | Convention |
| --- | --- |
| [Language] Classes / Types | [e.g., PascalCase] |
| [Language] Methods / Functions | [e.g., camelCase()] |
| [Language] Variables | [e.g., snake_case_] |
| Constants | [e.g., kPascalCase, UPPER_CASE] |
| Configuration Macros | [e.g., CONFIG_UPPER_SNAKE] |
| Device Tree Nodes | [e.g., vendor,node-name] |
| Protocol Characteristics | [e.g., camelCase] |
| Git Branches | [e.g., feature/desc, bugfix/desc] |
| Commit Messages | [e.g., Conventional Commits] |

---

## 12. Documentation

**Tool:** [e.g., Doxygen, Sphinx, MkDocs]

**Generation Command:**

```shell
[doxygen command]
```

**Key Groups/Tags:**

- `@defgroup` – [purpose]
- `@file` – [file header]
- `@param` – [parameter description]
- `@return` – [return description]
- `@note` – [important notes]

**Documentation Hosting:** [e.g., GitHub Pages, internal wiki, none]

---

## 13. Future Enhancements

- [Enhancement 1]
- [Enhancement 2]
- [Enhancement 3]

---

## 14. Open Questions / Risks

| Question / Risk | Impact | Mitigation / Answer |
| --- | --- | --- |
| [e.g., Sensor X not responding after deep sleep] | [High/Med/Low] | [e.g., add reset GPIO line] |
| [e.g., BLE throughput not meeting spec] | [High] | [e.g., fallback to 1M PHY] |

---

## 15. Deployment Diagram

```
+---------------------------+        +---------------------------+
| [CI System 1]             | -----> | [Build Artifacts]         |
| ([type])                  |        | ([validation type])       |
+---------------------------+        +---------------------------+
          |                                       |
          | ([trigger condition])                 | ([trigger condition])
          v                                       v
+---------------------------+        +---------------------------+
| [CI System 2 / HIL]       | -----> | [HIL Test Setup]          |
| ([type])                  |        | - [hardware x N]          |
|                           |        | - [programmer]            |
|                           |        | - [measurement equipment] |
+---------------------------+        +---------------------------+
          |
          | ([if all tests pass])
          v
+---------------------------+
| [Update Server / Storage] |
| ([signed artifacts])      |
+---------------------------+
          ↑
          |
+---------------------------+
| [Companion App]           |
| ([platform])              |
+---------------------------+
          |
          | ([protocol])
          v
+---------------------------+
| [Target Device]           |
| ([MCU] + [RTOS])          |
| - [sensor 1]              |
| - [sensor 2]              |
| - [actuator]              |
+---------------------------+
```

---

## Appendix A: [Optional - Pin Mapping Table]

| Pin | Function | Signal Name | Notes |
| --- | --- | --- | --- |
| P0.01 | I2C SCL | SENSOR_SCL | Pull-up 4.7k |
| P0.02 | I2C SDA | SENSOR_SDA | Pull-up 4.7k |
| ... | ... | ... | ... |

---

## Appendix B: [Optional - Acronyms]

| Acronym | Definition |
| --- | --- |
| BLE | Bluetooth Low Energy |
| DLE | Data Length Extension |
| HIL | Hardware-in-the-Loop |
| NCS | nRF Connect SDK |
| OTA | Over-the-Air |
| RTT | Real-Time Transfer |
| SMF | State Machine Framework |