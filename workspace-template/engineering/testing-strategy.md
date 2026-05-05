## Test System Architecture
- **startegy**: use dual device HIL rig
- **system under test**: mcu devboard running fw (release/debug/test)
- **hil tester**: high performant devboard to simulate physical environment & capture signals from devboard
	- **gpio interfacing**: captures pump pwm signal
	- **power control**: relay to the SUT to automate power cycle
- **orchestrator**: PC running pytest communicating with both boards via serial and parses logs

## Test Strategy
- **end-to-end**: release fw; testing end to end scenarios of happy and edge cases (rtc persistence, power on reset behavior). some tests can be automated by the hil tester, some tests is just faster to be done manually, especially functions still in development (not production verified yet)
- **trace based**: debug fw; using thread trace to detect thread starvation, timing issue (jitter)
- **component level**: test fw; tests each individual components thoroughly via shell commands, ensuring rigour testing of each
- **functional (logic)**: simulation target; unit testing all components, running on a simulation target.

## Test Scenarios
- ### scenario #1
	- **what**: power resilience & rtc initialization
	- **risk**: high (overdosing dut to inconsistent state / timing from power cycle)
	- **setup**: HIL tester toggles power on SUT with random intervals, some small, some large (5ms to 5s)
	- metrics:
		- **T_init**: time from power on to STATE_READY
		- RTC_sync offset between SUT internal clock and HIL tester clock after reboot
	- acceptance criteria:
		- **T_init**: ≤ 500ms
		- post reboot state must resume the last recorded dose (persistence)
		- **pass**: T_init ≤ 500ms, state == STATE_RESUMED, RTC_Sync offset < 5ms
- ### scenario #1.1
	- **what**: power supply noise / brownout recovery. not just on/off, but slow voltage ramp
	- **test**: hil tester control programmable power supply to run different voltage 3.3v → 2.7v → 3.3v repeated with random intervals and values
	- **pass/fail**: device stay operational or resets cleanly into safe state. no partial operation and corrupt states
- ### scenario #2
	- **what**: concurrency & timing stability
	- **risk**: high (thread starvation may affect pump periods)
	- **setup**: SUT run pump, HIL tester connect with the SUT pump IO while HIL tester floods serial events in a high frequency (command requests, reconnection)
	- metrics:
		- **jitter**: variance in pump signal period
		- **latency**: delay between command sent and pump motor signal
	- acceptance criteria:
		- latency ≤ 50ms
		- **jitter**: ≤ 3ms (standard deviation)
		- **pass**: no watchdog reset during 1 hr stress test; latency & jitter values correct.
- ### scenario #2.1
	- **what**: worst case timing interrupt storm. interrupts arrive at exact worst moment (during critical section of pump control)
	- **test**: hil tester generate back to back interrupt (GPIO edge) at 1us interval for 1s, synchronized to pump pwm edge
	- **pass/fail**: no missed pump cycle, no watchdog reset, thread starvation <1ms
- ### scenario #3
	- **what**: robust reconnection (wireless)
	- **risk**: medium (cpu spike during ble handshake causing pump drift)
	- **setup**: HIL tester initiates/drops BLE connection every 10 secs while monitoring SUT CPU Load via logs
	- metrics:
		- **CPU Load**: peak utilization during reconnection handshake
		- **pump cycle miss**: any missed pump cycles during reconnection
		- **pump event duration**: duration of a pump thread event, capture using systemview
	- **acceptance crit**:
		- pump event duration remains ≤ 3ms during entire handshake
		- system gracefully stop retrying after X fails
		- **pass**: pump cycle miss == 0
- ### scenario #3.1
	- **what**: resource exhaution, stack overflow, heap exhaution, watch dog timer near expiry
	- **test**: run device for 72 hrs with maximum logging. hil tester sends maximum rate commands while injecting BLE reconnections. monitor for unexpected reset or missed pump cycles after 48 hrs
	- **pass/fail**: no reset, pump jitter <= 3ms

## Mission Critical Failure Modes & Safety Testing
proving incorrect behavior is impossible / handled gracefully & safely

- ### failure mode #1
	- **what**: sensor / component failure
	- **risk**: pump contnue deliver wrong dose due to bad sensor data
	- **test setup**: il tester physically injects fault into sensor line (open circuit, short to ground, out of range voltage input)
	- **expected behavior**: device must detect fault and transition to STATE FAULT SAFE within 10ms, stopping pump
	- **pass/fail**: pump stops. logged error code. no automatic retry without manual reset
- ### failure mode #2
	- **what**: rtc corruption & invalid persistent state (after reset)
	- **risk**: on power up, if RTC or stored stat corrupt, device might default to start pumping now, possibly causing overdose
	- **test setup**: HIL tester corrupt RTC memory or stored dose state via power glitch or debug shell before reboot
	- **expected behavior**: device detect invalid state and enter STATE SAFE HAULT, not resuming pump
	- **pass/fail**: device halts, show error, no pump
- ### failure mode #3
	- **what**: safety transition latency (normal → stop alarm → safe). slow transition during alarm conditions may deliver unexpected extra dose
	- **risk**: hil tester force alarm condition (over pressure sensor, sensor not detected). measure time from fault injection to pump stop
	- **expected behavior**: transition latency ≤ 10ms from fault to pump signal stop
	- **pass/fail**: measured latency ≤ 10ms using oscilloscope / logic analyzier on SUT