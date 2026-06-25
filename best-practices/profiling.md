| Area | What you're hunting | Linux | Windows |
|---|---|---|---|
| **On-CPU hotspots** (sampling) | where cycles go | `perf` + Hotspot/flamegraph | WPR/WPA (ETW) |
| **Exact call graph / counts** | when sampling lies, precise caller-callee | Callgrind + KCachegrind | (none great free — run under WSL Callgrind) |
| **Off-CPU / wait** | slow while CPU idle: IO, locks, sleep | bpftrace `offcputime` (eBPF) | WPA (ETW has wait analysis built in) |
| **Heap / allocations** | leaks, churn, peak, temp allocs | Heaptrack | WPA heap snapshots / UMDH |
| **Heap over time** | growth curve | Massif + massif-visualizer | WPA |
| **Lock contention** | thread serialization | `perf lock` / bpftrace | WPA (ready-thread + wait) |
| **Function-level trace** | full call timeline | uftrace | Tracy (you have it) |
| **Causal** ("what gives real speedup") | rank by actual impact | Coz | — (run under WSL) |
| **System-wide trace** | whole-machine view | Perfetto (you have it) | WPR feeds Perfetto too |

**Practical takeaways:**
- **Linux is the rich side.** The eBPF stack (`bpftrace`, `offcputime`, `funclatency`) is the thing Windows has no clean free equivalent of, and off-CPU is where most "mysteriously slow" bugs hide.
- **Windows = ETW basically does everything.** WPR (record) + WPA (analyze) is one free toolchain covering CPU, wait, heap, locks. Learn it once, it replaces five Linux tools. Orbit is the standalone cross-platform fallback if WPA's UI defeats you.
- **Callgrind and Coz have no real Windows peer** — for those, profile under WSL.

So a clean split: **Linux** → perf + Hotspot (CPU), Heaptrack (mem), bpftrace (off-CPU). **Windows** → WPR/WPA for nearly everything, Tracy for realtime.