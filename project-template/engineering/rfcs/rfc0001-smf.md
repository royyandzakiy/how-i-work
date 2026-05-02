# RFC: Explore implementation of SMF (State Machine Framework)

## Introduction

### AEM (App Event Manager)

Functions as "nervous" system within the system. Events are generated for the purpose of communicating between modules. As known, after a module receives a certain event that it cares about, it then turns into it's internal thread that processes that event, and might (or might not) then manage said modules internal states. This internal state management currently does not follow any particular framework, just a written in a certain way out of habit. This may lead to inconsistency of implementation among modules causing bad readability and more prone to human error; also unrobustness because only enables flat state machines (non hierarchical).

### SMF (State Machine Framework)

SMF tries to solve this problem by providing a more robust State Machine implementation. It acts mainly as a generic and light weight state machine runner. It will act as the "brain" of a certain module, with it's formal structure. It has the feature of hierarchical structures, enabling the creation of Hierarchical State Machines (HSM). It also has a state entry, run, and exit for each state transition. The way it runs a hierarchical state transition, it acts like a tree node traverser, triggering `exit()` as soon as a child's parent goes "out of scope", and triggering `entry()` as soon as a parent node acting as "gateway" needs to get passed through.

> **Diagram: HSM Tree Traversal — Transition from state `1ba` to state `2aa`**
>
> ```
>                eg: transition from
>                 state 1ba, to state 2aa
>
>        ┌───────────┐                    ┌───────────┐
>        │     1     │ exit() ──────────► │     2     │
>        │           │              entry()           │
>        └───┬───┬───┘                    └─────┬─────┘
>            │   │                              │
>       ┌────┘   └────┐                    entry()
>       ▼             ▼                         ▼
>   ┌───────┐   ┌───────────┐            ┌───────────┐
>   │  1a   │   │    1b     │            │    2a     │
>   │ (grey)│   │  exit()   │            │  entry()  │
>   └───────┘   └───┬───┬───┘            └─────┬─────┘
>                   │   │                       │
>              ┌────┘   └────┐             entry()
>              ▼             ▼                  ▼
>         ┌─────────┐  ┌─────────┐       ┌───────────┐
>         │   1ba   │  │  1bb    │       │    2aa    │
>         │ exit()  │  │ (grey)  │       │  entry()  │
>         └─────────┘  └─────────┘       │   run()   │
>                                        └───────────┘
> ```

---

# Recreating `ble.c`

## Current Implementation

This is a typical BLE module implementation pattern found in many embedded projects.

_Note: Please do not focus on what I have written down here as implementation, focus on the refactoring possibilities and design patterns being used as discourse for decision making._

```c
// a static global state
static enum state_type {
    STATE_BLE_INIT,
    STATE_BLE_IDLE,
    STATE_BLE_ADVERTISING,
    STATE_BLE_CONNECTED,
    STATE_BLE_DATA_TRANSFER,
    STATE_BLE_DISCONNECTED,
    STATE_SHUTDOWN
} state;

static char *state2str(enum state_type state)
{
    switch (state) {
    case STATE_BLE_INIT:
        return "STATE_BLE_INIT";
    case STATE_BLE_IDLE:
        return "STATE_BLE_IDLE";
    case STATE_BLE_ADVERTISING:
        return "STATE_BLE_ADVERTISING";
    case STATE_BLE_CONNECTED:
        return "STATE_BLE_CONNECTED";
    case STATE_BLE_DATA_TRANSFER:
        return "STATE_BLE_DATA_TRANSFER";
    case STATE_BLE_DISCONNECTED:
        return "STATE_BLE_DISCONNECTED";
    case STATE_SHUTDOWN:
        return "STATE_SHUTDOWN";
    default:
        return "Unknown";
    }
}

// setting the static global state
static void state_set(enum state_type new_state)
{
    if (new_state == state) {
        LOG_DBG("State: %s", state2str(state));
        return;
    }

    LOG_DBG("State transition %s --> %s",
        state2str(state),
        state2str(new_state));

    state = new_state;
}

// ...

static void module_thread_fn(void)
{
    int err;
    struct ble_msg_data msg = { 0 };

    ble.self.thread_id = k_current_get();

    err = module_start(&ble.self);
    if (err) {
        LOG_ERR("Failed starting module, error: %d", err);
        SEND_ERROR(ble, BLE_EVT_ERROR, err);
    }

    state_set(STATE_BLE_INIT);

    while (true) {
        module_get_next_msg(&ble.self, &msg);

        switch (ble.state) {
        case STATE_BLE_INIT:
            on_state_init(&msg);
            break;
        case STATE_BLE_IDLE:
        case STATE_BLE_ADVERTISING:
        case STATE_BLE_CONNECTED:
        case STATE_BLE_DATA_TRANSFER:
            message_handler(&msg);
            break;
        case STATE_SHUTDOWN:
            break;
        default:
            LOG_ERR("Unknown state.");
            break;
        }
    }
}

static void message_handler(struct ble_msg_data *msg)
{
    if (IS_EVENT(msg, app, APP_EVT_BLE_START_ADVERTISING)) {
        LOG_DBG("Starting BLE advertising...");
        start_advertising();
        state_set(STATE_BLE_ADVERTISING);
    } else if (IS_EVENT(msg, app, APP_EVT_BLE_STOP_ADVERTISING)) {
        LOG_DBG("Stopping BLE advertising...");
        stop_advertising();
        state_set(STATE_BLE_IDLE);
    } else if (IS_EVENT(msg, app, APP_EVT_BLE_START_DATA_TRANSFER)) {
        LOG_DBG("Starting data transfer...");
        state_set(STATE_BLE_DATA_TRANSFER);
        start_streaming_data();
    } else if (IS_EVENT(msg, app, APP_EVT_BLE_STOP_DATA_TRANSFER)) {
        LOG_DBG("Stopping data transfer...");
        stop_streaming_data();
        state_set(STATE_BLE_CONNECTED);
    } else if (IS_EVENT(msg, app, APP_EVT_USB_CONNECTED)) {
        LOG_DBG("USB connected - shutting down BLE...");
        state_set(STATE_SHUTDOWN);
    }
}
```

## Refactored using SMF

### Substitution of classic State Logic with SMF

- Adding `smf_ctx` & `smf_event` inside `ble_t`
- Implementing `state_type` (in HSM "form") & EVENT BITs for SMF use. Intentionally added indentation of `state_type` to emphasize the parent-child state relationship
- Running `smf_set_state` where appropriate (could be inside a `state_ble_xxxx_yyyy`, or from a callback like `.connected`)
- Utilizing local events (`k_event_wait` with `K_NO_WAIT`) to help `state_ble_xxxx_run` decide what to process. Ends with return `SMF_EVENT_HANDLED`

```c
/* Replace the enum with SMF state objects */
static struct smf_state states[];

/* Event definitions */
#define EVENT_ADV_STOP          BIT(0)
#define EVENT_START_STREAMING   BIT(1)
#define EVENT_STOP_STREAMING    BIT(2)
#define EVENT_USB_CONNECTED     BIT(3)
#define EVENT_SETTINGS_READY    BIT(4)

typedef struct {
    struct smf_ctx ctx;
    struct k_event smf_event;
    struct bt_conn *current_conn;
    bool streaming_active;
    struct bt_le_adv_param adv_param;
    struct bt_data adv_data[2];
    struct k_sem ble_operation_sem;
    struct k_sem connect_sema;
    struct k_sem data_thread_sema;
} ble_t;

static ble_t ble;

// State enumeration
static enum state_type {
    STATE_BLE_INIT,
    STATE_BLE_IDLE,
        STATE_BLE_ADVERTISING,
        STATE_BLE_CONNECTED,
            STATE_BLE_DATA_TRANSFER,
        STATE_BLE_DISCONNECTED,
    STATE_SHUTDOWN
} current_state;

/* State functions */
static void state_ble_init_run(void *o) {
    bt_enable(NULL);
    smf_set_state(SMF_CTX(&ble), &states[STATE_BLE_IDLE]);
}


static void state_ble_idle_run(void *o) {
    // Always transition to DISCONNECTED when IDLE state runs
    smf_set_state(SMF_CTX(&ble), &states[STATE_BLE_DISCONNECTED]);
}

static void state_ble_disconnected_entry(void *o) {
    LOG_INF("Entering BLE_DISCONNECTED");
    start_advertising(); // ensure calls at least once, after initialization
}

static void state_ble_disconnected_run(void *o) {
    ble_t *s = (ble_t *)o;
    
    // Check if we have a connection (transition to CONNECTED)
    if (s->current_conn != NULL) {
        smf_set_state(SMF_CTX(s), &states[STATE_BLE_CONNECTED]);
        return;
    }
    
    // Check for shutdown event
    if (k_event_wait(&s->smf_event, EVENT_USB_CONNECTED, false, K_NO_WAIT)) {
        smf_set_state(SMF_CTX(s), &states[STATE_SHUTDOWN]);
        k_event_clear(&s->smf_event, EVENT_USB_CONNECTED);
    }
}

static void state_ble_disconnected_exit(void *o) {
    LOG_INF("Exiting BLE_DISCONNECTED");
    stop_advertising(); // just do once, when successfully ble connected to expected ble client
}

static void state_ble_connected_run(void *o) {
    ble_t *s = (ble_t *)o;
    
    // Check for disconnection
    if (s->current_conn == NULL) {
        smf_set_state(SMF_CTX(s), &states[STATE_BLE_DISCONNECTED]);
        return;
    }
    
    // Check for start streaming event
    if (k_event_wait(&s->smf_event, EVENT_START_STREAMING, false, K_NO_WAIT)) {
        smf_set_state(SMF_CTX(s), &states[STATE_BLE_DATA_TRANSFER]);
        k_event_clear(&s->smf_event, EVENT_START_STREAMING);
        return;
    }
    
    // Check for USB connected (shutdown)
    if (k_event_wait(&s->smf_event, EVENT_USB_CONNECTED, false, K_NO_WAIT)) {
        if (s->current_conn) {
            bt_conn_disconnect(s->current_conn, BT_HCI_ERR_REMOTE_USER_TERM_CONN);
        }
        smf_set_state(SMF_CTX(s), &states[STATE_SHUTDOWN]);
        k_event_clear(&s->smf_event, EVENT_USB_CONNECTED);
    }
}

static void state_ble_data_transfer_entry(void *o) {
    LOG_INF("Entering BLE_DATA_TRANSFER");
    ble_t *s = (ble_t *)o;
    s->streaming_active = true;
}

static void state_ble_data_transfer_run(void *o) {
    ble_t *s = (ble_t *)o;
    
    // Check for stop streaming event
    if (k_event_wait(&s->smf_event, EVENT_STOP_STREAMING, false, K_NO_WAIT)) {
        smf_set_state(SMF_CTX(s), &states[STATE_BLE_CONNECTED]);
        k_event_clear(&s->smf_event, EVENT_STOP_STREAMING);
        return;
    }
    
    // Check for disconnection
    if (s->current_conn == NULL) {
        smf_set_state(SMF_CTX(s), &states[STATE_BLE_DISCONNECTED]);
        return;
    }
    
    // Check for USB connected (shutdown)
    if (k_event_wait(&s->smf_event, EVENT_USB_CONNECTED, false, K_NO_WAIT)) {
        if (s->current_conn) {
            bt_conn_disconnect(s->current_conn, BT_HCI_ERR_REMOTE_USER_TERM_CONN);
        }
        smf_set_state(SMF_CTX(s), &states[STATE_SHUTDOWN]);
        k_event_clear(&s->smf_event, EVENT_USB_CONNECTED);
        return;
    }
    
    // Stream data if available
    if (s->streaming_active && data_available()) {
        sensor_data_t data = acquire_sensor_data();
        process_and_filter_data(&data);
        send_message_by_ble((uint8_t*)&data, sizeof(data));
    }
}

static void state_ble_data_transfer_exit(void *o) {
    LOG_INF("Exiting BLE_DATA_TRANSFER");
    ble_t *s = (ble_t *)o;
    s->streaming_active = false;
}

static const struct smf_state states[] = {
    [STATE_BLE_INIT] = SMF_CREATE_STATE(NULL, state_ble_init_run, NULL, NULL, NULL),
    [STATE_BLE_IDLE] = SMF_CREATE_STATE(NULL, state_ble_idle_run, NULL, NULL, NULL),
    [STATE_BLE_CONNECTED] = SMF_CREATE_STATE(NULL, state_ble_connected_run, NULL, &states[STATE_BLE_IDLE], NULL),
    [STATE_BLE_DISCONNECTED] = SMF_CREATE_STATE(state_ble_disconnected_entry, state_ble_disconnected_run, state_ble_disconnected_exit, &states[STATE_BLE_IDLE], NULL),
    [STATE_BLE_DATA_TRANSFER] = SMF_CREATE_STATE(state_ble_data_transfer_entry, state_ble_data_transfer_run, state_ble_data_transfer_exit, &states[STATE_BLE_CONNECTED], NULL),
    [STATE_SHUTDOWN] = SMF_CREATE_STATE(state_shutdown_entry, NULL, NULL, NULL, NULL)
};
```

### Add on to BLE Logic

- Calling state change and `event_post`

```c
static void connected(struct bt_conn *conn, uint8_t err)
{
    ble.current_conn = bt_conn_ref(conn);
    smf_set_state(SMF_CTX(&ble.ctx), &states[STATE_BLE_CONNECTED]);
    k_event_post(&ble.smf_event, EVENT_ADV_STOP);
}

static void disconnected(struct bt_conn *conn, uint8_t err)
{
    if (ble.current_conn) {
        bt_conn_unref(ble.current_conn);
        ble.current_conn = NULL;
    }
    smf_set_state(SMF_CTX(&ble.ctx), &states[STATE_BLE_DISCONNECTED]);
}

BT_CONN_CB_DEFINE(conn_callbacks) = {
    .connected          = connected,
    .disconnected       = disconnected,
};
```

### Add on to AEM Logic

- Offloading state logic from `message_handler`, to just calling `k_event_post`. Later will be processed within each `state_ble_xxxx_run`. From an architectural point of view, this means **rather than having all state logic inside a single per module level message_handler, each state_run will be it's own state level message_handler**. This therefore requires using `k_event`. This has its pro cons.
- `STATE_BLE_INIT` is not needed anymore, logic is moved within the `state_ble_init_run`
- `smf_run_state` moved below `message_handler`, in case there be a call for state change, it can then process it properly

```
ble_event_handler --> message_handler --> state_ble_xxx_run
```

```c
static void module_thread_fn(void)
{
    int err;
    struct ble_msg_data msg = { 0 };

    ble.self.thread_id = k_current_get();

    err = module_start(&ble.self);
    if (err) {
        LOG_ERR("Failed starting module, error: %d", err);
        SEND_ERROR(ble, BLE_EVT_ERROR, err);
    }

    smf_set_initial(&ctx, &states[STATE_BLE_INIT]);

    while (true) {
        module_get_next_msg(&ble.self, &msg);

        /* CHANGE: no need for on_state_init. it is assumed to happen automatically during state_ble_init_run()
        switch (ble.state) {
        case STATE_BLE_INIT:
            on_state_init(&msg);
        */
        
        /* Process message in current state context. Usually a request to do state change */
        message_handler(&msg);

        /* SMF handles any requests of state change from external events; Also runs any state_run logic (beware, can be blocking!) */
        smf_run_state(&ctx);
        
        k_sleep(K_MSEC(10)); // Small yield
    }
}

/* CHANGE: Logic moved away from within message_handler, to within the statemachine */
static void message_handler(struct ble_msg_data *msg)
{
    if (IS_EVENT(msg, app, APP_EVT_BLE_START_ADVERTISING)) {
        LOG_DBG("Start advertising event");
        k_event_post(&ble_smf_obj.smf_event, EVENT_START_ADVERTISING);
        
    } else if (IS_EVENT(msg, app, APP_EVT_BLE_STOP_ADVERTISING)) {
        LOG_DBG("Stop advertising event");
        k_event_post(&ble_smf_obj.smf_event, EVENT_STOP_ADVERTISING);
        
    } else if (IS_EVENT(msg, app, APP_EVT_BLE_START_DATA_TRANSFER)) {
        LOG_DBG("Start streaming event");
        k_event_post(&ble_smf_obj.smf_event, EVENT_START_STREAMING);
        
    } else if (IS_EVENT(msg, app, APP_EVT_BLE_STOP_DATA_TRANSFER)) {
        LOG_DBG("Stop streaming event");
        k_event_post(&ble_smf_obj.smf_event, EVENT_STOP_STREAMING);
        
    } else if (IS_EVENT(msg, app, APP_EVT_USB_CONNECTED)) {
        LOG_DBG("USB connected event");
        k_event_post(&ble_smf_obj.smf_event, EVENT_USB_CONNECTED);
    }
    // ... other events
}

static bool ble_event_handler(const struct app_event_header *aeh)
{
    struct ble_msg_data msg = {0};
    bool enqueue_msg = false;

    if (is_app_module_event(aeh)) {
        struct app_module_event *evt = cast_app_module_event(aeh);
        msg.module.app = *evt;
        enqueue_msg = true;
    }

    // ...

    if (enqueue_msg) {
        int err = module_enqueue_msg(&ble.self, &msg);

        if (err) {
            LOG_ERR("Message could not be enqueued");
            SEND_ERROR(ble, BLE_EVT_ERROR, err);
        }
    }
    return false;
}

K_THREAD_DEFINE(ble_module_thread, CONFIG_BLE_THREAD_STACK_SIZE,
        module_thread_fn, NULL, NULL, NULL,
        BLE_HIGH_PRIO, 0, 0);
APP_EVENT_LISTENER(MODULE, ble_event_handler);
```

---

# Composite vs Flat States

Composite states automatically triggers **child_run propagation** — an _**upwards**_ **automatic propagation behavior**, in which if a parent gets set state, assuming it has been created as "composite" with it's chosen child state with initial state active, it will act as if both were called sequentially.

- Entry will happen as such: `composite_parent_entry` → `composite_child_entry`
- Run will happen as such: `composite_child_run` → `composite_parent_run`
- Exit will happen as such: `composite_child_exit` → `composite_parent_exit`

The above flow indicates that logically, if a `composite_parent` is being set as state, the `composite_child` will implicitly be set as "hidden" state, and will be called first for run, then automatically propagates the run call upwards sequentially.

## Example Snippet Demonstrating Both Behaviors

```c
#include <zephyr/kernel.h>
#include <zephyr/smf.h>
#include <zephyr/logging/log.h>

#define MODULE MAIN_MODULE
LOG_MODULE_REGISTER(MODULE, 3);

static const struct smf_state states[];

enum state_id { COMP_PARENT, COMP_CHILD_A, COMP_CHILD_B, FLAT_PARENT, FLAT_CHILD };

struct s_object {
    struct smf_ctx ctx;
} obj;

// ------------ COMP_PARENT ------------ 
static void comp_parent_entry(void *o) { LOG_INF("COMP_PARENT_ENTRY"); }
static void comp_parent_run(void *o) { 
    static int count = 0;
    LOG_INF("COMP_PARENT_RUN - count: %d", ++count);
    if (count == 3) {
        count = 0;
        LOG_INF("*** Transitioning to FLAT_PARENT state ***");
        smf_set_state(SMF_CTX(o), &states[FLAT_PARENT]);
    }
} 
    
static void comp_parent_exit(void *o) { LOG_INF("COMP_PARENT_EXIT"); }

// ------------ COMP_CHILD_A ------------ 
static void comp_child_a_entry(void *o) { LOG_INF("COMP_CHILD_A_ENTRY"); }
static void comp_child_a_run(void *o) { 
    static int count = 0;
    LOG_INF("COMP_CHILD_A_RUN - count: %d", ++count);
    if (count == 3) {
        count = 0;
    }
}
static void comp_child_a_exit(void *o) { LOG_INF("COMP_CHILD_A_EXIT"); }

// ------------ COMP_CHILD_B ------------ 
// this is never called...
static void comp_child_b_entry(void *o) { LOG_INF("COMP_CHILD_B_ENTRY"); }
static void comp_child_b_run(void *o) { LOG_INF("COMP_CHILD_B_RUN"); }
static void comp_child_b_exit(void *o) { LOG_INF("COMP_CHILD_B_EXIT"); }

// ------------ FLAT_PARENT ------------ 
static void flat_parent_entry(void *o) { LOG_INF("FLAT_PARENT_ENTRY"); }
static void flat_parent_run(void *o) { 
    static int count = 0;
    LOG_INF("FLAT_PARENT_RUN - count: %d", ++count);
    if (count == 3) {
        count = 0;
        LOG_INF("*** Transitioning to COMP_PARENT state ***");
        smf_set_state(SMF_CTX(o), &states[COMP_PARENT]);
    }
}
static void flat_parent_exit(void *o) { LOG_INF("FLAT_PARENT_EXIT"); }

// ------------ FLAT_CHILD ------------ 
// this is never called...
static void flat_child_entry(void *o) { LOG_INF("FLAT_CHILD_ENTRY"); }
static void flat_child_run(void *o) { LOG_INF("FLAT_CHILD_RUN"); }
static void flat_child_exit(void *o) { LOG_INF("FLAT_CHILD_EXIT"); }

/* State definitions */
static const struct smf_state states[] = {
    /* COMP_PARENT: True hierarchy with initial substate */
    [COMP_PARENT] = SMF_CREATE_STATE(comp_parent_entry, comp_parent_run, comp_parent_exit, 
                                  NULL, &states[COMP_CHILD_A]),
    [COMP_CHILD_A] = SMF_CREATE_STATE(comp_child_a_entry, comp_child_a_run, comp_child_a_exit, 
                               &states[COMP_PARENT], NULL),
    [COMP_CHILD_B] = SMF_CREATE_STATE(comp_child_b_entry, comp_child_b_run, comp_child_b_exit, 
                               &states[COMP_PARENT], NULL),
    
    /* FLAT_PARENT: Parent relationship but no initial substate */
    [FLAT_PARENT] = SMF_CREATE_STATE(flat_parent_entry, flat_parent_run, flat_parent_exit, 
                             NULL, NULL),
    [FLAT_CHILD] = SMF_CREATE_STATE(flat_child_entry, flat_child_run, flat_child_exit, 
                              &states[FLAT_PARENT], NULL),
};

int main(void)
{
    LOG_INF("=== STARTING SMF DEMO ===");
    LOG_INF("Initial state: COMP_PARENT (true hierarchy)");
    smf_set_initial(SMF_CTX(&obj), &states[COMP_PARENT]);

    while(1) {
        smf_run_state(SMF_CTX(&obj));
        k_msleep(1000);
    }
}
```

## Expected Output

```
*** Using Zephyr OS v4.1.99-ff8f0c579eeb ***
[00:00:00.251,739] <inf> MAIN_MODULE: === STARTING SMF DEMO ===
[00:00:00.251,770] <inf> MAIN_MODULE: Initial state: COMP_PARENT (true hierarchy)
[00:00:00.251,770] <inf> MAIN_MODULE: COMP_PARENT_ENTRY
[00:00:00.251,770] <inf> MAIN_MODULE: COMP_CHILD_A_ENTRY
[00:00:00.251,800] <inf> MAIN_MODULE: COMP_CHILD_A_RUN - count: 1 <<<<< child run
[00:00:00.251,800] <inf> MAIN_MODULE: COMP_PARENT_RUN - count: 1  <<<<< parent run
[00:00:01.251,892] <inf> MAIN_MODULE: COMP_CHILD_A_RUN - count: 2
[00:00:01.251,922] <inf> MAIN_MODULE: COMP_PARENT_RUN - count: 2
[00:00:02.252,075] <inf> MAIN_MODULE: COMP_CHILD_A_RUN - count: 3
[00:00:02.252,105] <inf> MAIN_MODULE: COMP_PARENT_RUN - count: 3
[00:00:02.252,105] <inf> MAIN_MODULE: *** Transitioning to FLAT_PARENT state ***
[00:00:02.252,105] <inf> MAIN_MODULE: COMP_CHILD_A_EXIT
[00:00:02.252,105] <inf> MAIN_MODULE: COMP_PARENT_EXIT
[00:00:02.252,136] <inf> MAIN_MODULE: FLAT_PARENT_ENTRY
[00:00:03.252,227] <inf> MAIN_MODULE: FLAT_PARENT_RUN - count: 1 <<<<< a non composite state does not call it's child
[00:00:04.252,410] <inf> MAIN_MODULE: FLAT_PARENT_RUN - count: 2
[00:00:05.252,532] <inf> MAIN_MODULE: FLAT_PARENT_RUN - count: 3
[00:00:05.252,563] <inf> MAIN_MODULE: *** Transitioning to COMP_PARENT state ***
[00:00:05.252,563] <inf> MAIN_MODULE: FLAT_PARENT_EXIT
[00:00:05.252,563] <inf> MAIN_MODULE: COMP_PARENT_ENTRY
[00:00:05.252,593] <inf> MAIN_MODULE: COMP_CHILD_A_ENTRY
[00:00:06.252,716] <inf> MAIN_MODULE: COMP_CHILD_A_RUN - count: 1
[00:00:06.252,746] <inf> MAIN_MODULE: COMP_PARENT_RUN - count: 1
[00:00:07.252,838] <inf> MAIN_MODULE: COMP_CHILD_A_RUN - count: 2
[00:00:07.252,868] <inf> MAIN_MODULE: COMP_PARENT_RUN - count: 2
[00:00:08.253,021] <inf> MAIN_MODULE: COMP_CHILD_A_RUN - count: 3
[00:00:08.253,051] <inf> MAIN_MODULE: COMP_PARENT_RUN - count: 3
[00:00:08.253,051] <inf> MAIN_MODULE: *** Transitioning to FLAT_PARENT state ***
```

---

# Feature in ncs-v3.2.0: `smf_state_result`

In this newer ncs version `3.2.0` (or to be precise, the `zephyr v4.2`), the `enum smf_state_result funct(void *obj)` is introduced as an option of a state function alongside with the traditional `void funct(void *obj)`. It enables a return of type enum `SMF_EVENT_PROPAGATE` and `SMF_EVENT_HANDLED` which calls the parent_state_run of a child_state.

> **Zephyr v4.2 Migration Guide — State Machine Framework changes:**
>
> - `smf_set_handled()` has been removed.
> - State run actions now return an `smf_state_result` value instead of void, and the return code determines if the event is propagated to parent run actions or has been handled. A run action that handles the event completely should return `SMF_EVENT_HANDLED`, and run actions that propagate handling to parent states should return `SMF_EVENT_PROPAGATE`.
> - Flat state machines ignore the return value; returning `SMF_EVENT_HANDLED` would be the most technically accurate response.

**Source:**
- [Zephyr v4.2 Migration Guide — State Machine Framework](https://docs.zephyrproject.org/latest/releases/migration-guide-4.2.html#state-machine-framework)
- [State Machine Framework (ncs v3.2.0-preview3)](https://docs.nordicsemi.com/bundle/ncs-3.2.0-preview3/page/zephyr/services/smf/index.html)

### Enables Back-Propagation

Another feature that can be used is event propagating. Assuming the state `2aa` is triggered, after it runs, it can call to run its parent `2a` and grand-parent `2` to also run, **while still maintaining `2aa` as the active state**. This can be particularly useful if `2aa` is a specific running event, for instance `state_ble_sending_streaming_data`. The parent might be `ble_connected`. Therefore if there be any event to stop the streaming data or if there be a disconnect, the `state_ble_connected` can process it, and manipulate the `state_ble_sending_streaming_data` as needed.

> **Diagram: Event Run Propagation — `SMF_EVENT_PROPAGATE` vs `SMF_EVENT_HANDLED`**
>
> ```
>     ┌──────────────────────────────────────┐
>     │  trigger run() of parent right       │
>     │  after run() of child                │
>     └──────────────────────────────────────┘
>
>     SMF_EVENT_PROPAGATE (left)       SMF_EVENT_HANDLED (right)
>
>     ┌─────────┐                      ┌─────────┐
>     │    2    │                      │    2    │
>     │  run()  │  ◄── propagated      │ (grey)  │  ← not reached
>     └────┬────┘                      └────┬────┘
>          │                                │
>          ▲ return SMF_EVENT_PROPAGATE      │
>          │                                │
>     ┌────┴────┐                      ┌────┴────┐
>     │   2a   │                      │   2a   │
>     │  run()  │  ◄── propagated      │ (grey)  │  ← not reached
>     └────┬────┘                      └────┬────┘
>          │                                │
>          ▲ return SMF_EVENT_PROPAGATE      │
>          │                                │
>     ┌────┴────┐                      ┌────┴────┐
>     │   2aa  │                      │   2aa  │
>     │  run()  │                      │  run()  │
>     └─────────┘                      └─────────┘
>      return SMF_EVENT_PROPAGATE       return SMF_EVENT_HANDLED
> ```

The below code properly utilizes `SMF_EVENT_PROPAGATE` (eg: in the state of `state_ble_data_transfer_run` but would want it's parent state `state_ble_connected_run` to still process events). This triggers **event run propagation**, an _**upwards**_ propagation that causes `parent_run` to run. Event propagation is a **manually set propagation behavior** through return of `smf_state_result` by child state `child_run`, that will be processed by it's parent state, triggering `parent_run`.

```c
/* Replace the enum with SMF state objects */
static struct smf_state states[];

/* Event definitions */
#define EVENT_ADV_STOP          BIT(0)
#define EVENT_START_STREAMING   BIT(1)
#define EVENT_STOP_STREAMING    BIT(2)
#define EVENT_USB_CONNECTED     BIT(3)
#define EVENT_SETTINGS_READY    BIT(4)

typedef struct {
    /* This must be first */
    struct smf_ctx ctx;

    /* Events */
    struct k_event smf_event;
    
    /* Your existing BLE data */
    struct bt_conn *current_conn;
    bool streaming_active;
    struct bt_le_adv_param adv_param;
    struct bt_data adv_data[2];
    struct k_sem ble_operation_sem;
    struct k_sem connect_sema;
    struct k_sem data_thread_sema;
} ble_t;

static ble_t ble;

// a static global state, indentation just to help emphasize parent-child relationship
static enum state_type {
    STATE_BLE_INIT,
    STATE_BLE_IDLE,
        STATE_BLE_ADVERTISING,
        STATE_BLE_CONNECTED,
            STATE_BLE_DATA_TRANSFER,
        STATE_BLE_DISCONNECTED,
    STATE_SHUTDOWN
} states;

/* State declarations */
static enum smf_state_result state_ble_init_run(void *o) {
    bt_enable(NULL);
    smf_set_state(SMF_CTX(&ble.ctx), &states[STATE_BLE_IDLE]);
    return SMF_EVENT_HANDLED; // Event consumed, no parent propagation
}

static enum smf_state_result state_ble_idle_run(void *o) {
    smf_set_state(SMF_CTX(&ble.ctx), &states[STATE_BLE_DISCONNECTED]);
    return SMF_EVENT_HANDLED;
}

static enum smf_state_result state_ble_disconnected_run(void *o) {
    start_advertising();
    return SMF_EVENT_HANDLED;
}

static enum smf_state_result state_ble_connected_run(void *o) {
    ble_t *s = (ble_t *)o;
    uint32_t events = k_event_wait(&s->smf_event, 
                         EVENT_START_STREAMING | EVENT_STOP_STREAMING | EVENT_USB_CONNECTED,
                         true, K_NO_WAIT);
    
    if (events & EVENT_START_STREAMING) {
        smf_set_state(SMF_CTX(s), &states[STATE_BLE_DATA_TRANSFER]);
        k_event_clear(&s->smf_event, EVENT_START_STREAMING);
        return SMF_EVENT_HANDLED; // Event handled, stop propagation
    }
    
    if (events & EVENT_USB_CONNECTED) {
        if (s->current_conn) {
            bt_conn_disconnect(s->current_conn, BT_HCI_ERR_REMOTE_USER_TERM_CONN);
        }
        k_event_clear(&s->smf_event, EVENT_USB_CONNECTED);
        return SMF_EVENT_HANDLED;
    }
    
    // No events to handle - let parent state (STATE_BLE_IDLE) run if needed
    return SMF_EVENT_PROPAGATE;
}

static enum smf_state_result state_ble_data_transfer_run(void *o) {
    ble_t *s = (ble_t *)o;
    
    // Check for stop event first
    if (k_event_wait(&s->smf_event, EVENT_STOP_STREAMING, true, K_NO_WAIT)) {
        smf_set_state(SMF_CTX(s), &states[STATE_BLE_CONNECTED]);
        k_event_clear(&s->smf_event, EVENT_STOP_STREAMING);
        return SMF_EVENT_HANDLED;
    }
    
    if (s->streaming_active && data_available()) {
        sensor_data_t data = acquire_sensor_data();
        process_and_filter_data(&data);
        send_message_by_ble((uint8_t*)&data, sizeof(data));
    }
    
    return SMF_EVENT_PROPAGATE; // Let parent (CONNECTED) handle other events
}

static enum smf_state_result state_shutdown_run(void *o) {
    return SMF_EVENT_HANDLED;
}

// ... similar for other states

/* State table - replaces manual state handling */
static const struct smf_state states[] = {
    [STATE_BLE_INIT] = SMF_CREATE_STATE(NULL, state_ble_init_run, NULL, NULL, NULL),
    [STATE_BLE_IDLE] = SMF_CREATE_STATE(NULL, state_ble_idle_run, NULL, NULL, NULL),
    [STATE_BLE_CONNECTED] = SMF_CREATE_STATE(NULL, state_ble_connected_run, NULL, &states[STATE_BLE_IDLE], NULL),
    [STATE_BLE_DISCONNECTED] = SMF_CREATE_STATE(NULL, state_ble_disconnected_run, NULL, &states[STATE_BLE_IDLE], NULL),
    [STATE_BLE_DATA_TRANSFER] = SMF_CREATE_STATE(NULL, state_ble_data_transfer_run, NULL, &states[STATE_BLE_CONNECTED], NULL),
    [STATE_SHUTDOWN] = SMF_CREATE_STATE(NULL, state_shutdown_run, NULL, NULL, NULL)
};
```

---

# References

- SMF Concept (ncs v3.1.1): [https://docs.nordicsemi.com/bundle/ncs-3.1.1/page/zephyr/services/smf/index.html](https://docs.nordicsemi.com/bundle/ncs-3.1.1/page/zephyr/services/smf/index.html)
- SMF API Ref (ncs v3.1.1): [https://docs.nordicsemi.com/bundle/ncs-3.1.1/page/zephyr/doxygen/html/group__smf.html](https://docs.nordicsemi.com/bundle/ncs-3.1.1/page/zephyr/doxygen/html/group__smf.html)
- Example Flat Calculator: [https://docs.nordicsemi.com/bundle/ncs-3.1.1/page/zephyr/samples/subsys/smf/smf_calculator/README.html#smf_calculator](https://docs.nordicsemi.com/bundle/ncs-3.1.1/page/zephyr/samples/subsys/smf/smf_calculator/README.html#smf_calculator)
- Example HSM: [https://docs.nordicsemi.com/bundle/ncs-3.1.1/page/zephyr/samples/subsys/smf/hsm_psicc2/README.html#smf_hsm_psicc2](https://docs.nordicsemi.com/bundle/ncs-3.1.1/page/zephyr/samples/subsys/smf/hsm_psicc2/README.html#smf_hsm_psicc2)
- Example HSM Asset Tracker: [https://github.com/nrfconnect/Asset-Tracker-Template](https://github.com/nrfconnect/Asset-Tracker-Template)