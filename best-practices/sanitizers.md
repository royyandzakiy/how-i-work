# How I Use Sanitizers

Sanitizers are the cheapest way to catch the bugs that hurt most in C++ — memory corruption,
use-after-free, leaks, data races, undefined behavior. But "turn them all on" is the wrong
instinct. Here's how I actually run them.

## Two things to get straight first

1. **ASan, TSan, and MSan are mutually exclusive.** You cannot put them in one binary — the
   compiler rejects it. So "max sanitizers in one build" doesn't exist; it's always one of them
   (plus UBSan, which composes with any).
2. **They're complementary, not ranked.** Each catches a different bug class. So the strategy
   isn't "pick the best one" — it's "match each one to how often I'm willing to pay for it."

The costs differ a lot: ASan ~2×, MSan ~3×, **TSan ~5–15×**. That alone kills any
"always run everything" idea.

## The tiered workflow

**Inner loop (all day): ASan + UBSan + LSan.**
The sweet spot — it catches the highest-value bugs immediately at ~2× slowdown, and LSan rides
along with ASan for free. I develop under this by default. If memory bugs exist, I find them the
moment a test runs, not three weeks later in production.

**Situational (when the code calls for it):**
- **TSan** — only when I'm touching threads/concurrency. Separate build, much slower, can't coexist
  with ASan. Not an every-build thing.
- **MSan** — rarely locally. It needs an instrumented standard library to avoid false positives, so
  it's noisy outside CI. I treat it as a CI/specialized tool.

**CI (the exhaustive net): everything.**
ASan, TSan, and MSan as separate jobs, across both GCC and Clang, plus ASan on Windows. CI is
exactly where the slow, thorough passes belong — they don't slow my keyboard down, and nothing
lands on `main` un-sanitized.

## `-Werror`: off locally, on in CI

Warnings-as-errors in the inner loop is friction — it hard-blocks builds on in-progress trivia (an
unused variable mid-edit) and sometimes on toolchain-specific argument warnings that aren't even my
code's fault. I keep warnings **visible** locally (`-Wall -Wextra`) but not fatal, and let **CI**
enforce zero. Same philosophy as sanitizers: see everything always, gate hard only where it doesn't
cost me iteration speed.

## Windows is its own thing

On Windows only **ASan** exists (no TSan/MSan/LSan). MSVC's ASan is the easy path — it matches the
default debug runtime. clang-cl's ASan requires the *release* CRT (`/MD`), which means its
dependencies must also be `/MD` or you hit iterator-debug-level link mismatches. So on Windows I
don't make ASan my always-on build; I use a plain debug build for iteration and reach for ASan
deliberately when chasing a memory bug.

## TL;DR

| Context | What I run | `-Werror` |
|---|---|---|
| Daily dev (Linux) | ASan + UBSan + LSan | off |
| Daily dev (Windows) | plain debug; ASan when bug-hunting | off |
| Touching threads | TSan | off |
| CI | full matrix (asan/tsan/msan × gcc/clang + Windows asan) | on |

Match the cost to the frequency. ASan+UBSan is cheap enough to always wear; TSan/MSan are tools you
pick up when the job needs them; CI wears all of it so you don't have to.

---

These are wired up as ready-to-use CMake presets (e.g. `clang-linux-asan`, `clang-linux-tsan`,
`msvc-asan`) in my C++ project template — including the fiddly Windows ASan setup:
<https://github.com/royyandzakiy/cpp-project-template>
