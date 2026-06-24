Nic Barker outlines four general rules of thumb for optimizing code for the CPU to improve game performance (21:27):

link: https://youtu.be/WwkuAqObplU?si=1H28sygo2oLlPQqr

- Efficient data layout and storage: Focus on organizing data to be as efficient as possible for the CPU cache. This includes using small data types, storing data by locality of use so that related items are packed together in memory, and minimizing waste in cache lines (21:50-22:23).
- Do work in bulk: Amortize the cost of setup by performing tasks in groups. If you need to do something, try to process an array of things rather than individual items whenever possible (22:25-22:47).
- Do as much ahead of time as possible: If work doesn't need to be done in real-time, pre-compute or "bake" it during initialization or at startup so you don't miss your frame budget (22:48-23:38).
- Utilize parallelism: If you have multiple CPU cores, use them! Aim for independent streams of work that do not require complex synchronization, as having one core pegged at 100% while others are idle is inefficient (23:39-24:45).

---

Jonathan Müller provides several practical tips for writing cache-friendly C++ code to improve performance:

link: https://youtu.be/LHPOAcKqWFc?si=smvTsh5F6TKgjb-1

**Data Structure & Size Optimization:**
* **Minimize type size:** Smaller data types allow more elements to fit into a cache line, reducing cache misses (11:21). Use `short` or `uint8_t` instead of `int` if they are sufficient for your data range (11:46).
* **Reorder struct members:** Manually reorder members in your structs to minimize padding, which effectively reduces the size of your objects (17:21).
* **Leverage pointer alignment:** If your data is guaranteed to be aligned, you can use the unused lower bits of a pointer to store small flags or metadata, though this requires caution (20:15).
* **Use cache-friendly containers:** Default to `std::vector` because it provides contiguous memory layout (26:30). Avoid `std::list`, `std::map`, and `std::unordered_map` (when using node-based implementations) as they involve pointer chasing and waste cache space (26:54-29:26).

**Algorithm & Design Patterns:**
* **Adopt Data-Oriented Design (DOD):** Focus on algorithms that transform data (33:00). Prefer *Structure of Arrays* (SoA) over *Array of Structures* (AoS) when your algorithm only needs specific subsets of data (34:58).
* **Keep code hot-path small:** Minimize the amount of code in hot loops to ensure it fits within the instruction cache (32:42).
* **Avoid branching in hot loops:** If a condition is rarely true, move it out of the main loop to keep the execution flow linear and predictable for the CPU (48:06).

**Multi-threading & Hardware Considerations:**
* **Avoid False Sharing:** Ensure that data modified by different threads resides on different cache lines. If threads modify nearby variables that share a cache line, you will see performance degradation (43:16).
* **Always Benchmark:** Do not blindly apply these optimizations. CPU architectures vary, and what is faster on one might not be on another. Always measure performance before and after any changes (2:07, 12:20, 21:39).
* **Use specific tools:** On Linux, use `perf` to analyze cache misses and understand where your performance bottlenecks actually are (53:35).