# Safety-Critical Medical C++ — Quick Reference

> Source: *Building Safe and Reliable Surgical Robotics with C++* — Milad Khaledyan & Alex Drew (J&J MedTech), CppCon 2024. Context: a Class C robotically-assisted surgical platform, millions of LOC in C++.

Link: https://youtu.be/Lnr75tbeYyA

---

## 1. Why this matters (the 30-second version)

- **64%** of medical device failures trace to software (FDA MAUDE 2006–2011 study). Software has been the **#1 recall cause for 9 quarters in a row** (Stericycle).
- NSA / CISA / White House say: avoid memory-unsafe languages (C/C++). Reality: you often **can't** — safety-certified RTOS / middleware vendors frequently support **only C++**. So harden it.
- Goal triad: **functional correctness** (verifiable spec, deterministic I/O) + **functional safety** (zero unintended behavior; fail *safe*) + **security**.
- Therac-25 (race condition, radiation overdose) is the cautionary tale that spawned the whole regulatory regime.

---

## 2. Regulatory cheat sheet (know the names)

| Doc | What it is | Key takeaway |
|---|---|---|
| **IEC 62304** | Functional safety lifecycle for medical SW | Says *what*, not *how*. Classifies SW. |
| **Software classes** | A / B / C | A = no injury · B = minor injury · C = serious injury/death |
| **AAMI TIR45** | Agile applied to medical devices | V-model + Agile/Scrum hybrid = industry norm |
| **Principles of SW Validation** | Testing emphasis | Demand **100% coverage**; catch bugs early (exponentially cheaper) |
| **AAMI TIR57 / risk mgmt** | SW failure → risk control → verification | 200+ worked examples (e.g. memory leak → no DMA in critical path → verify via static analysis) |
| **SOUP** | Software of Unknown Provenance | Anything not built in-house: 3rd-party libs, OS, **even the compiler**. Must be tested, documented, validated. |
| **SBOM / CBOM** | (Cyber)Security Bill of Materials | Full inventory: version, author, supply chain. Submitted to FDA. |

> ⚠️ Standards are **necessary but not sufficient**. They say "should" = recommendation, not requirement. Medical guidelines are *less* prescriptive than automotive/avionics. You must do more.
>
> Alarming 2018 Barr Group survey of safety-critical devs: 41% skip regression testing, 33% skip static analysis, 43% don't do code reviews.

---

## 3. The 5 pillars of safer C++

1. **Culture** — every dev asks "could this line hurt a patient?" Write a test per bug.
2. **Architecture** — modular, low coupling, high cohesion → cheaper to swap a component (even to a memory-safe language) later.
3. **Processes** — enforced CI/CD, reviews, quality gates.
4. **Tooling** — compiler hardening, sanitizers, static/dynamic analysis.
5. **Exploration** — go beyond the above to mitigate language risk.

---

## 4. Risk-driven architecture

- A Class C **product** ≠ every component is Class C. Logging can be Class A.
- **Segregate by class and by timing:**

| Tier | Tolerance | Example | Runs on |
|---|---|---|---|
| Non-real-time | no timing guarantee | logging | NRTPC |
| Soft real-time | flexible deadlines | vision overlays | NRTPC / RTOS |
| Firm real-time | few missed deadlines OK | UDP comms | RTOS |
| **Hard real-time** | strict, non-negotiable | control loop, vision | **bare metal / FPGA / embedded / RTOS** |

- Vision needing guaranteed latency **bypasses the OS** (bare metal / FPGA) so it isn't slowed down.
- Physical *and* logical segregation: separate cores / threads so components don't interfere.

---

## 5. Processes & tooling

### Coding standards (IEC 62304 Annex B.5.5)
Medical has **no** dedicated safety-critical coding standard → borrow from others:

- **MISRA C++ 2023** (targets C++17) — *safety critical*, the good one
- **AUTOSAR C++14** — *safety critical* (automotive)
- **JSF++** — *safety critical* (avionics)
- **C++ Core Guidelines / Google Style** — *generic*
- **SEI CERT C++** — *generic + security*
- Keep an **internal standard** to resolve conflicts between these.

### Compiler hardening (do this EARLY)
Use the **OpenSSF Compiler Options Hardening Guide for C/C++**. Disallow compiler extensions. FDA: ship with **zero warnings/errors**.

```
-O2 -Wall -Wextra -Werror -Wformat -Wformat=2 -Wconversion -Wimplicit-fallthrough \
-Werror=format-security \
-U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=3 \
-D_GLIBCXX_ASSERTIONS \
-fstrict-flex-arrays=3 \
-fstack-clash-protection -fstack-protector-strong \
-Wl,-z,nodlopen -Wl,-z,noexecstack \
-Wl,-z,relro -Wl,-z,now \
-Wl,--as-needed -Wl,--no-copy-dt-needed-entries
```
Useful individual flags: `-Wsign-conversion`, `-fno-delete-null-pointer-checks`, `-fno-strict-overflow`, `-fno-strict-aliasing`, `-ftrivial-auto-var-init`.

### Sanitizers
ASan · TSan · LSan · MSan · UBSan · **RTSan** (real-time sanitizer — catches allocations/blocking in RT code; new and ideal here).
- Too slow / risky for production → run in **unit tests** first.
- Then combine **non-conflicting** sanitizers in test runs to catch hard bugs (e.g. race conditions).

### CI/CD pipeline (shift left — limited CD in medical)
Local Dev (rich IDE, AI tools, static analysis, sanitizers) → Pre-commit hooks → Multi-platform build → Multi-compiler build → Test build → ASan → TSan → LSan → UBSan → MSan → RTSan → Quality Gate → Code Review → Merge.

Goals: no new bugs/vulnerabilities, all security hotspots reviewed, limited tech debt/duplication, new code covered by tests.

### Testing (shift left, stress it)
Unit → Integration → E2E pyramid, **all with sanitizers**, plus: simulation, SQA, performance, HW, **fuzzing**, **fault-injection** (FDA-required, with recovery report), stress, reliability, regression.
- **Penetration testing must be 3rd-party / unbiased**, HW + SW.

### Frequent upgrades = a superpower
Migrating off old SOUP (e.g. ancient Boost) gets new features, bug fixes, and **vulnerability patches**. Enabled by modular architecture + good dependency mgmt (e.g. Conan) + solid test framework.

---

## 6. Safety-critical path — coding rules

> **After startup, once instruments are in the patient's body**, in the critical path AVOID:
> - **Unsafe ops:** UB, pointer arithmetic, uninitialized vars, out-of-bound r/w, improper casts, nullptr deref, buffer violations, race conditions.
> - **Non-deterministic behavior:** dynamic memory allocation (DMA), exceptions.
> - **OS blocking calls:** unnecessary locking, I/O.

### Avoid Dynamic Memory Allocation (DMA)
**Why:** non-deterministic timing (syscall may overrun deadline), heap fragmentation, can just run out.

**Spot hidden DMA — red flags:**
- STL type accepts an **allocator** param.
- Method can throw `bad_alloc`.
- Conceptually: unknown/unbounded size at compile time → `vector`, `map`, `set`, `string`; `unique_ptr` to a base class; type erasure.

**Sneaky example:** logging a long static string into `void log(const std::string&)` constructs a temporary `std::string` → allocates. Fix: overload taking `const char*` or better `std::string_view`.

**Catch it:** static asserts, static/dynamic analysis, unit tests. One trick: **overload `new` in a unit test and fail if called** (catches regressions; misses raw `malloc`, awkward with mock frameworks). **RTSan / performance-constraint attributes are better if available.**

**How to write it instead:**

| Approach | Verdict |
|---|---|
| Allocate all at init, then never in the loop | OK but risky — `vector`/`make_unique` sit next to critical code, easy regression |
| Pass STL allocator params (arena allocator) | Disliked — those APIs assume DMA + throw |
| **Stack-friendly containers** | ✅ Preferred |

Stack containers: **Boost** `static_vector` (fixed max), `small_vector` (grows then allocates), `flat_set`, `flat_map`; **STL** `array`, `bitset`, `tuple`; **Chromium** `StackVector`/`StackString` family.

**Prefer concrete over generic.** Instead of `std::array<std::variant<A,B>,2>` + `std::visit`, just declare two concrete stack variables with good names — simpler, no heap, no forced genericity.

### Avoid Exceptions
May allocate, non-deterministic timing. Use instead:
- **Optional:** `std::optional`, `boost::optional`, `absl::optional`
- **Error code:** `std::system_error`, custom
- **Result type:** `std::expected`, `tl::expected`, `absl::Status`, Boost.Outcome

### Avoid Blocking Calls
File I/O, network I/O, mutex acquisition = unbounded waits.

**Bad:**
```cpp
std::mutex m;
std::queue<char> q;
std::lock_guard lock(m);
if (!q.empty()) { /* drain whole queue → unbounded work */ }
```

**Better:**
```cpp
boost::lockfree::spsc_queue<MyType> q(42);   // wait-free, single producer/consumer
if (q.read_available() > 0) { /* ... */ }
```
- Must pick a **max size** at construction — analyze the system to know it, and what to do when full.
- **Bound consumption per step** (e.g. 1 or 10 items) so a flooded queue can't blow your deadline. (A low-risk Class A producer can otherwise push a high-risk Class C component over its limit.)

### `unique_ptr`: default, but not a silver bullet
Use it by default. But `get()` (raw ptr, no ownership) and `release()` (transfers ownership) are unsafe APIs — use **only** when interfacing legacy/3rd-party code:
- Callee takes ownership → `release()`
- Callee just manipulates, no ownership → `get()`

### Lean on the zero-overhead principle
In code review you can reject "I did it this way for performance" — write the **safer, clearer** version and trust the optimizer. Example: hand-rolled loop vs `std::find_if` — `find_if` benchmarks *faster* and has fewer places to introduce bugs. Expressive name = self-documenting.

### Stack overflow (Q&A)
Stack also can't fail. Mitigations: ban large objects (no image data on stack), check empirically, **pre-allocate + OS memory-locking** (lock a region so other processes can't touch it; drastically cuts latency vs default ~8 MB Linux stack).

---

## 7. Trade-offs (there is no free lunch)

- **Performance for safety?** Often you *gain* performance (no DMA, no exceptions are faster). Sometimes you pay (e.g. null-check every tick of a high-freq loop). Performance is subjective — **measure it**. A 250 Hz control loop is fast for some, slow for others.
- **Correctness for safety?** Usually safer = more correct. But: fixing static-analysis findings = more code = possible new bugs; dynamic analysis can change behavior; degraded performance can mean "less correct" to a client depending on it.
- **Flexibility for safety?** Mostly yes you lose some — wrapper types, stricter checks, encapsulation, avoiding "knives" all reduce flexibility. Worth it.
- **Cost for safety?** All this tooling/process costs real engineering hours + infrastructure. Safer C++ is **more expensive**.

---

## 8. Takeaways

- Building safe, complex medical devices is genuinely hard.
- Standards/regulations are **required but not sufficient**.
- Software is *the* key risk and will stay so (esp. with cyber threats).
- **Testing alone isn't enough** — you need fail-safe design.
- Safety in C++ comes from **culture + architecture + tooling + processes**.
- **Everything is a trade-off. There is no simple solution.**
