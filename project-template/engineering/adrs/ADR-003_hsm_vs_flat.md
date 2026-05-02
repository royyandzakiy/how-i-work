# ADR-003: Hierarchical (HSM) vs Flat State Machines

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **Date** | 2026-05-02 |
| **Deciders** | @embedded-lead |
| **Related** | [ADR-001](./ADR-001_smf_over_manual.md) |

---

## Context

SMF supports both flat (no parent) and hierarchical (parent with initial child) state machines. Which should be the default for our modules?

**Trade-offs:**
- **Flat:** Simpler, less nesting, no automatic parent propagation, no learning curve
- **Hierarchical:** Code reuse, DRY parent logic (e.g., disconnect handling in parent), but steeper learning curve

**Observed pattern in BLE:** `CONNECTED` parent with children `DATA_TRANSFER`, `SECURE_PAIRING` — all need to react to `DISCONNECT` event. Without hierarchy, each child must re-implement disconnect logic.

---

## Decision

**We will use hierarchical (HSM) when a parent state has behavior that applies to all children, otherwise flat.**

**Guidelines:**

| Use Case | Pattern | Example |
|----------|---------|---------|
| Parent has common entry/exit/run logic | HSM | `CONNECTED` → `DATA_TRANSFER` |
| States share no behavior | Flat | `INIT` → `READY` → `SHUTDOWN` |
| Maximum depth | 3 levels | Root → Parent → Child |
| Composite states | Always specify initial child | `&states[INITIAL_CHILD]` |

**Examples:**

✅ **HSM good:**
```
BLE_CONNECTED
  ├── BLE_DATA_TRANSFER
  └── BLE_SECURE_PAIRING
```

❌ **HSM overkill:**
```
INIT → READY → RUNNING → PROCESSING → COMPLETED  (no shared behavior)
```

✅ **Flat good:**
```
INIT → READY → SHUTDOWN  (no shared inheritance)
```

---

## Consequences

### Positive

- **DRY** — disconnect logic written once in parent
- **Entry/exit automatically chain** — no manual propagation needed
- **Models real-world hierarchy** (connected → transferring data)
- **Reduces bugs** from missing edge cases in child states

### Negative (Costs/Trade-offs)

- Developers must understand parent → child semantics
- Propagation return codes required for parent `_run()` to execute
- Debugging harder (which state's `_run()` is executing?)
- Can lead to over-engineering simple sequences

### Risks

| Risk | Mitigation |
|------|------------|
| Excessive nesting (4+ levels) | Enforce 3-level max in code review; reject PRs with deeper nesting |
| Confusion about when parent vs child runs | Diagram in each module's header; logging with indentation showing current state stack |
| Accidental state leaks (child active without parent) | SMF prevents this — parent always in stack if child active |

---

## Alternatives Considered

### Alternative 1: Flat only for all modules

- **Pros:** Simpler, no propagation confusion, easier debugging
- **Cons:** Duplicated code across states (e.g., same disconnect check in 3 child states)
- **Why rejected:** BLE module already shows duplication; hierarchy measurably reduces bugs

### Alternative 2: Deep hierarchy (5+ levels)

- **Pros:** Maximum code reuse, fine-grained behavior
- **Cons:** Impossible to debug, violates KISS, team cannot reason about state
- **Why rejected:** Over-engineering; 3 levels sufficient for all identified use cases

### Alternative 3: Always HSM (even single states)

- **Pros:** Consistent pattern everywhere
- **Cons:** Unnecessary complexity for simple modules (LED: on/off doesn't need hierarchy)
- **Why rejected:** Adds boilerplate without benefit

---

## Implementation Guidance

### Code Example

**HSM template:**
```c
static const struct smf_state states[] = {
    [PARENT] = SMF_CREATE_STATE(
        parent_entry, parent_run, parent_exit, 
        NULL,                    // no parent
        &states[INITIAL_CHILD]   // initial child
    ),
    [CHILD] = SMF_CREATE_STATE(
        child_entry, child_run, child_exit, 
        &states[PARENT],  // parent pointer
        NULL              // no children
    ),
};
```

**Flat template:**
```c
static const struct smf_state states[] = {
    [STATE_A] = SMF_CREATE_STATE(entry_a, run_a, exit_a, NULL, NULL),
    [STATE_B] = SMF_CREATE_STATE(entry_b, run_b, exit_b, NULL, NULL),
};
```

### Checklist for Engineers

- [ ] Parent has >1 child? Consider HSM
- [ ] Parent logic applies to all children? Use HSM
- [ ] Max depth ≤3? If no, refactor
- [ ] Initial child specified for composite states
- [ ] Logging shows full state path (e.g., `CONNECTED/DATA_TRANSFER`)

---

## Related

- [ADR-001: SMF over Manual](./ADR-001_smf_over_manual.md)
- [SMF HSM Example](https://docs.nordicsemi.com/bundle/ncs-3.1.1/page/zephyr/samples/subsys/smf/hsm_psicc2/README.html)

---

## Notes

**Debugging tip:** Add state name logging that shows full path:

```c
static void log_state_path(struct smf_ctx *ctx) {
    LOG_INF("State: %s", smf_get_current_state_name(ctx));
    if (smf_get_parent_state(ctx)) {
        LOG_INF("  Parent: %s", smf_get_parent_state_name(ctx));
    }
}
```

**Maximum depth decision rationale:** 3 levels (Root → Parent → Child) covers all identified use cases:
- Level 1: Module root (e.g., `BLE`)
- Level 2: Major mode (e.g., `CONNECTED` or `DISCONNECTED`)
- Level 3: Sub-mode (e.g., `DATA_TRANSFER` or `SECURE_PAIRING`)

We have no current need for Level 4. If needed in future, revisit this ADR.

---

**Last updated:** 2026-05-02  
**Updated by:** @embedded-lead