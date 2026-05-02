# Architecture Decision Records (ADR)

## What this is

Documented, timestamped, version-controlled decisions that affect the system's architecture, technology choices, patterns, or processes.

**Each ADR has its own file in [`/adr/`](./adr/) directory.** This index provides quick navigation and status overview.

---

## Quick Navigation

| ID | Title | Status | Date | Area | File |
|----|-------|--------|------|------|------|
| ADR-001 | SMF over Manual State Handling | Accepted | 2026-05-02 | State Management | [details](./adr/ADR-001_smf_over_manual.md) |
| ADR-002 | Event-driven Communication via AEM | Proposed | 2026-05-02 | IPC | [details](./adr/ADR-002_event_driven_aem.md) |
| ADR-003 | Hierarchical (HSM) vs Flat State Machines | Accepted | 2026-05-02 | SMF Design | [details](./adr/ADR-003_hsm_vs_flat.md) |
| ADR-004 | Zephyr RTOS Version Upgrade Policy | Deprecated | 2026-04-15 | Toolchain | [details](./adr/ADR-004_zephyr_upgrade_policy.md) |
| ADR-005 | Logging Strategy for Embedded Systems | Accepted | 2026-05-01 | Observability | [details](./adr/ADR-005_logging_strategy.md) |

---

## Quick Rules for ADRs

| Rule | Guideline |
|------|-----------|
| **When to write** | Any non-trivial architectural decision affecting multiple modules, technology stack, patterns, or deployment |
| **Who writes** | The engineer proposing the change (usually with tech lead approval) |
| **Review** | At least one other engineer + tech lead |
| **Format** | See [template](./adr/TEMPLATE.md) |
| **Storage** | `adr/ADR-XXX_short_title.md` |
| **Linking** | Reference ADRs from code comments, PRs, RFCs, and other ADRs |

---

## Adding a New ADR

1. Copy the template from [`adr/TEMPLATE.md`](./adr/TEMPLATE.md)
2. Name the file: `ADR-XXX_short_title.md` (XXX = next available number)
3. Fill in all sections
4. Add an entry to the table above
5. Update the dashboard counts
6. Link from relevant RFCs or code comments

---

## References

- [Michael Nygard's ADR blog post](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [Zephyr SMF Docs](https://docs.zephyrproject.org/latest/services/smf/index.html)
- [RFC Index](./rfc.md)
- [Bug Tracker](./bugs.md)

---

**Last updated:** 2026-05-02  
**ADR count:** 5 (1 proposed, 2 accepted, 1 deprecated, 0 superseded)  
**Maintainer:** @tech-lead