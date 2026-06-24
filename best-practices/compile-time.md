Samuel Privett discusses how to identify, understand, and mitigate slow C++ build times. Here are the main tips and strategies provided:

link: https://youtu.be/Oih3K-3eZ4Y?si=m1rMcV98Y2qYTN_a

1. Measure and Visualize Your Builds

Profile your builds: Use tools like Ninja's log file (6:02) and Clang's -ftime-trace flag (8:31) to generate data on your compilation performance.
Visualize performance: Load these traces into Perfetto (6:37) or use Clang Build Analyzer (20:21) to identify hotspots, such as files that take the longest to parse, expensive templates, or large header dependencies.

2. Optimize Individual Source Files

Manage includes: Refactor massive headers (13:38). Use Include What You Use (IWYU) (14:47) to ensure you are only including what is strictly necessary, as unnecessary headers still incur a parsing cost.
Templates: Be aware of "template explosions." Consider if you can replace complex template meta-programming with constexpr or consteval (18:28), or re-evaluate your API if you are generating excessive template specializations.

3. Improve Project-Level Structure

Translation Units: Recognize that include and template costs occur per translation unit (22:21). Use pre-compiled headers or modules (23:29) to avoid redundant parsing.
Explicit Dependencies: Explicitly express dependencies in your build system (28:59) to enable better parallelization and avoid bottlenecks.
Include Paths: Limit the number of include paths (-I), as excessive paths can scale poorly and increase file system query overhead (31:07).

4. Higher-Order Solutions

Follow a process: Implement a structured approach (33:25): Measure/Identify → Delete/Refactor → Module/PCH. Only move to the next step if the previous one doesn't solve the issue.
Balance hardware vs. optimization: While you can use more hardware or distributed builds, acknowledge Wirth's Law—software often gets slower faster than hardware becomes faster (37:33). Local optimizations and higher-order solutions should work in harmony (37:54).