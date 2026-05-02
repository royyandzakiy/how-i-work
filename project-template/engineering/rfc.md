# Request for Comments (RFCs)

## What this is

Central index and tracking for all RFCs (Request for Comments) — proposals for significant changes, new features, or architectural decisions that require team discussion and approval before implementation.

**Each RFC has its own file in [`/rfcs/`](./rfcs/) directory.** This index provides quick navigation, status tracking, and decision history.

---

## Quick Navigation

| ID | Title | Status | Author | Date | File |
|----|-------|--------|--------|------|------|
| RFC-001 | SMF Implementation for BLE Module | Accepted | @embedded-lead | 2026-05-02 | [details](./rfcs/RFC-001_smf_implementation.md) |
| RFC-002 | USB Power Management Strategy | Proposed | @usb-team | 2026-05-01 | [details](./rfcs/RFC-002_usb_power_mgmt.md) |
| RFC-003 | OTA Firmware Update Mechanism | In Review | @firmware-team | 2026-04-28 | [details](./rfcs/RFC-003_ota_updates.md) |
| RFC-004 | Logging System Redesign | Accepted | @system-architect | 2026-04-20 | [details](./rfcs/RFC-004_logging_redesign.md) |
| RFC-005 | Sensor Sampling Rate Configuration | Draft | @sensors-team | 2026-05-03 | [details](./rfcs/RFC-005_sensor_sampling.md) |
| RFC-006 | BLE Security Bonding Strategy | Rejected | @ble-team | 2026-04-15 | [details](./rfcs/RFC-006_ble_bonding.md) |

---

## Quick Rules for RFCs

| Rule | Guideline |
|------|-----------|
| **When to write** | Any change >1 sprint, affecting multiple modules, or introducing new patterns/technologies |
| **Who writes** | The engineer proposing the change (with input from affected teams) |
| **Review** | At least 2 engineers + tech lead; open for comments for minimum 3 business days |
| **Format** | See [template](./rfcs/TEMPLATE.md) |
| **Storage** | `rfcs/RFC-XXX_short_title.md` |
| **Linking** | Reference ADRs from accepted RFCs; link PRs back to RFCs |

---

## RFC Lifecycle

```
DRAFT → IN REVIEW → PROPOSED → ACCEPTED/REJECTED → IMPLEMENTED/DEPRECATED
```

1. **DRAFT** — Author begins writing, not yet ready for feedback
2. **IN REVIEW** — Open for comments (min 3 days); label with review deadline
3. **PROPOSED** — Ready for decision; tech lead schedules discussion
4. **ACCEPTED** — Approved; assign owner and target completion date
5. **REJECTED** — Not approved; document rationale
6. **IMPLEMENTED** — Code merged, feature complete
7. **DEPRECATED** — Superseded by newer RFC

---

## Adding a New RFC

1. Copy the template from [`rfcs/TEMPLATE.md`](./rfcs/TEMPLATE.md)
2. Name the file: `RFC-XXX_short_title.md` (XXX = next available number)
3. Fill in all sections (DRAFT status)
4. Add entry to table above
5. Create PR for the RFC
6. Discuss in Slack/email and annotate RFC with feedback
7. Update status when resolved

---

## RFC Template Sections

| Section | Required | Description |
|---------|----------|-------------|
| Problem Statement | ✅ | What problem are we solving? |
| Proposed Solution | ✅ | What change is proposed? |
| Alternatives Considered | ✅ | What else did we think of? |
| Rollout Plan | ✅ | How will we implement? |
| Rollback Plan | ✅ | How do we undo if it fails? |
| Success Metrics | ✅ | How do we measure success? |
| Owners | ✅ | Who's responsible? |
| Timeline | ✅ | When will this happen? |
| Open Questions | Optional | What still needs discussion? |

---

## References

- [ADR Index](./adr.md)
- [Bug Tracker](./bugs.md)
- [RFC Best Practices](https://about.gitlab.com/handbook/engineering/rfcs/)
- [Zephyr RFC Process](https://docs.zephyrproject.org/latest/contribute/index.html)

---

**Last updated:** 2026-05-03  
**RFC count:** 6 (2 active, 2 accepted, 1 rejected, 1 draft)  
**Maintainer:** @tech-lead