# Testing Philosophy
*Distilled from observed patterns across multiple C++ projects*

---

## Core Principle: Mirror the Architecture

**The test suite is a shadow of the production code's structure.**
Each production component gets its own test file.
Each layer owns its own correctness before integration is attempted.
This is not just organization — it is a statement about responsibility.

**Example.** A Bluetooth weather station has this production pipeline:

```
SensorReader  →  PacketBuilder  →  BleTransmitter
                                         ↓
                               BleReceiver  →  PacketDecoder  →  ReadingStore
```

The test suite mirrors it exactly:

```
Production code              →    Test files
──────────────────────────────────────────────────────
src/SensorReader.cpp              Test/TestSensorReader.cpp
src/PacketBuilder.cpp             Test/TestPacketBuilder.cpp
src/PacketDecoder.cpp             Test/TestPacketDecoder.cpp
src/ReadingStore.cpp              Test/TestReadingStore.cpp
src/Utils.cpp                     Test/TestUtils.cpp
(all together)                    Test/TestIntegration.cpp
(combinatorial cases)             Test/TestIntegrationGenerated.cpp
```

If `PacketDecoder` is broken, `TestPacketDecoder` tells you immediately — without having to trace a failing integration test back through five layers.

---

## The Layer Contract

Tests flow in the same order as data does in production:

```
Unit layer:      prove each component's contract in isolation
Helper layer:    prove stateless utility functions directly
Integration:     prove the pipeline composes correctly
Generated/bulk:  prove breadth across a behavior family
```

Never test integration before units pass. Never test units through integration.

**Example — what belongs at each layer:**

```
Unit:
  Does SensorReader clamp temperature to the valid -40°C–85°C range?
  Does PacketBuilder set the correct 2-byte CRC at the end of every packet?
  Does PacketDecoder reject a packet whose CRC doesn't match?
  Does ReadingStore return the most recent reading when queried?

Helper (stateless utils):
  Does celsius_to_fahrenheit(0.0) return 32.0?
  Does encode_humidity(55.3) pack into the right 2-byte format?
  Does checksum(buffer) produce the expected value for a known byte sequence?

Integration:
  Does a full sensor reading pass through PacketBuilder → BleTransmitter →
  BleReceiver → PacketDecoder and arrive at ReadingStore unchanged?

Generated/bulk:
  500 parameterized (raw_sensor_value, expected_decoded_value) pairs covering
  negative temperatures, humidity at 0% and 100%, pressure at altitude
  extremes, and malformed/truncated packets.
```

---

## How a Test Is Defined

A test answers exactly one question. The name is the question.

```
TEST(SensorReader, ClampsTemperatureAboveMax)   → Does 120°C get clamped to 85°C?
TEST(PacketBuilder, CrcIsLastTwoBytes)          → Does the CRC always occupy bytes [n-2, n-1]?
TEST(PacketDecoder, RejectsTruncatedPacket)     → Does a 3-byte packet (min is 6) return an error?
TEST(ReadingStore, LatestReadingOverwritesPrev) → Does a newer reading replace the stored one?
```

The format is always: `TEST(Component, SpecificBehavior)`. The component is the class or function under test. The behavior is the atomic fact being asserted.

**One behavior. One test. Always.**

**Counterexample — what not to do:**

```cpp
// BAD: three behaviors jammed into one test
TEST(PacketDecoder, DecodesPackets) {
    EXPECT_EQ(decode_temp(packet_a), 21.5f);
    EXPECT_EQ(decode_humidity(packet_b), 60.0f);    // separate fact
    EXPECT_FALSE(decode(truncated_packet).ok());     // separate fact again
}

// GOOD: each fact is independently provable and independently named
TEST(PacketDecoder, DecodesTemperature)     { EXPECT_EQ(decode_temp(packet_a),     21.5f); }
TEST(PacketDecoder, DecodesHumidity)        { EXPECT_EQ(decode_humidity(packet_b), 60.0f); }
TEST(PacketDecoder, RejectsTruncatedPacket) { EXPECT_FALSE(decode(truncated).ok());        }
```

When a test fails, you want the name to tell you exactly what broke.

---

## How to Define What to Test

For each component, enumerate:

1. **The contract** — every input type and mode it accepts
2. **The variants** — each flag, option, or configuration that changes behavior
3. **The edge cases** — boundary values, empty input, max/min sensor ranges
4. **The error paths** — what should fail and how (bad CRC, truncated packet, out-of-range value)
5. **The return value separately from the side effect** — if a function mutates and returns a bool, test both

Do not skip the boring enumeration. The obvious cases catch the bugs that "obviously work."

**Example — enumerating `PacketDecoder`:**

```
Contract (valid packets):
  decode(temp_packet)       → Reading with correct °C value
  decode(humidity_packet)   → Reading with correct % value
  decode(pressure_packet)   → Reading with correct hPa value
  decode(combined_packet)   → Reading with all three fields populated

Variants (encoding options):
  decode(fahrenheit_flag_set) → temperature field is in °F, convert before storing
  decode(high_res_flag_set)   → humidity has 0.1% precision instead of 1%

Edge cases:
  decode(temp = -40°C)   → lowest valid temperature, must not be clamped
  decode(temp = 85°C)    → highest valid temperature, must not be clamped
  decode(humidity = 0%)  → valid zero — must not be treated as missing
  decode(humidity = 100%) → valid max

Error paths:
  decode(empty buffer)        → returns error, does not crash
  decode(3-byte packet)       → returns error (minimum is 6 bytes)
  decode(bad CRC)             → returns error with CRC mismatch code
  decode(unknown sensor type) → returns error with unknown-type code
```

Write one test per row. Don't skip rows because they seem obvious.

---

## Fixture vs Free Test

Use a `TEST_F` fixture when:
- Setup/teardown is non-trivial and shared across the suite
- The component has meaningful state that must be reset between tests
- You want to express the suite as a coherent set of facts about one class

Use a plain `TEST` when:
- The function is stateless or near-stateless
- Setup is a one-liner local helper
- You are testing a free function, not a class

**Example — stateful component uses a fixture:**

```cpp
class ReadingStoreTest : public ::testing::Test {
protected:
    ReadingStore store;

    void SetUp() override { store.clear(); }

    // Vocabulary helpers live on the fixture, not duplicated in every test
    bool hasReadingFor(SensorId id)  { return store.latest(id).has_value(); }
    float latestTemp(SensorId id)    { return store.latest(id)->temperature_c; }
};

TEST_F(ReadingStoreTest, EmptyStoreHasNoReadings) {
    EXPECT_FALSE(hasReadingFor(SensorId::INDOOR));
}
TEST_F(ReadingStoreTest, StoresFirstReading) {
    store.insert({SensorId::INDOOR, 21.5f, 55.0f, 1013.0f});
    EXPECT_TRUE(hasReadingFor(SensorId::INDOOR));
    EXPECT_FLOAT_EQ(latestTemp(SensorId::INDOOR), 21.5f);
}
TEST_F(ReadingStoreTest, NewerReadingReplacesOlder) {
    store.insert({SensorId::INDOOR, 21.5f, 55.0f, 1013.0f});
    store.insert({SensorId::INDOOR, 22.0f, 54.0f, 1013.0f});
    EXPECT_FLOAT_EQ(latestTemp(SensorId::INDOOR), 22.0f);
}
```

**Example — stateless function uses plain `TEST`:**

```cpp
TEST(TempConversion, FreezingPoint) { EXPECT_FLOAT_EQ(celsius_to_fahrenheit(0.0f),    32.0f);  }
TEST(TempConversion, BoilingPoint)  { EXPECT_FLOAT_EQ(celsius_to_fahrenheit(100.0f),  212.0f); }
TEST(TempConversion, Negative)      { EXPECT_FLOAT_EQ(celsius_to_fahrenheit(-40.0f),  -40.0f); }
```

---

## The Helper Pattern

Every test file that does non-trivial setup defines a file-local helper function. The helper wraps exactly the thing under test and returns only what tests need to inspect. Tests become pure assertions: call helper, check result. If the calling convention changes, you fix the helper — not every test.

**Example — one helper per test file:**

```cpp
// TestPacketBuilder.cpp
static Packet build(float temp_c, float humidity_pct, float pressure_hpa) {
    SensorReading r{temp_c, humidity_pct, pressure_hpa};
    return PacketBuilder().build(r);
}

TEST(PacketBuilder, MinimumLength)      { EXPECT_GE(build(20, 50, 1013).size(), 6u); }
TEST(PacketBuilder, CrcIsLastTwoBytes)  { auto p = build(20, 50, 1013);
                                          EXPECT_EQ(p.crc(), crc16(p.payload())); }
TEST(PacketBuilder, TemperatureEncoded) { EXPECT_EQ(build(-10, 50, 1013).temp_raw(),
                                                    encode_temp(-10.0f)); }
```

```cpp
// TestPacketDecoder.cpp
// Two helpers: one that appends a valid CRC (for content tests),
// one that doesn't (for error path tests).
static DecodeResult decode(std::initializer_list<uint8_t> bytes) {
    Buffer buf(bytes);
    buf.append_crc();
    return PacketDecoder().decode(buf);
}
static DecodeResult decode_raw(std::initializer_list<uint8_t> bytes) {
    return PacketDecoder().decode(Buffer(bytes));
}

TEST(PacketDecoder, DecodesTemperature)     { EXPECT_FLOAT_EQ(decode({0x01,0x09,0xC4}).reading().temperature_c, 25.0f); }
TEST(PacketDecoder, RejectsTruncatedPacket) { EXPECT_FALSE(decode_raw({0x01, 0x02}).ok()); }
TEST(PacketDecoder, RejectsBadCrc)          { EXPECT_EQ(decode_raw({0x01,0x09,0xC4,0xFF,0xFF}).error(),
                                                        DecodeError::CrcMismatch); }
```

```cpp
// TestIntegration.cpp — full pipeline, all options in one function
static SensorReading round_trip(float temp_c, float hum, float pressure,
                                bool fahrenheit = false, bool high_res = false,
                                SensorId id = SensorId::INDOOR) {
    auto packet = PacketBuilder()
                    .set_fahrenheit(fahrenheit)
                    .set_high_res(high_res)
                    .build({temp_c, hum, pressure, id});
    ReadingStore store;
    auto result = PacketDecoder().decode(packet);
    EXPECT_TRUE(result.ok());
    store.insert(result.reading());
    return store.latest(id).value();
}
```

---

## Assertion Strategy

Use `EXPECT_*` by default — the test keeps running and collects all failures. Use `ASSERT_*` only when a failed precondition would crash or make subsequent checks meaningless.

**`EXPECT` when all checks are independent:**

```cpp
TEST(SensorReading, AllFieldsPopulated) {
    SensorReading r = make_reading(21.5f, 55.0f, 1013.25f);
    EXPECT_FLOAT_EQ(r.temperature_c,  21.5f);
    EXPECT_FLOAT_EQ(r.humidity_pct,   55.0f);
    EXPECT_FLOAT_EQ(r.pressure_hpa, 1013.25f);
}
```

**`ASSERT` guards a dereference or array index:**

```cpp
TEST(PacketDecoder, DecodedReadingHasCorrectTemperature) {
    auto result = PacketDecoder().decode(make_temp_packet(21.5f));
    ASSERT_TRUE(result.ok());                              // crashes if !ok without this
    EXPECT_FLOAT_EQ(result.reading().temperature_c, 21.5f);
}

TEST(ReadingStore, ReturnsTwoMostRecentReadings) {
    ReadingStore store;
    store.insert(make_reading(20.0f));
    store.insert(make_reading(21.0f));
    auto recent = store.recent(2);
    ASSERT_EQ(recent.size(), 2u);                          // UB to index without this
    EXPECT_FLOAT_EQ(recent[0].temperature_c, 21.0f);
    EXPECT_FLOAT_EQ(recent[1].temperature_c, 20.0f);
}
```

**Structural invariant loop:**

```cpp
TEST(PacketBuilder, AllSequenceIdsAreUnique) {
    std::set<uint8_t> seen;
    for (float temp = -40.0f; temp <= 85.0f; temp += 5.0f) {
        auto p = build(temp, 50.0f, 1013.0f);
        EXPECT_EQ(seen.count(p.sequence_id()), 0u) << "duplicate id at temp=" << temp;
        seen.insert(p.sequence_id());
    }
}
```

---

## Testing Error Paths

Fatal errors are tested with `EXPECT_EXIT` — assert the exit code and a message substring. Return-value failures are tested by checking the signal separately from any side effect.

**Process-exit on unrecoverable hardware fault:**

```cpp
TEST(SensorReader, ImpossibleTemperatureDies) {
    EXPECT_EXIT(
        SensorReader().process_raw(0xFFFF),   // hardware fault sentinel
        ::testing::ExitedWithCode(1),
        "sensor fault"
    );
}
```

**Return value tested separately from mutation:**

```cpp
// bool PacketDecoder::try_decode(Buffer& buf, SensorReading& out);
// Returns true and populates `out` on success.
// Returns false and leaves `out` unchanged on failure.

TEST(PacketDecoder, ReturnsTrueAndPopulatesOnSuccess) {
    Buffer buf = make_valid_packet(21.5f, 55.0f, 1013.0f);
    SensorReading out{};
    EXPECT_TRUE(PacketDecoder().try_decode(buf, out));   // signal
    EXPECT_FLOAT_EQ(out.temperature_c, 21.5f);          // effect
}

TEST(PacketDecoder, ReturnsFalseAndLeavesOutputUnchangedOnBadCrc) {
    Buffer buf = make_packet_with_bad_crc();
    SensorReading out{};
    out.temperature_c = 99.0f;                               // sentinel value
    EXPECT_FALSE(PacketDecoder().try_decode(buf, out));      // signal
    EXPECT_FLOAT_EQ(out.temperature_c, 99.0f);              // unchanged
}
```

---

## Integration Testing

Integration tests use a single end-to-end harness that hides all pipeline wiring. Tests are organized by **feature category**, not by internal component. Every public feature gets at least one positive test and one negative.

```cpp
// ── Temperature ───────────────────────────────────────────────
TEST(Integration, PositiveTemperatureRoundTrips) {
    EXPECT_FLOAT_EQ(round_trip(21.5f, 50.0f, 1013.0f).temperature_c, 21.5f);
}
TEST(Integration, NegativeTemperatureRoundTrips) {
    EXPECT_FLOAT_EQ(round_trip(-10.0f, 50.0f, 1013.0f).temperature_c, -10.0f);
}
TEST(Integration, FreezePointRoundTrips) {
    EXPECT_FLOAT_EQ(round_trip(0.0f, 50.0f, 1013.0f).temperature_c, 0.0f);
}

// ── Humidity ──────────────────────────────────────────────────
TEST(Integration, HumidityRoundTrips)     { EXPECT_FLOAT_EQ(round_trip(20.0f, 73.0f, 1013.0f).humidity_pct, 73.0f); }
TEST(Integration, ZeroHumidityRoundTrips) { EXPECT_FLOAT_EQ(round_trip(20.0f,  0.0f, 1013.0f).humidity_pct,  0.0f); }

// ── Unit conversion ───────────────────────────────────────────
TEST(Integration, FahrenheitFlagConvertsCorrectly) {
    auto r = round_trip(21.5f, 50.0f, 1013.0f, /*fahrenheit=*/true);
    EXPECT_NEAR(r.temperature_c, 21.5f, 0.1f);
}

// ── Multiple sensors ──────────────────────────────────────────
TEST(Integration, IndoorAndOutdoorStoredSeparately) {
    auto indoor  = round_trip(21.5f, 55.0f, 1013.0f, false, false, SensorId::INDOOR);
    auto outdoor = round_trip( 5.0f, 80.0f, 1008.0f, false, false, SensorId::OUTDOOR);
    EXPECT_FLOAT_EQ(indoor.temperature_c,  21.5f);
    EXPECT_FLOAT_EQ(outdoor.temperature_c,  5.0f);
}

// ── Error conditions ──────────────────────────────────────────
TEST(Integration, CorruptPacketProducesNoReading) {
    Buffer buf = make_valid_packet(20.0f, 50.0f, 1013.0f);
    buf.corrupt_byte(3);
    EXPECT_FALSE(PacketDecoder().decode(buf).ok());
}
```

---

## Generated / Bulk Tests

When a behavior family is well-defined and repetitive, build a case table. Compute expected values with a simple reference function — not by running the system under test.

```cpp
struct TemperatureCase {
    std::string name;
    float input_celsius;
    float expected_celsius;
};

// Reference: plain arithmetic, no PacketBuilder involved
static float reference_round_trip(float c) {
    int16_t raw = static_cast<int16_t>(std::round(c * 10.0f));
    return raw / 10.0f;
}

static std::vector<TemperatureCase> make_temperature_cases() {
    std::vector<TemperatureCase> cases;
    for (float t = -40.0f; t <= 85.0f; t += 0.5f)
        cases.push_back({"temp_" + std::to_string(t), t, reference_round_trip(t)});
    return cases;
}

class GeneratedTempTest : public ::testing::TestWithParam<TemperatureCase> {};

TEST_P(GeneratedTempTest, RoundTripWithinOneTenth) {
    auto& c = GetParam();
    float actual = round_trip(c.input_celsius, 50.0f, 1013.0f).temperature_c;
    EXPECT_NEAR(actual, c.expected_celsius, 0.05f) << c.name;
}

INSTANTIATE_TEST_SUITE_P(TemperatureCases, GeneratedTempTest,
    ::testing::ValuesIn(make_temperature_cases()));
```

Group case tables by behavior family (temperature, humidity, pressure, malformed packets) so a failure in one family doesn't obscure the others.

---

## RAII and Test Isolation

Every resource acquired in a test must be released deterministically. Global state must be saved and restored by the harness, not each individual test.

```cpp
// Simulated BLE radio: opens loopback on construction, closes on scope exit
struct FakeBleChannel {
    FakeBleChannel()  { BleRadio::open_loopback(); }
    ~FakeBleChannel() { BleRadio::close_loopback(); }
};

// Temp file for replaying recorded sensor data, deleted on scope exit
struct SensorDataFile {
    std::string path;
    SensorDataFile(const std::vector<uint8_t>& bytes) {
        char buf[] = "/tmp/sensor_XXXXXX";
        int fd = mkstemp(buf);
        path = buf;
        write(fd, bytes.data(), bytes.size());
        close(fd);
    }
    ~SensorDataFile() { unlink(path.c_str()); }
};
```

```cpp
// Global state saved and restored in the harness — not in each test
static SensorReading round_trip(float temp_c, float hum, float pressure) {
    FakeBleChannel ble;                        // opens and closes automatically

    auto saved_log = g_diagnostic_log_level;
    g_diagnostic_log_level = LogLevel::Off;    // silence hardware noise during tests

    auto packet = PacketBuilder().build({temp_c, hum, pressure});
    auto result = PacketDecoder().decode(packet);
    EXPECT_TRUE(result.ok());

    g_diagnostic_log_level = saved_log;        // always restored, even if EXPECT fails
    return result.reading();
}
```

---

## Naming Conventions

| Level | Convention | Example |
|---|---|---|
| Suite | Component or class name | `SensorReader`, `PacketDecoder`, `Integration` |
| Test | Specific behavior, PascalCase | `NegativeTemperatureRoundTrips`, `RejectsBadCrc` |
| File-local helper | Lowercase verb | `build()`, `decode()`, `round_trip()` |
| Fixture helper | Expressive boolean | `hasReadingFor()`, `latestTemp()` |

Comment every logical group with a visual separator — files stay flat and grep-friendly:

```cpp
// ── Temperature ───────────────────────────────────────────────
// ── Humidity ──────────────────────────────────────────────────
// ── Error conditions ──────────────────────────────────────────
```

---

## What Not to Do

**Don't test implementation details.**
```cpp
// BAD:  EXPECT_EQ(store.internal_cache_.size(), 3u);
// GOOD: EXPECT_FLOAT_EQ(store.latest(SensorId::INDOOR)->temperature_c, 21.5f);
```

**Don't share state between tests.**
```cpp
// BAD:  static ReadingStore store;   // one test's inserts bleed into the next
// GOOD: each test gets its own store through the fixture or harness
```

**Don't name a test with "And" — split it.**
```cpp
// BAD:  TEST(PacketDecoder, DecodesTemperatureAndHumidityAndRejectsBadCrc)
// GOOD: TEST(PacketDecoder, DecodesTemperature)
//       TEST(PacketDecoder, DecodesHumidity)
//       TEST(PacketDecoder, RejectsBadCrc)
```

**Don't generate expected values by running the system under test.**
```cpp
// BAD:  both sides wrong if the encoder has a bug
// GOOD: EXPECT_FLOAT_EQ(round_trip(21.5f, ...).temperature_c, reference_round_trip(21.5f));
```

**Don't assert error paths only by checking nothing crashes.**
```cpp
// BAD:  EXPECT_NO_FATAL_FAILURE(PacketDecoder().decode(corrupt_buf));
// GOOD: EXPECT_EQ(PacketDecoder().decode(corrupt_buf).error(), DecodeError::CrcMismatch);
```

---

## Summary Table

| Question | Answer |
|---|---|
| How many test files? | One per production component + one integration + one bulk/generated |
| What framework? | gtest (preferred); Catch2 fine for smaller projects |
| How large is one test? | One behavior, 3–10 lines of body |
| Where does setup live? | File-local helper function, or fixture `SetUp()` |
| When to use `ASSERT`? | When the next line would crash or be meaningless if the assertion fails |
| How to test error paths? | `EXPECT_EXIT` for fatal exits; return value tested separately from mutation |
| How to handle hardware/I/O? | RAII wrapper; open/close in harness, not in individual tests |
| How to get breadth? | Parameterized case tables with reference-computed expected values |
| What does a test name say? | Exactly the one behavior it proves — readable as a sentence |