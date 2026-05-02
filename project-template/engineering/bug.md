# Bug Tracker

## What this is
Canonical bug and triage log for the project. **This is not a substitute for the issue tracker** — it's a summary and reference for decision-making, release readiness, and historical context.

**Each bug has its own file in [`/bugs/`](./bugs/) directory.** This index provides quick navigation and status overview.

---

## Quick Navigation

| ID | Title | Severity | Status | File |
|----|-------|----------|--------|------|
| BUG-001 | BLE connection drops after 30 seconds on nRF52840 | High | In Progress | [details](./bugs/BUG-001_ble_connection_drop.md) |
| BUG-002 | Memory leak in sensor data streaming path | Critical | Triaged | [details](./bugs/BUG-002_sensor_memory_leak.md) |
| BUG-003 | LED blink pattern inconsistent during boot | Low | Won't Fix | [details](./bugs/BUG-003_led_blink_boot.md) |
| BUG-004 | USB enumeration fails after system resume | High | Fixed | [details](./bugs/BUG-004_usb_resume_failure.md) |

---

## Adding a New Bug

1. Copy the template from [`bugs/TEMPLATE.md`](./bugs/TEMPLATE.md)
2. Name the file: `BUG-XXX_short_description.md` (XXX = next available number)
3. Fill in all fields
4. Add an entry to the table above
5. Update the dashboard counts

---

**Last updated:** 2026-05-02  
**Maintainer:** @engineering-manager