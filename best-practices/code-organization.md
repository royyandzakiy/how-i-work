In the software design world, how you slice up your code *inside* `src/` generally boils down to three main vocabulary terms:

---

## 1. Package-by-Layer (Horizontal Slicing)

This is what you mean when you use folders like `managers/`, `business/`, `services/`, `engine/`, or `hardware/`.

* **The Term:** **Layered Pattern** or **Technical-Role Grouping**.
* **The Concept:** You group files by what they *are* from a technical standpoint. All objects that manage things go into `managers/`, and all raw business logic formulas go into `business/`.
* **C++ Context:** This is common in smaller systems or when working close to hardware (e.g., splitting your firmware into `drivers/`, `middleware/`, and `application/`). It enforces clean horizontal boundaries—dependencies only flow downward.

---

## 2. Package-by-Feature (Vertical Slicing)

This is when you group code by domain capability instead of technical role.

* **The Term:** **Feature Folders**, **Domain-Driven Layout**, or **Component-Based Structuring**.
* **The Concept:** Instead of a giant `managers/` folder, you might have `telemetry/`, `ui/`, and `motor_control/`. Inside the `telemetry/` folder, you will find its specific manager, its specific business logic, and its data parsing files sitting side-by-side.
* **C++ Context:** Essential for medium-to-large projects. It prevents your folders from turning into dumping grounds. If you need to delete or rewrite how the "telemetry" works, you just delete that one folder rather than hunting down files in five different technical layers.

---

## 3. Core vs. App Separation (Subsystem Decoupling)

This is when you have folders explicitly named `core/` and `app/` inside `src/`.

* **The Term:** **Core/Application Split** or **Engine/Client Architecture**.
* **The Concept:**
* `core/` contains pure, reusable, platform-agnostic business logic or your underlying engine. It doesn't care *how* it's being used; it just executes the primary rules of your system.
* `app/` (or `main/`, `cli/`, `gui/`) acts as the orchestrator. It handles the entry point, wires up the dependencies, configurations, and handles the specific runtime environment.


* **C++ Context:** Highly practical if you plan to target multiple platforms or use the core logic across different executables. For example, your `core/` compiles into a static library (`.a` or `.lib`), and your `app/` links against it to build the final binary.

---

### Quick Comparison

| Pattern Name | Folders Inside `src/` look like... | Best For |
| --- | --- | --- |
| **Package-by-Layer** | `drivers/`, `managers/`, `business/` | Firmware, strict hardware abstractions, small utilities |
| **Package-by-Feature** | `networking/`, `graphics/`, `audio/` | Large-scale codebases, team-isolated modules |
| **Core/App Split** | `core/`, `app/` (or `engine/`, `client/`) | Code intended for reuse, multi-platform applications |

---

## Cherno

App, Core layout

https://github.com/TheCherno/Architecture/tree/main