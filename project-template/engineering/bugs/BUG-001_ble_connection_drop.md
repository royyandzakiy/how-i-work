# BUG-001: BLE connection drops after 30 seconds on nRF52840

| Field           | Value                                           |
|----------------|-------------------------------------------------|
| **Severity**    | High                                            |
| **Discovered**  | 2026-05-01                                      |
| **Component**   | `ble.c` - connection management                 |
| **Owner**       | @firmware-team                                  |
| **Status**      | In Progress                                     |
| **Issue Link**  | [JIRA: BLE-123](https://jira.example.com/BLE-123) |
| **Fixed in**    | —                                               |

---

## Reproduction Steps

1. Power on device with BLE enabled
2. Connect from mobile app (iOS or Android)
3. Observe connection
4. Wait 30 seconds
5. Observe connection drop
6. Device re-enters advertising state

---

## Expected Behavior

Connection remains stable until explicit disconnect from either side

---

## Actual Behavior

Connection drops exactly at 30 seconds automatically

---

## Impact

- Data transfer sessions cannot complete
- User must reconnect repeatedly
- Streaming applications fail
- Affects all nRF52840 units (100% reproducible)

---

## Root Cause (if known)

Connection supervision timeout misconfigured — using default 30s instead of application-required 5s.

In `ble.c`:
```c
// Current (incorrect):
static struct bt_conn_param conn_param = {
    .interval_min = 15,
    .interval_max = 30,
    .latency = 0,
    .timeout = 300,  // 300 * 10ms = 3000ms? Actually this is 30 seconds
};

// Should be:
static struct bt_conn_param conn_param = {
    .interval_min = 15,
    .interval_max = 30,
    .latency = 0,
    .timeout = 50,   // 50 * 10ms = 500ms for aggressive supervision
};
```

Note: `timeout` is in units of 10ms. Value 300 = 3000ms (30 seconds).

---

## Investigation Notes

### 2026-05-02
- Captured HCI logs showing disconnect reason: `BT_HCI_ERR_CONN_TIMEOUT` (0x08)
- Verified supervision timeout value using `bt_conn_get_param()`
- Found default timeout value 300 instead of application-required 50
- Does NOT reproduce on nRF5340 — has different default configuration

### 2026-05-01
- Initial bug report from QA
- Reproduced on 3 different nRF52840 units
- Not reproducible on development board with debugger attached

---

## Workaround (if any)

None — device must be reconnected manually every 30 seconds

---

## Fix (if completed)

Not yet fixed — in progress

### Proposed Implementation
Change `timeout` parameter from 300 to 50 in connection parameters

### Verification (to do)
1. Apply fix
2. Flash to nRF52840
3. Connect and monitor for 5 minutes
4. Verify no disconnection

---

## Related Bugs

- None known

---

## Notes

Occurs on all nRF52840 units; does not reproduce on nRF5340 (different BLE controller). May also affect nRF52832.

HCI log snippet:
```
< 0x08 0x00 0x00 0x00 0x08 0x00 0x00 0x00  // Disconnect with timeout reason
```

---

**Last updated:** 2026-05-02  
**Updated by:** @firmware-lead
```