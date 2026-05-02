# TRD: [Project Name Template]

**Version:** `1.0` (Last Updated: 2026-4-29)
**PRD:** [Link to Internal PRD Document]

---

## 1. System Architecture

### 1.1 Architecture Pattern

Embedded firmware on custom hardware with BLE 5.4 (High Throughput) connectivity to companion desktop application:

```
+-------------------------+       BLE 5.4 GATT      +-------------------------+
|   Body Sensor Device    |  <------------------>  |   Companion Desktop App  |
|   (nRF5340 + NCS 3.3)   |     (2 Mbps, DLE)      |   (C++23 / Qt6 + BlueZ)  |
+-------------------------+                       +-------------------------+
          ↑                                              |
          |                                              ↓
   +--------------------+                       +---------------------+
   | Onboard Sensors    |                       | Local Storage       |
   | (HR, Temp, IMU)    |                       | (CSV/Parquet)       |
   +--------------------+                       +---------------------+
          |
          ↓
   +--------------------+
   | Flash Storage      |
   | (Data Buffering)   |
   +--------------------+
```

### 1.2 Technology Stack

| Layer | Technology | Notes |
| --- | --- | --- |
| **RTOS/SDK** | nRF Connect SDK (NCS) 3.3 | Includes Zephyr RTOS |
| **Connectivity** | Bluetooth 5.4 | 2 Mbps PHY, DLE (251 bytes), High Throughput |
| **MCU** | nRF5340 | Dual-core ARM Cortex-M33 (App + Net cores) |
| **Sensors** | MAX30102 (HR), BME280 (Temp), BMI270 (IMU) | I2C/SPI interface |
| **Firmware Language** | C++23 | Modern C++ for embedded, no RTTI/no dynamic_cast |
| **Firmware Library** | ETL (Embedded Template Library) | STL-like for constrained environments |
| **State Machine (FW)** | Zephyr SMF | State Machine Framework (traceable) |
| **Desktop App Language** | C++23 | Qt6 + BlueZ (Linux) |
| **State Machine (Desktop)** | Boost.SML | High-performance, compile-time state machines |
| **Build System (FW)** | West + CMake + Ninja | NCS's meta-tool |
| **Build System (Desktop)** | Modern CMake (≥3.20) + vcpkg | OS/IDE agnostic, manifest mode |
| **CI/CD (Non-HIL)** | GitHub Actions | Native simulation builds, unit tests |
| **CI/CD (HIL)** | Jenkins | Hardware-in-the-Loop test automation |
| **Testing (Non-HIL)** | Native POSIX simulation + ztest | Zephyr's native_sim board |
| **Testing (HIL)** | pytest + custom C++ fixtures | Real hardware validation |
| **Emulation** | Zephyr emul subsystem | Mock sensors for unit testing |
| **Mocking (FW)** | ztest mock + emul | No dynamic polymorphism |
| **Debugging** | Segger RTT, J-Link, nRF Connect for Desktop | Real-time logging |
| **Firmware Updates** | MCUboot + SMP Server | Over-the-air (OTA) via desktop app |
| **Static Analysis** | clang-tidy, clang-analyzer, cmake-format | CI-enforced |
| **Code Formatting** | clang-format, cmake-format | Automatic formatting |
| **Documentation** | Doxygen | Both firmware and desktop |
| **Version Handling** | Semantic Versioning + CMake project version | Single source of truth |
| **Compiler (FW)** | GCC ARM None EABI | All static analyzers enabled (-Wall -Wextra -Wconversion -Wshadow -Wundef -Wpedantic -fanalyzer) |
| **Compiler (Desktop)** | GCC/Clang | Address Sanitizer, Undefined Behavior Sanitizer |
| **Development Environment** | Dev Containers (VS Code) | nRF Connect extension pre-installed |

### 1.3 Architecture Decision Record (ADR)

| **Date of Approval** | **Decision** | **Description** |
| --- | --- | --- |
| [Date] | Approved/Ongoing | Motivation: [Link to Internal Design Doc] |

### 1.4 Additional Technical Documents

- **Architecture Backlog**: [Internal Link]
- **Hardware Abstraction Layer (HAL) Design**: [Internal Link]
- **BLE GATT Profile Specification**: [Internal Link]
- **Pin Mapping & Schematics**: [Internal Link]

---

## 2. Repositories

### 2.1 Firmware Repository

**Repository Link:** [Internal Link]
**Framework:** NCS 3.3 + C++23 + West + ETL + Zephyr SMF

**.devcontainer/devcontainer.json:**
```json
{
    "name": "Firmware Dev Container",
    "image": "ghcr.io/zephyrproject-rtos/zephyr-build:latest",
    "extensions": [
        "ms-vscode.cpptools",
        "nordic-semiconductor.nrf-connect",
        "marus25.cortex-debug",
        "zacharyrees.vscode-cmake-format"
    ],
    "settings": {
        "nrf-connect.boardRoots": ["${workspaceFolder}/boards"]
    }
}
```

**Structure:**

```
firmware/
 ├─ .devcontainer/
 │   └─ devcontainer.json
 ├─ app/
 │   ├─ src/
 │   │   ├─ main.cpp
 │   │   ├─ smf_states.cpp          # Zephyr SMF state definitions
 │   │   ├─ sensors/
 │   │   │   ├─ sensor_manager.cpp
 │   │   │   ├─ max30102.cpp
 │   │   │   ├─ bme280.cpp
 │   │   │   └─ bmi270.cpp
 │   │   ├─ ble/
 │   │   │   ├─ ble_service.cpp
 │   │   │   ├─ gatt_handlers.cpp
 │   │   │   └─ throughput_manager.cpp
 │   │   ├─ power/
 │   │   │   └─ power_manager.cpp
 │   │   ├─ data/
 │   │   │   ├─ ring_buffer.cpp     # ETL circular_buffer
 │   │   │   └─ flash_storage.cpp
 │   │   └─ utils/
 │   ├─ include/
 │   ├─ dts/
 │   │   └─ bindings/
 │   ├─ CMakeLists.txt
 │   ├─ prj.conf
 │   └─ version.h.in                # Version header template
 ├─ boards/
 │   └─ custom/
 │       ├─ board.c
 │       ├─ board.h
 │       └─ board_defconfig
 ├─ drivers/
 │   ├─ sensor/
 │   └─ custom/
 ├─ emul/                           # Zephyr emulators for testing
 │   ├─ emul_max30102.cpp
 │   ├─ emul_bme280.cpp
 │   └─ emul_bmi270.cpp
 ├─ tests/
 │   ├─ unit/                       # ztest unit tests
 │   ├─ integration/                # ztest integration
 │   └─ native_sim/
 │       └─ test_runner.cpp
 ├─ scripts/
 │   ├─ build.py
 │   ├─ flash.py
 │   └─ simulate.py
 ├─ .clang-format
 ├─ .clang-tidy
 ├─ .cmake-format.yaml
 ├─ Doxyfile
 └─ west.yml
```

**Dependencies (vcpkg.json / west.yml):**

- `nrfconnect/sdk-nrf` – NCS 3.3 core
- `mcuboot` – Bootloader for OTA
- `littlefs` – Flash file system
- `etl` – Embedded Template Library (no STL allocators)
- `nanopb` – Protocol buffers for serialization
- `mbedtls` – Cryptographic primitives

**C++23 Features Used (with constraints):**
- `std::span` – Buffer views
- `std::optional` – Error/null state handling
- `std::variant` – Sensor data types (compile-time dispatch)
- `concepts` – Driver interface constraints (no virtual)
- `constexpr` – Static configuration
- `std::chrono` – Time handling (Zephyr kernel time)

**Explicitly Disabled:**
- RTTI (`-fno-rtti`)
- Exceptions (`-fno-exceptions`)
- Dynamic polymorphism (no virtual functions in drivers)

**Build & Deploy:**

```shell
west build -b custom_board app -p always
west flash --runner jlink
```

**GitHub Actions (Non-HIL):**
```yaml
- name: Build native_sim with full warnings
  run: |
    west build -b native_sim app \
      -DCMAKE_CXX_FLAGS="-Wall -Wextra -Wconversion -Wshadow -Wundef -Wpedantic -fanalyzer"
- name: Run ztest suite
  run: ./build/zephyr/zephyr.exe --test
- name: clang-tidy
  run: run-clang-tidy -p build/
- name: clang-format check
  run: find . -name '*.cpp' -o -name '*.hpp' | xargs clang-format --dry-run --Werror
- name: cmake-format check
  run: cmake-format --check CMakeLists.txt
```

**Compiler Flags (GCC):**
```cmake
target_compile_options(firmware PRIVATE
    -Wall -Wextra -Wpedantic
    -Wconversion -Wshadow -Wundef
    -Wdouble-promotion -Wformat=2
    -Wno-unused-parameter
    -fno-rtti -fno-exceptions
    -ffunction-sections -fdata-sections
    -fanalyzer  # Static analysis in CI
)
```

**Env vars:**

```
BOARD=custom_board
CONF_FILE=prj.conf
NCS_VERSION=3.3.0
```

### 2.2 Companion Desktop Application Repository

**Repository Link:** [Internal Link]
**Framework:** C++23 + Qt6 + BlueZ + Boost.SML

**.devcontainer/devcontainer.json:**
```json
{
    "name": "Desktop Dev Container",
    "image": "mcr.microsoft.com/devcontainers/cpp:ubuntu-22.04",
    "features": {
        "ghcr.io/devcontainers/features/qt:1": {
            "version": "6.5"
        }
    },
    "postCreateCommand": "vcpkg install"
}
```

**Structure:**

```
desktop-app/
 ├─ .devcontainer/
 │   └─ devcontainer.json
 ├─ src/
 │   ├─ main.cpp
 │   ├─ sml_states.cpp               # Boost.SML state definitions
 │   ├─ ble/
 │   │   ├─ ble_scanner.cpp          # BlueZ D-Bus API
 │   │   ├─ gatt_client.cpp
 │   │   └─ throughput_monitor.cpp
 │   ├─ data/
 │   │   ├─ data_recorder.cpp
 │   │   └─ data_export.cpp
 │   ├─ ui/
 │   │   ├─ main_window.cpp
 │   │   ├─ sensor_dashboard.cpp
 │   │   └─ ota_updater.cpp
 │   └─ utils/
 ├─ tests/
 │   ├─ unit/                        # GTest unit tests
 │   └─ integration/                 # GMock integration
 ├─ CMakeLists.txt
 ├─ vcpkg.json                       # Manifest mode
 ├─ .clang-format
 ├─ .clang-tidy
 ├─ .cmake-format.yaml
 ├─ Doxyfile
 └─ version.h.in
```

**vcpkg.json:**
```json
{
    "name": "body-sensor-desktop",
    "version": "1.0.0",
    "dependencies": [
        "qt6-base",
        "qt6-charts",
        "sqlite3",
        "spdlog",
        "gtest",
        "gmock",
        "boost-sml",
        "libprotobuf"
    ]
}
```

**Boost.SML Example:**
```cpp
struct BleEvents {
    struct connect { std::string address; };
    struct disconnect {};
    struct data_received { std::vector<uint8_t> payload; };
    struct update_available { std::string version; };
};

struct BleStateMachine {
    auto operator()() {
        using namespace boost::sml;
        return make_transition_table(
            *"disconnected"_s + event<BleEvents::connect> = "connecting"_s,
            "connecting"_s + event<BleEvents::data_received> = "connected"_s,
            "connected"_s + event<BleEvents::disconnect> = "disconnected"_s,
            "connected"_s + event<BleEvents::update_available> = "updating"_s,
            "updating"_s + event<BleEvents::data_received> = "connected"_s
        );
    }
};
```

### 2.3 HIL Test Repository (Jenkins + pytest)

**Repository Link:** [Internal Link]
**Framework:** Python + pytest + custom C++ fixtures

**Structure:**

```
hil-tests/
 ├─ fixtures/
 │   ├─ device_controller.py        # PySerial + J-Link command line
 │   ├─ ble_scanner.py              # BlueZ integration
 │   └─ power_monitor.py            # SCPI over LAN
 ├─ tests/
 │   ├─ test_ble_throughput.py
 │   ├─ test_sensor_accuracy.py
 │   ├─ test_power_consumption.py
 │   ├─ test_ota_update.py
 │   └─ test_reliability.py
 ├─ conftest.py                      # pytest fixtures
 ├─ requirements.txt
 ├─ Jenkinsfile
 └─ pytest.ini
```

**pytest Fixtures:**
```python
@pytest.fixture
def device():
    dev = DeviceController(serial="12345678")
    dev.flash_firmware("latest.hex")
    dev.power_cycle()
    yield dev
    dev.cleanup()

@pytest.fixture
def ble_connection(device):
    scanner = BLEScanner()
    device.start_advertising()
    connection = scanner.connect(device.address)
    yield connection
    connection.disconnect()
```

---

## 3. Functional Modules

### 3.1 Sensor Acquisition Module (C++23 + Concepts + ETL)

**Purpose:** Acquire data from onboard body sensors (heart rate, temperature, IMU)

**Concept-Based Interface (No Virtual):**

```cpp
template<typename T>
concept SensorDriver = requires(T& t, uint8_t* buf, size_t len) {
    // Compile-time interface (no dynamic dispatch)
    { T::init() } -> std::same_as<bool>;
    { t.sample() } -> std::same_as<std::optional<typename T::data_type>>;
    { t.read_fifo(etl::span<uint8_t> buf) } -> std::same_as<size_t>;
    // Static asserts for required types
    requires std::is_standard_layout_v<typename T::config_type>;
};

// Concrete driver (no inheritance)
class Max30102Driver {
public:
    using data_type = etl::array<uint16_t, 200>;  // ETL array, not std::array
    using config_type = struct { uint8_t led_current; };
    
    static bool init();
    std::optional<data_type> sample();
    size_t read_fifo(etl::span<uint8_t> buffer);
};
```

**SensorManager (Template-based, No Virtual):**

```cpp
template<SensorDriver... Drivers>
class SensorManager {
public:
    bool init() {
        return (Drivers::init() && ...);  // C++17 fold expression
    }
    
    auto sample_all() {
        etl::vector<std::variant<typename Drivers::data_type...>, sizeof...(Drivers)> results;
        (results.push_back(Drivers{}.sample().value_or(typename Drivers::data_type{})), ...);
        return results;
    }
    
private:
    // Zephyr thread, not std::jthread
    static void sampling_thread_fn(void*, void*, void*);
    
    // Zephyr SMF state tracking
    struct smf_state {
        const char* name;
        uint32_t samples_sent;
    } state_;
};
```

**Key Functions:**

| Function | Description |
| --- | --- |
| `SensorManager::sample_all()` | Sample all sensors, return variant data (ETL) |
| `HeartRateDriver::calculate_bpm()` | Process PPG data (constexpr lookup tables) |

### 3.2 Zephyr SMF State Machine (Firmware)

```cpp
#include <zephyr/smf.h>

/* States */
static void sensor_idle_entry(void* o) { /* entry actions */ }
static void sensor_sampling_run(void* o) { /* sampling logic */ }
static void sensor_error_handle(void* o) { /* error recovery */ }

/* State transitions - fully traceable */
static const struct smf_state sensor_states[] = {
    [SENSOR_STATE_IDLE] = SMF_CREATE_STATE(
        .entry = sensor_idle_entry,
        .run = sensor_idle_run,
        .exit = sensor_idle_exit
    ),
    [SENSOR_STATE_SAMPLING] = SMF_CREATE_STATE(
        .entry = sensor_sampling_entry,
        .run = sensor_sampling_run  // Called in thread loop
    ),
    [SENSOR_STATE_ERROR] = SMF_CREATE_STATE(.run = sensor_error_handle)
};

SMF_STATE_ENTRY_DEFINE(sensor_state, sensor_states, SENSOR_STATE_IDLE);
```

### 3.3 BLE Communication Module (High Throughput)

**Purpose:** Manage BLE 5.4 connections with 2 Mbps PHY and DLE

**GATT Services:**

| Service | UUID | Characteristics | Throughput |
| --- | --- | --- | --- |
| Device Info | 0x180A | Manufacturer, Serial, FW Version | - |
| Sensor Data | Custom | Data Stream (Notifications) | ~1.4 Mbps |
| Control | Custom | Sample Rate, Commands | - |
| OTA Control | Custom | Firmware Image, Control Point | ~200 Kbps |

**Throughput Configuration:**
- PHY: 2 Mbps
- ATT MTU: 247 bytes
- DLE: 251 bytes max payload
- Connection Interval: 7.5 ms

**Zephyr SMF for BLE Connection:**
```cpp
static const struct smf_state ble_states[] = {
    [BLE_STATE_DISCONNECTED] = SMF_CREATE_STATE(
        .entry = ble_adv_start,
        .run = ble_wait_connection
    ),
    [BLE_STATE_CONNECTED] = SMF_CREATE_STATE(
        .entry = ble_negotiate_2m_phy,
        .run = ble_stream_data,
        .exit = ble_save_state
    )
};
```

### 3.4 Power Management Module

**Purpose:** Optimize battery life through dynamic power states

**Power States (nRF5340):**

| State | Current | Wake Sources | Latency |
| --- | --- | --- | --- |
| Active | ~25mA | All peripherals | - |
| Idle (Net core sleep) | ~10mA | BLE (2M PHY active) | 10μs |
| System OFF (App core) | ~3μA | GPIO, RTC, BLE wake | 200ms |

---

## 4. Data Flow (Zephyr Primitives)

```
Sensor Interrupt (GPIO)
 ↓
ISR → k_sem_give(&data_ready_sem)
 ↓
Sensor Thread (K_THREAD_DEFINE)
    k_sem_take(&data_ready_sem, K_FOREVER)
 ↓
Processing Pipeline (ETL + constexpr)
 ↓
Ring Buffer (ETL::circular_buffer, lock-free atomic)
 ↓
BLE Stack (NCS 3.3 Host + Controller)
 ↓
GATT Notification (2 Mbps PHY) → Desktop App
```

**Example Flow (High Throughput):**
IMU at 100 Hz → BMI270 FIFO interrupt → sensor thread reads burst (20 samples) → compress to CBOR → ETL span for zero-copy → BLE notification with DLE (251 bytes) → desktop app receives at ~1.4 Mbps

---

## 5. Non-Functional Requirements

| Category | Requirement |
| --- | --- |
| **Performance** | Sensor sampling ≥100 Hz; BLE throughput ≥1 Mbps sustained |
| **Latency** | End-to-end (sensor → desktop) ≤10 ms |
| **Power** | Battery life ≥5 days (continuous streaming) |
| **Memory** | RAM ≤256 KB (App core); Flash ≤1 MB (App core) |
| **Usability** | OTA update <2 minutes; Connection setup <3 seconds |
| **Security** | BLE 5.4 LE Secure Connections; Encrypted DFU |
| **Availability** | 99.9% uptime (watchdog recovery <500 ms) |
| **Traceability** | Zephyr SMF state transitions logged via Doxygen |
| **Maintainability** | C++23 concepts enforced; ztest coverage ≥80% via emul |
| **Extensibility** | Sensor drivers via ETL::variant and concepts (no virtual) |

---

## 6. CI/CD Pipeline

### 6.1 GitHub Actions (Non-HIL)

**Triggers:** PR, push to non-main branches

**Pipeline:**
```
1. Setup NCS 3.3 + devcontainer
2. west update (manifest fetch)
3. Build for native_sim with full GCC static analyzers
   - -Wall -Wextra -Wpedantic -Wconversion -Wshadow -Wundef -fanalyzer
4. Run ztest (unit tests with Zephyr emul subsystem)
5. Run clang-tidy (C++23 checks, no-virtual enforcement)
6. Run clang-format --check
7. Run cmake-format --check
8. Generate Doxygen documentation
9. Build for target board (nRF5340)
```

### 6.2 Jenkins (HIL) with pytest

**Triggers:** Merge to main, scheduled nightly, manual

**Hardware Setup:**
- 5x Production devices in test jig
- J-Link (programming + RTT)
- BLE sniffer (Ellisys or nRF Sniffer)
- Power monitor (Keysight or Nordic PPK2)

**Jenkins Pipeline + pytest:**
```
1. Flash all devices with latest firmware
2. Power cycle and verify boot
3. Run pytest suite:
   - test_ble_throughput.py (2 Mbps, DLE)
   - test_sensor_accuracy.py (against reference)
   - test_ota_update.py (full cycle)
   - test_power_consumption.py (48-hour soak)
   - test_reliability.py (1,000 connection cycles)
4. Generate HIL test report (pytest-html)
5. Deploy firmware signature if all pass
```

---

## 7. Documentation (Doxygen)

**Firmware Doxyfile:**
```doxygen
PROJECT_NAME = "Body Sensor Firmware"
INPUT = app/src app/include drivers emul
RECURSIVE = YES
EXTRACT_ALL = NO
EXTRACT_STATIC = NO
SOURCE_BROWSER = YES
STRIP_CODE_COMMENTS = NO
USE_MDFILE_AS_MAINPAGE = README.md
```

**Desktop Doxyfile:**
```doxygen
PROJECT_NAME = "Body Sensor Desktop"
INPUT = src
EXTRACT_PRIVATE = NO
EXTRACT_STATIC = NO
CALL_GRAPH = YES
```

**SMF State Documentation (Auto-extracted):**
```cpp
/**
 * @defgroup smf_ble BLE State Machine
 * @brief Traceable BLE connection states using Zephyr SMF
 *
 * State transition diagram:
 * DISCONNECTED -> CONNECTED (BLE_GAP_EVT_CONNECTED)
 * CONNECTED -> DISCONNECTED (BLE_GAP_EVT_DISCONNECTED)
 * CONNECTED -> STREAMING (user start command)
 */
```

---

## 8. Testing Strategy

### 8.1 Unit Testing (ztest + Zephyr emul)

```cpp
/* Mock sensor using Zephyr emul subsystem */
static int emul_max30102_sample(const struct emul *emul, struct sensor_value *val)
{
    /* Return predefined test values */
    val[0].val1 = 72;  /* BPM */
    return 0;
}

ZTEST(sensor_suite, test_sample_bpm)
{
    const struct device *sensor = DEVICE_DT_GET(DT_NODELABEL(max30102));
    ASSERT_TRUE(device_is_ready(sensor));
    
    struct sensor_value value;
    sensor_sample_fetch(sensor);
    sensor_channel_get(sensor, SENSOR_CHAN_HEART_RATE, &value);
    
    ASSERT_EQ(value.val1, 72);
}

ZTEST_SUITE(sensor_suite, NULL, NULL, NULL, NULL, NULL);
```

### 8.2 Mocking (No Virtual)

Using concepts + ztest emulation layer:

```cpp
template<typename T>
concept SensorDriver = requires(T& t) {
    { T::init() } -> std::same_as<bool>;
    { t.sample() } -> std::same_as<std::optional<typename T::data_type>>;
};

// Production driver
class RealSensor { ... };

// Test driver (compile-time substitution)
class MockSensor {
public:
    using data_type = etl::array<uint16_t, 200>;
    static bool init() { return true; }
    std::optional<data_type> sample() { return mock_data_; }
    void set_mock_data(data_type data) { mock_data_ = data; }
private:
    static data_type mock_data_;
};

// Test uses MockSensor, production uses RealSensor
template<typename SensorManager>
void test_driver() {
    SensorManager mgr;
    mgr.init();
    auto data = mgr.sample_all();
    ASSERT(data.size() > 0);
}
```

### 8.3 HIL Testing (pytest)

```python
def test_ble_throughput(device, ble_connection):
    """Test BLE 5.4 high throughput (2 Mbps + DLE)"""
    device.configure_2m_phy()
    device.enable_dle()
    
    bytes_received = 0
    start_time = time.time()
    
    for packet in ble_connection.receive_notifications(timeout=10):
        bytes_received += len(packet)
        if time.time() - start_time >= 5:
            break
    
    throughput = bytes_received / 5 / 1_000_000  # Mbps
    assert throughput >= 1.2  # Expect at least 1.2 Mbps
```

---

## 9. Error Handling (No Exceptions)

| Error Type | Condition | Recovery Action |
| --- | --- | --- |
| `sensor_timeout` | I2C NACK | Re-init bus, retry x3 → reset sensor via GPIO |
| `ble_disconnected` | Link loss | ret = smf_set_state(BLE_STATE_DISCONNECTED) |
| `flash_corrupt` | CRC mismatch | `std::expected<T, Error>` fallback to factory |
| `ota_verify_fail` | Signature invalid | Return `-EINVAL`, abort, reboot old image |
| `low_battery` | Voltage <3.0V | `power_manager_deep_sleep()` |

**Watchdog Configuration:**
```c
#define WATCHDOG_TIMEOUT_MS 60000
watchdog_install(wdt, WATCHDOG_TIMEOUT_MS);
watchdog_feed(wdt); /* Called only in main loop */
```

---

## 10. Security Overview

| Category | Implementation |
| --- | --- |
| **BLE Security** | LE Secure Connections (Just Works/Passkey) |
| **BLE Privacy** | Resolvable Private Addresses (RPA) |
| **Firmware Signing** | MCUboot with ECDSA P-256 signatures |
| **Rollback Protection** | Image version check in bootloader |
| **Secure Storage** | NCS Settings subsystem + OTP |
| **Debug Port** | Disabled in production via APPROTECT |
| **Device Identity** | Factory-programmed unique serial + cert |

---

## 11. Naming Conventions

| Context | Convention |
| --- | --- |
| C++ Classes | PascalCase |
| C++ Methods | camelCase() |
| C++ Variables | snake_case_ (private) |
| C++ Constants | kPascalCase |
| Kconfig Options | CONFIG_UPPER_SNAKE_CASE |
| Device Tree Nodes | vendor,node-name |
| GATT Characteristics | camelCase |
| Git Branches | feature/description, bugfix/description |
| CMake Variables | snake_case |
| Doxygen Groups | @defgroup lower_case |

---

## 12. Future Enhancements

- Channel sounding for high-precision ranging (BLE 5.4 feature)
- Multiple sensor fusion (Madgwick filter)
- On-device anomaly detection (C++23 constexpr ML)
- LE Audio for real-time alerts
- Dual-core offloading (Network core handles BLE stack)

---

## 13. C++23 Specific Features Utilized

| Feature | Usage | Embedded Safe? |
| --- | --- | --- |
| `std::span` | Buffer views without copying | ✅ (no allocation) |
| `std::optional` | Sensor read may fail | ✅ (stack-only) |
| `std::expected` | Error propagation | ✅ (C++23, no exception) |
| `std::variant` | Multiple sensor data types | ✅ (compile-time dispatch) |
| `concepts` | Sensor driver constraints | ✅ (zero overhead) |
| `constexpr` | Lookup tables, calibration | ✅ |
| `std::chrono` | k_uptime_get() wrapper | ✅ |
| `std::atomic` | Lock-free ring buffer | ✅ (no mutex) |
| `designated initializers` | SMF state init | ✅ |

**Excluded (not safe for embedded):**
- `std::jthread` → use Zephyr `k_thread`
- RTTI → `-fno-rtti`
- Exceptions → `-fno-exceptions`
- Dynamic dispatch → concepts + templates
- STL containers → ETL alternatives

---

## 14. Build & Version Management

**Single Version Source (version.h.in):**
```cmake
# CMakeLists.txt
project(body-sensor-firmware VERSION 1.2.3)

configure_file(version.h.in ${CMAKE_CURRENT_BINARY_DIR}/version.h)

# Git tags enforce version
set(CPACK_GENERATOR "TGZ")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
```

**Compiler Flags (Full Static Analysis):**
```cmake
target_compile_options(firmware PRIVATE
    $<$<CXX_COMPILER_ID:GNU>:
        -Wall -Wextra -Wpedantic
        -Wconversion -Wshadow -Wundef
        -Wdouble-promotion -Wformat=2
        -Wno-unused-parameter
        -ffunction-sections -fdata-sections
        -fanalyzer  # Deep static analysis in CI
        -Werror     # Treat warnings as errors in CI
    >
)
```

---

## 15. Deployment Diagram

```
+---------------------------+        +---------------------------+
| Dev Container (VS Code)   |        | GitHub Actions            |
| - nRF Connect extension   | -----> | - native_sim build        |
| - clangd                  |        | - ztest + emul            |
| - Cortex-Debug            |        | - clang-tidy              |
+---------------------------+        | - clang-format            |
          |                           | - doxygen                 |
          | (PR merge)                +---------------------------+
          v                                       |
+---------------------------+                      | (trigger)
| Jenkins (HIL + pytest)    |                      |
| - Run pytest suite        |                      |
| - BLE throughput test     |                      |
| - Sensor accuracy         |                      |
| - OTA cycle               |                      |
| - Power soak (48h)        |                      |
+---------------------------+                      |
          |                                       |
          | (all HIL tests pass)                  v
          v                           +---------------------------+
+---------------------------+          | OTA Update Server         |
| HIL Test Jig              |          | (Signed firmware)         |
| - 5 x nRF5340 devices     |          +---------------------------+
| - J-Link x5               |                      ↑
| - Power monitor (PPK2)    |                      |
| - BLE sniffer             |                      |
+---------------------------+                      |
                                                   |
+---------------------------+                      |
| Companion Desktop App     |                      |
| (C++23 / Qt6 / BlueZ)     | --------------------+
| - Boost.SML state machine | (BLE 5.4, 2 Mbps, DLE)
| - vcpkg dependencies      |
+---------------------------+                      |
          |                                       |
          v                                       |
+---------------------------+                      |
| Body Sensor Device        | <--------------------+
| (nRF5340 + NCS 3.3)       |
| - Zephyr SMF (traceable)  |
| - ETL containers          |
| - MAX30102, BME280, BMI270|
| - No RTTI / no exceptions |
+---------------------------+
```